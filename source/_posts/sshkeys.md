---
title: ssh key密钥生成与使用
date: 2018-08-09 12:23:41
tags:
  - Linux
  - GIT
categories:
  - Others
---

### 生成密钥ssh key

#### 第一步
在命令行输入以下命令来生成ssh key
```shell
ssh-keygen -t rsa
```
会出现以下信息：
```shell
Generating public/private rsa key pair.
Enter file in which to save the key (/home/orange/.ssh/id_rsa):
```
以上信息告诉我们要输入一个文件名来存储`ssh key`，它的存储目录为`/home/orange/.ssh/`。（如果什么都不输入，那么文件名默认为`id_rsa`），这里我们使用默认文件名，直接按回车<kbd>Enter</kbd>进入下一步。

#### 第二步
会出现以下信息：(第3 ~ 4行出现的信息)
```shell
Generating public/private rsa key pair.
Enter file in which to save the key (/home/orange/.ssh/id_rsa):
Created directory '/home/orange/.ssh'.
Enter passphrase (empty for no passphrase):
```
以上信息告诉我们自动创建了目录`/home/orange/.ssh`来存储`ssh key`文件，然后需要为`ssh key`创建密码（什么都不输入，默认为无密码），这里我们不需要设置密码，直接按回车<kbd>Enter</kbd>进入下一步。

#### 第三步
会出现以下信息：(第5行出现的信息)
```shell
Generating public/private rsa key pair.
Enter file in which to save the key (/home/orange/.ssh/id_rsa):
Created directory '/home/orange/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```
直接回车<kbd>Enter</kbd>进入下一步。

#### 第四步
会出现以下信息：(第6 ~ 21行出现的信息)
```shell
Generating public/private rsa key pair.
Enter file in which to save the key (/home/orange/.ssh/id_rsa):
Created directory '/home/orange/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/orange/.ssh/id_rsa.
Your public key has been saved in /home/orange/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:wMJW8YgH0fYCbKwnE2qDuoviuoHxBVty11dbDotpU54 orange@orange
The key's randomart image is:
+---[RSA 2048]----+
|   oooo.    + .  |
|  ..==o+   * B   |
|..o+Bo*.o * E .  |
|oo+B.+...o .     |
|+ o+.  .S        |
|oo .             |
|o..              |
|+.               |
|O+               |
+----[SHA256]-----+
```
到这一步，我们就生成了`id_rsa`，ssh key就储存在这个文件中。我们需要查看这个文件，把里面的内容复制到需要的地方。（github、 gitlab、 远程服务器等。）
#### 查看`id_rsa`文件
```shell
cat ~/.ssh/id_rsa.pub
```
会出现类型以下信息：（`id_rsa.pub`文件内容）
```shell
ssh-rsa AAAAB3NzaC1yc2EAAfR1V3LgCM/J/6fDTLXRhLv1GCxU3m4P09yRI6TNmDYDPrdu4l
VS0ukSRZISmtzekcP4ra3KyXRa/npf4FztH2Peq+NiBlk5zCrgAd+sdXTcrtte1foVFrMa9vIZ
nrAGIAAAAB3NzaC1yc2EAAfR1V3LgCM/J/6fDTLXRhLv1GCxU3m4P09yRI6TNmDYDPrdu4lVS0
ukSRZISmtzekcP4ra3KyXRa/npf4FztH2Peq+NiBlk5zCrgAd+sdXTcrtte1foVFrMa9vIZnrAGI orange@orange
```
### github配置 ssh key

复制以上出现的一大串内容，粘贴到自己github里面，这样以后，在当前电脑上提交/拉取代码的时候就不再需要输入密码。（其他没有生成ssh key的电脑上需要输入密码）。具体见下图：
![](https://qiniu.zcheng.site/sshkeys-ssh_key_github.png)

### gitlab配置 ssh key

类似于github配置步骤，这里不再具体说明😜。

### 远程服务器配置ssh key免密码登录
##### 正常登录远程服务器流程：
```shell
ssh user@服务器ip地址
```
出现以下信息：
```shell
user@服务器ip地址's password:
```
输入正确密码才能登录上去。

##### 接下来配置无密码登录
先通过密码登录到服务器，然后：
```shell
cd ~/.ssh/
vim authorized_keys
```
将ssh key粘贴进去，保存，退出重新登录即可不用输入密码。

The End😀
