---
title: VS2019 (Visual Studio IDE) 添加自定义发布Nuget
date: 2020-12-14 14:54:05
tags:
  - Visual Studio 2019
  - VS2019
  - Nuget
categories:
  - 工具
---

# 打开外部工具

打开VS 【工具】 - 【外部工具】，如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201214143730259.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R6MTgyMjgwMjc4NQ==,size_16,color_FFFFFF,t_70)


# 添加发布Nuget 命令

点击 【添加】  

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201214144105746.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R6MTgyMjgwMjc4NQ==,size_16,color_FFFFFF,t_70)

> 标题：随便填写
> 
> 命令：cmd.exe 
> 
> 参数：/c  del /q  *.nupkg && nuget pack && nuget  push  *.nupkg 【你的 Nuget Key】 -Source 【Nuget 源】
> 
> 初始目录：$(ProjectDir)    $(ProjectDir) 为项目根目录
> 
> 勾上使用输出窗口查看日志


***命令的顺序 从上往下依次为【外部命令1-9】，这个在下一步要用到，切记！***


# 添加自定义菜单

点击 【工具】-【自定义】-【命令】

选中 【上下文菜单】

下拉选中【项目和解决方案上下文菜单|项目 】

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020121414480518.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R6MTgyMjgwMjc4NQ==,size_16,color_FFFFFF,t_70)

点击 【添加命令】，左侧点击工具，右侧选择相对应的外部命令1-9，确认之后点击【修改所选内容】，就能修改按钮名称

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201214144843716.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R6MTgyMjgwMjc4NQ==,size_16,color_FFFFFF,t_70)

选中要发布的项目点击自定义按钮，在输出窗口可以查看是否发布成功