---
title: OData——让查询变的随心所欲
date: 2020-05-08 00:49:31
top: true
tags:
  - .NET CORE
  - OData
categories:
  - .NET CORE
---

# OData是什么
Open Data Protocol（开放数据协议，OData）是用来查询和更新数据的一种Web协议，其提供了把存在于应用程序中的数据暴露出来的方式。OData运用且构建于很多Web技术之上，比如HTTP、Atom Publishing Protocol（AtomPub）和JSON，提供了从各种应用程序、服务和存储库中访问信息的能力。OData被用来从各种数据源中暴露和访问信息，这些数据源包括但不限于：关系数据库、文件系统、内容管理系统和传统Web站点。更多详细定义可以查阅[OData官网](https://www.odata.org/)，接下来用示例看看OData是怎么让查询随心所欲。

# 示例
新建一个.NET CORE 3+ WEBAPI 项目，安装 **Microsoft.AspNetCore.OData** 及其所有依赖项
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200507231549791.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R6MTgyMjgwMjc4NQ==,size_16,color_FFFFFF,t_70)
添加测试模型Student，用来进行数据查询

```csharp
public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int Age { get; set; }
}
```

添加数据调用控制器，继承 **ControllerBase** ，添加Get 方法以便于插叙Student 数据，添加 **[EnableQuery]** ，用来支持OData查询选项。

```csharp
[Route("api/[controller]")]
public class TestController : ControllerBase
{
    private List<Student> students = new List<Student>()
    {
        new Student()
        {
            Id = 1,
            Name = "张三",
            Age = 18,
        },
        new Student()
        {
            Id = 2,
            Name = "李四",
            Age = 88,
        },
        new Student()
        {
            Id = 3,
            Name = "赵五",
            Age = 20,
        },
        new Student()
        {
            Id = 4,
            Name = "王六",
            Age = 42,
        }
    };

    [EnableQuery]
    [HttpGet]
    public List<Student> Get()
    {
        return students;
    }
}
```
接下来在Startup 配置OData

```csharp
public void ConfigureServices(IServiceCollection services)
{

    services.AddControllers();
    //添加OData
    services.AddOData();
}
```

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }

    app.UseHttpsRedirection();

    app.UseRouting();

    app.UseAuthorization();

	//配置OData 路由节点
    app.UseEndpoints(endpoints =>
    {
    	endpoints.MapControllers();
        endpoints.EnableDependencyInjection();
        endpoints.Filter().Count().Expand().OrderBy().Select().MaxTop(null).;

    });
}
```
**这里需要注意的是，查阅了很多文档资料，都是用的以下配置，先禁用掉了控制器的端点路由配置，然后在Configure中使用MVC路由配置，这样也是可以了，但是OData7.4版本已经支持端点路由配置了，也没有必要那样去做了**
```csharp
//不推荐写法

public void ConfigureServices(IServiceCollection services)
{

    services.AddControllers(mvcOptions=>mvcOption.EnableEndpointRouting = false);
    //添加OData
    services.AddOData();
}

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }

    app.UseHttpsRedirection();

    app.UseRouting();

    app.UseAuthorization();

	app.UseMvc(routeBuilder =>
    {
        routeBuilder.EnableDependencyInjection();
        routeBuilder.Filter().Count().Expand().OrderBy().Select().MaxTop(null).;
    });   
}
```
现在可以在数据上尝试$select，$orderby，$filter，$count，$skip 和$top的常规操作，结果如图所示：

- **默认情况下  /api/test**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200508000512650.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R6MTgyMjgwMjc4NQ==,size_16,color_FFFFFF,t_70)
 - **$ orderby  /api/test?$orderby=age desc**
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200508000754861.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R6MTgyMjgwMjc4NQ==,size_16,color_FFFFFF,t_70)

**$ orderby  /api/test?$filter=age eq 42**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200508000913100.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R6MTgyMjgwMjc4NQ==,size_16,color_FFFFFF,t_70)
filter 语法条件列表
|条件  | 备注 | 示例 |
|--|--|--|
eq |等于	|$filter=priority et 1
ne|	不等于	|$filter=priority ne 1
gt|	大于|	$filter=priority gt 1
ge|	大于或等于|	$filter=priority ge 1
lt	|少于	|$filter=priority lt 1
le|	小于或等于	|$filter=priority le 1
and|	并且	|$filter=priority gt 1 and priority lt 10
or|	或者|	$filter=priority gt 1 or priority lt 10
not|	不是	|$filter=not endswith(name,'task')

 - **$ skip& $ top   /api/test? $skip=2& $top=2**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200508003458278.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R6MTgyMjgwMjc4NQ==,size_16,color_FFFFFF,t_70)
但是在执行 select 的时候数据出现了问题
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200508003721577.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R6MTgyMjgwMjc4NQ==,size_16,color_FFFFFF,t_70)

这是因为采用非**Edm**路线配置OData，则需要安装Microsoft.AspNetCore.Mvc.NewtonsoftJson 软件包来解决Json格式问题 ，然后修改Startup文件ConfigureService 以启用Json格式扩展方法

```csharp
public void ConfigureServices(IServiceCollection services)
{

    services.AddControllers().AddNewtonsoftJson();

    services.AddOData();
}
```
配置完成之后我们在来看看select 结果，很显然数据有了变动，只查出了name字段
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200508004317993.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R6MTgyMjgwMjc4NQ==,size_16,color_FFFFFF,t_70)

这里顺便在提一下**Edm**路线配置OData，主要区别在与OData路由策略的配置

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }

    app.UseHttpsRedirection();

    app.UseRouting();

    app.UseAuthorization();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();
        endpoints.Select().Filter().OrderBy().Count().MaxTop(null);
        endpoints.MapODataRoute("odata", "odata", GetEdmModel());
    });
}

private IEdmModel GetEdmModel()
{
    var odataBuilder = new ODataConventionModelBuilder();
    odataBuilder.EntitySet<Student>("Student");

    return odataBuilder.GetEdmModel();
}
```
# 总结
现在OData 在.NET CORE 3.1的配置已经初步配置完，是不是觉得数据查询变的随心所欲，在也不需要为了需求的变动来回修改Dto了，OData 的语法远远不止这些，如需了解请移步到[官网](http://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part2-url-conventions.html#_Toc31361043)查看更多语法




