---
title: nswag基于多个接口程序生成
date: 2019-06-24 09:57:14
top: false
tags:
  - Angular
  - nswag
categories:
  - Angular
---

为了兼容微服务架构的，我们需要将多个服务接口项目对接一个Angular前端项目，此文档我们将使用Nswag基于多个接口程序生成请求代码。

项目基于麦扣的Angular前端框架做详细说明，找到根目录下的**nswag**文件夹。

>   对每个接口服务都建一个*service.config.[接口服务名].nswag。*
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190624095505952.png)

>   *service.config.[接口服务名].nswag* 内容基本一样，需要修改地方如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190624095532256.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R6MTgyMjgwMjc4NQ==,size_16,color_FFFFFF,t_70)

>   Swagger.json 地址改成对应的地址。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190624095549903.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R6MTgyMjgwMjc4NQ==,size_16,color_FFFFFF,t_70)

>   codeGenerators节点下ClassName
>   注意加上接口服务名称的前缀，以便调用的时候区分。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190624095614639.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R6MTgyMjgwMjc4NQ==,size_16,color_FFFFFF,t_70)

>   codeGenerators节点下output，本节点是生成的代码文件保存地址，只要修改最后文件名，格式为*[接口服务名]-service-proxies.ts*

>   以上修改完成之后执行Nswag命令 ，可见执行了两个Nswag文件
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190624095625784.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R6MTgyMjgwMjc4NQ==,size_16,color_FFFFFF,t_70)

>   最后在*service-proxy.module.ts*文件中引入注册就能用了
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190624095638119.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R6MTgyMjgwMjc4NQ==,size_16,color_FFFFFF,t_70)
