---
title: Autofac在.NET Core 中的使用
date: 2020-05-06 23:40:37
top: true
tags:
  - .NET CORE
  - Autofac
categories:
  - .NET CORE
---

# 前言
Autofac 是一款.NET IoC 容器 . 它管理类之间的依赖关系, 从而使应用在规模及复杂性增长的情况下依然可以轻易地修改 。
.NET CORE 中也内置了依赖注入，但是有些情况下需要用到Autofac去进行依赖注入，Autofac支持的所有注入方式以外，还支持属性注入和方法注入。接下来我们通过示例来简单了解Autofac的使用

# 示例
新建两个.NET CORE 项目，一个WEB层，一个服务层

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020050623032390.png)

服务层中添加几个测试服务和模块文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200506230427891.png)

服务代码都如图所示

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200506230507972.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R6MTgyMjgwMjc4NQ==,size_16,color_FFFFFF,t_70)

引入Autofac Nuget包文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200506230817823.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R6MTgyMjgwMjc4NQ==,size_16,color_FFFFFF,t_70)

NetCoreAutofacServiceModule 类继承Autofac.Module，并重写Autofac管道中的Load方法，如下图多种方式注入服务。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200506230652922.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R6MTgyMjgwMjc4NQ==,size_16,color_FFFFFF,t_70)
接下来就是在WEB层配置Autofac，这里需要注意的是.Net Core2+ 和 .Net Core3+ 的配置方法稍有不同

 ***.NET CORE 2+***

在NET Core 2.1时候，AutoFac返回一个 IServiceProvider 参数注入到ConfigureServices .NET Core 服务中，写法如下：
```csharp
public IServiceProvider ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
    return AutofacProvider.RegisterForNetCore2(services);
}
```

```csharp
//将定义的策略和AutoFac 一起替换内置DI
public static IServiceProvider RegisterForNetCore2(IServiceCollection services) {
    var builder = new ContainerBuilder();
    builder.Populate(services);
    //按模块注入服务
    builder.RegisterModule<NetCoreAutofacServiceModule>(); 
    var Container = builder.Build();
    return new AutofacServiceProvider(Container);
}
```

 ***.NET CORE 3+*** 
 
 在.NET Core3.0 使用上面的写法，框架运行之后会报错：
 
 **ConfigureServices returning an System.IServiceProvider isn't supported.**
 
.NET Core 3.0 引入了具有强类型容器配置的功能。它提供了 ConfigureContainer 方法，可以在其中使用Autofac来注册事物，而不必通过 ServiceCollection 来注册事物。首先需要在 Program.cs 中修改服务工厂，内置是 ServiceProviderFactory 的，修改指定为： AutofacServiceProviderFactory 。

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        })
    .UseServiceProviderFactory(new AutofacServiceProviderFactory());
```
　然后在 Startup.cs 中添加方法 ConfigureContainer ，并配置Autofac策略
　

```csharp
public void ConfigureContainer(ContainerBuilder builder)
{
    AutofacProvider.RegisterForNetCore3(builder);
}
```

```csharp
public static void RegisterForNetCore3(ContainerBuilder builder)
{
    builder.RegisterModule<NetCoreAutofacServiceModule>();
}
```


最后在控制器中依赖注入服务，可以在方法上用[FromServices]注入，也可以通过构造函数注入

```csharp
[HttpGet]
[Route("GetName")]
public string GetName([FromServices] IThreeRepository threeRepository, 
    [FromServices] IOneService oneService,
    [FromServices] ITwoService twoService)
{
    return $"【threeRepository】 : {threeRepository.GetName()}; 【oneService】 : {oneService.GetName()} ; 【twoService】 : {twoService.GetName()}";
}
```
启动服务看看结果
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020050623340457.png)
服务已经注册成功
ThreeRepository 与 IThreeRepository 通过 RegisterType 方法注册；

```csharp
builder.RegisterType<ThreeRepository>().AsImplementedInterfaces();
```

OneService、IOneService、TwoService、ITwoService 则是通过RegisterAssemblyTypes方式注册；

```csharp
builder.RegisterAssemblyTypes(typeof(NetCoreAutofacServiceModule).Assembly).Where(a => a.Name.EndsWith("Service")).AsImplementedInterfaces();
```
