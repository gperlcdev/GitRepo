# 我用龙虾

## 一、前言

### （一）工作内容

使用龙虾提高工作效率。

### （二）开发环境

#### 1.硬件环境

#阿里云轻量应用服务器，2vCPU、4GiB、ESSD 50Gib。

腾讯云轻量应用服务器，2vCPU、2GiB、ESSD 40Gib。

#### 2.软件环境

操作系统：Linux

AI助理：OpenClaw

大模型：DeepSeek

## 二、环境搭建

### （一）创建新用户

为方便系统管理，建议创建一个新用户，作为龙虾的运行环境。最简单增加用户的方法仅需要两步：

#### 1.增加用户claw

``` bash
useradd claw
```


#### 2.设置密码(clawpass@1981)

```bash
passwd claw
```

~~~~
小贴士：linux系统的密码策略

以爬虫开发所用的腾讯云轻量应用服务器为例，与密码策略相关的主要是两个配置文件：

/etc/pam.d/system-auth

/etc/security/pwquality.conf

密码策略主要参数：

minclass=3 \# 至少包含3种字符类型（小写、大写、数字、特殊）

dcredit=-1 \# 至少1个数字

ucredit=-1 \# 至少1个大写字母

lcredit=-1 \# 至少1个小写字母

ocredit=-1 \# 至少1个特殊字符
~~~~

### （二）获取DeepSeek API-Key

登录DeepSeek官网获取API-Key

sk-f7d3c18fd9f54ea384e041436351827c

base-url https://api.deepseek.com

### （三）安装OpenClaw

#### 1.获取OpenClaw安装脚本

~~~bash
curl -fsSL <https://openclaw.ai/install.sh>
~~~

#### 2.在服务器中安装npm

~~~bash
yum install npm #已经安装npm的可以忽略
~~~

#### 3.升级node版本

在我的腾讯云环境中，使用OpenClaw安装脚本时提升node版本不对。可以使用nvm对node进行版本管理，执行以下脚本：

~~~
小贴士：如何安装nvm

# 问题现象：root用户可以正常运行nvm，claw用户不能运行nvm


curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

我所在服务器的问题是：环境变量NVM_DIR设置为"/root/.nvm"，claw用户没有写root目录的权限。我的做法是临时改变NVM_DIR的值，设置为claw用户主目录下的.nvm文件夹。脚本：export NVM_DIR="/home/claw/.nvm"
~~~

#### 4.安装OpenClaw

在claw主目录下执行：bash install.sh

安装完成后，执行openclaw config命令，对model进行设置。

## 三、设置技能
