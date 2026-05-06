我的大模型

一、前言

（一）工作内容

研究一个开源大模型，通过python/pytorch对大模型进行微调。

（二）开发环境

1.硬件环境

#阿里云轻量应用服务器，2vCPU、4GiB、ESSD 50Gib。

腾讯云轻量应用服务器，2vCPU、2GiB、ESSD 40Gib。

2.软件环境

操作系统：Linux

大模型：DeepSeek?Qwen?Gemma?

3.用户

root r00tPass@1981

二、环境搭建

（一）创建新用户

为方便系统管理，建议创建一个新用户，作为大模型的运行环境。最简单增加用户的方法仅需要两步：

1.增加用户llm

useradd llm

2.设置密码(11mPass@1981)

passwd llm

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

（二）安装基础软件

1.安装ollamma

小贴士：将指定用户加入sudoer

以腾讯云服务器为例，1.首先查看sudoer使用的用户组

sudo cat /etc/sudoers

\# 通常情况下CentOS 默认有 %wheel ALL=(ALL) ALL

2.将用户加入到wheel用户组

sudo usermod -aG wheel llm

3.确认用户所在用户组信息

groups llm

使用官方脚本进行安装：

curl -fsSL https://ollama.com/install.sh \> install.sh

将ollama配置为服务：

#使用官方脚本安装时，会自动配置systemd服务

sudo systemctl enable ollama

sudo systemctl enabel ollama

#配置ollama服务，可通过其他设备访问

（1）编辑服务配置

sudo systemctl edit ollama

在配置文件中增加以下内容：

\[Service\]

Environment=\"OLLAMA_HOST=0.0.0.0:11434\"

（2）重启服务

sudo systemctl daemon-reload

sudo systemctl restart ollama

2.安装pip

yum install python3-pip

3.安装open webui

\# 安装 open-webui

pip install open-webui

\# 启动 open-webui 服务

open-webui serve

4.将open-webui配置为系统服务

（1）创建服务文件

sudo nano /etc/systemd/system/open-webui.service

#文件主要内容

\[Unit\]

Description=open-webui service

\# 等待网络连接后再启动，避免网络错误

After=network.target

\[Service\]

Type=simple

\# 指定用户和用户组

User = llm

Group = llm

\# 设置环境变量

Environment="HF_ENDPOINT=https://hf-mirror.com"

\# 关键：指定启动服务的命令，请确认此路径与你系统上的一致

ExecStart=/usr/local/bin/open-webui serve

\# 如果服务意外停止，会自动尝试重启

Restart=on-failure

\[Install\]

\# 将服务关联到多用户运行级别，实现开机自启

WantedBy=multi-user.target

\# 重新加载服务配置

sudo systemctl daemon-reload

\# 设置开机自启

sudo systemctl enable open-webui.service

\# 立即启动服务（不用重启电脑即可生效）

sudo systemctl start open-webui.service

（2）启动open-webui

\# 在llm用户下执行，等待一段时间后（5分钟左右），服务即可启动

open-webui serve

\# 使用浏览器登陆<http://175.178.57.32:8080/>

系统将创建管理员（信息如下）：

15504289319

<15504289319@163.com>

123qwe!@#
