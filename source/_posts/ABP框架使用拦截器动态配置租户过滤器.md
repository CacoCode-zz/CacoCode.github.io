---
title: ABP框架使用拦截器动态配置租户过滤器
date: 2020-04-25 00:31:20
tags:
  - .NET CORE
  - ABP
  - AOP
categories:
  - .NET CORE
---

# 前言

最近项目要求在ABP框架中根据TenantId是否为空来配置是否禁用租户过滤器。ABP自身给我我们禁用租户过滤器的两种方法[官方文档](https://aspnetboilerplate.com/Pages/Documents/Data-Filters)


 - 方法一：使用工作单元
```csharp
using (_unitOfWorkManager.Current.DisableFilter(AbpDataFilters.MayHaveTenant))
{
    var people2 = _personRepository.GetAllList();                
}
```

 - 方法二：全局禁用过滤器
 

```csharp
Configuration.UnitOfWork.OverrideFilter(AbpDataFilters.MayHaveTenant, false);
```

但是方法一要修改的地方很多，嫌麻烦；方法二只能全局在Configuration中配置，不能动态改变，也不合适。于是我查阅了APB [AOP和拦截技术](https://aspnetboilerplate.com/Pages/Documents/Articles/Aspect-Oriented-Programming-using-Interceptors/index.html)，另外查阅了ABP自身注册了拦截器——UnitOfWorkRegistrar，会默认为继承自IRepository或者是IApplicationService的两种类型添加UnitOfWork特性，于是便可以通过拦截方法去实现动态禁用过滤器。

# 具体实现
首先在Application 层新建一个**TenantInterceptor** 继承**IInterceptor**接口

```csharp
public class TenantInterceptor : IInterceptor
{
    public ILogger Logger { get; set; }

    public TenantInterceptor()
    {
        Logger = NullLogger.Instance;
    }

    public void Intercept(IInvocation invocation)
    {
        // 从invocation中拿到当前注册进来的工作单元，主要用于获取TenantId
        Type t = invocation.InvocationTarget.GetType();
        var unitOfWorkManager = (t.GetProperty("UnitOfWorkManager").GetValue(invocation.InvocationTarget)) as IUnitOfWorkManager;
        //根据TenantId是否禁用租户过滤器
        if (unitOfWorkManager.Current.GetTenantId().HasValue)
        {
            invocation.Proceed(); // 执行方法体
        }
        else {
        	// 禁用租户
        	// PS:这里不可以使用 using      	
            unitOfWorkManager.Current.DisableFilter(AbpDataFilters.MayHaveTenant, AbpDataFilters.MustHaveTenant);
            invocation.Proceed(); // 执行方法体
        }
    }
}
```
拦截器里的内容很简单，主要就是根据工作单元获取TenantId来动态禁用过滤器。因为这里没有需要返回的东西，也就不用分同步异步去拦截。
接下来就是为所需要禁用租户过滤器的类注册拦截器

```csharp
public static class TenantInterceptorRegistrar
{
    public static void Initialize(IKernel kernel)
    {

        kernel.ComponentRegistered += Kernel_ComponentRegistered;
    }

    private static void Kernel_ComponentRegistered(string key, IHandler handler)
    {
        var implementationType = handler.ComponentModel.Implementation.GetTypeInfo();
        // 为实现了接口IRepository接口的所有类注册拦截器
        //if (typeof(IRepository).IsAssignableFrom(implementationType))
        //{
        //    handler.ComponentModel.Interceptors.Add(new InterceptorReference(typeof(TenantInterceptor)));
        //}
		
		// 为指定类注册拦截器
        if (InternalAsyncHelper.DisableFilterTenantTypes.Any(a => a.IsAssignableFrom(implementationType)))
        {
            handler.ComponentModel.Interceptors.Add(new InterceptorReference(typeof(TenantInterceptor)));
        }
    }
}

internal static class InternalAsyncHelper
{
    public static Type[] DisableFilterTenantTypes =
    {
        typeof(IRepository<Student,Guid>),
        typeof(IRepository<School,Guid>)
    };
}
```
在**TenantInterceptorRegistrar**的**Initialize**方法中，首先会注入整个ABP系统中唯一的IIocManager,然后就是订阅唯一的IocContainer这个容器的ComponentRegistered事件，在订阅事件中首先是获取当前触发此事件的类型信息，然后根据需求注册**TenantInterceptor**这个拦截器。

**这里有一点需要注意，本来想为实现了IApplicationService接口的类注册拦截器，但是ASP.NET Boilerplate使用动态方法拦截的功能有一些限制**

 -  如果通过接口调用该方法，则可以将其用于任何公共方法（例如，通过接口使用的Application Services）。
 -  如果直接从类引用（例如ASP.NET MVC或Web API控制器）中调用方法，则该方法应为虚拟方法。
 -  一种方法应该是虚拟的，如果它的保护。

**也就是如果将服务作为客户端的Web API控制器公开，那么方法必须是虚方法（virtual）** 附上[官方Git issues](https://github.com/aspnetboilerplate/aspnetboilerplate/issues/3237)

最后一步就是把拦截器在模块文件中初始化

```csharp
public class ApplicationCoreModule : AbpModule
{
    public override void PreInitialize()
    {
        TenantInterceptorRegistrar.Initialize(IocManager.IocContainer.Kernel);
    }

    public override void Initialize()
    {
    }
}
```

这样就可以按着自己的需要在**DisableFilterTenantTypes**  中配置自己想配置的仓储了。
