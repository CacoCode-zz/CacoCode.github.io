---
title: 如何搭建一个简易的个人网站
date: 2018-04-10 12:46:50
tags:
  - Hexo
categories:
  - Hexo
  - Setup and deployment
---
> 以下全部操作均在Ubuntu 18.04.1 LTS中进行测试。

## 先做好准备工作

### 安装工具

所需要的工具： `node.js`、`hexo`、`git`，如你已经安装，跳过这一步。

#### 安装node.js

```bash
$ node -v
```

如果提示命令没找到，按照下面进行安装，否者跳过这一步。
我这里安装的是12.x版本，安装其他版本参见[链接](https://github.com/nodesource/distributions/blob/master/README.md#installation-instructions)

```bash
$ curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
$ sudo apt-get install -y nodejs
```

#### 安装hexo

```bash
$ hexo -v
```
如果提示命令没找到，按照下面进行安装，否者跳过这一步。

```bash
$ npm install -g hexo-cli
```
#### 安装git

```bash
$ git --version
```

如果提示命令没找到，按照下面进行安装，否者跳过这一步。

```bash
$ sudo apt-get install git
```

## 开始搭建博客

### 1. 注册一个github账号（有账号跳过这一步）

[注册地址](https://github.com/)

### 2. 新建一个repository

如下图所示
![](https://qiniu.zcheng.site/hexo-build-new_a_repository.png)

点击`New repository`后会出现
![](https://qiniu.zcheng.site/hexo-build-creating_repository.png)

注意这里的`Repository name`文本框。`黄色`框起来的`username`你可以随便取名。<code style="color: #3a95e4">浅蓝色</code>框起来的<code style="color: #3a95e4">github.io</code>必须得填<code style="color: #3a95e4">github.io</code>。也就是形式得是username.`github.io`。至于为什么，具体看[官方说明](https://pages.github.com/)。

其他的自己看着填入即可。

### 3. 拉取repository到本地电脑

![](https://qiniu.zcheng.site/hexo-build-clone_repository.png)

```bash
# 拉取username.github.io.git到本地
$ git clone https://github.com/czorange123/username.github.io.git
# 等待克隆完毕后
$ cd username.github.io.git
# 新建一个hexo项目
$ hexo init blog
# 等待执行完毕，会生成一个blog文件夹，里面就是我们的博客项目文件了
# 将.git文件夹复制到blog文件夹下，这一步是为了将blog项目提交的github上去
$ cd blog
$ cp -r ../.git/ .
# 测试hexo博客是否创建成功
$ hexo generate
$ hexo server
```

出现如下信息说明搭建成功

> INFO  Start processing
  INFO  Hexo is running at http://localhost:4000 . Press Ctrl+C to stop.

项目默认运行在4000端口，点击[地址](http://localhost:4000)即可访问。

### 4. 将项目提交到github

为了能使用github提供的github pages，我们需要将项目提交到github

```bash
# 在blog目录下
$ git status
$ git add .
$ git commit -m"init hexo-blog"
$ git push
# 提示输入github用户名和密码，而后即可提交上去
```

提交上去后github上项目是这样的
![](https://qiniu.zcheng.site/hexo-build-gitpush.png)

### 5. 部署项目

使用编辑器打开项目（blog）文件夹，在根目录下找到`_config.yml`文件，修改`deploy`配置，如下

1. 修改root配置
![](https://qiniu.zcheng.site/hexo-build-update_config.png)

1. 修改deploy配置
![](https://qiniu.zcheng.site/hexo-build-deploy_gh-pages.png)

1. 然后开始部署
```bash
# 在blog目录下
# 先安装部署用的包
$ npm install hexo-deployer-git --save
# 等待安装完毕后
$ hexo deploy
# 提示输入github用户名和密码，而后即可部署上去
```

### 6. 在线访问

打开github刚提交的项目，点击settings，如下
![](https://qiniu.zcheng.site/hexo-build-github_settings.png)

往下滚动找到`GitHub Pages`
![](https://qiniu.zcheng.site/hexo-build-gh-pages.png)

通过链接即可访问部署好的博客

---

#### 相关文档 & 资料

- [hexo官方中文文档](https://hexo.io/zh-cn/docs/)
- [Markdown 语法介绍](https://coding.net/help/doc/project/markdown.html)

#### demo & 项目地址

- [demo在线地址](https://zchengsite.github.io/username.github.io/)
- [项目地址](https://github.com/zchengsite/username.github.io)

The End😀
