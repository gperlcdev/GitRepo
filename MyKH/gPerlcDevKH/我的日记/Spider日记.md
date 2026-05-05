# 自己写爬虫

##  一、前言

### （一）工作内容

1.编写网络爬虫，对指定网站进行定期爬取，提升个人信息获取量。

2.编写个人网页，与服务号相关联，做为个人信息获取渠道。

### （二）开发环境

#### 1.硬件环境

#阿里云轻量应用服务器，2vCPU、4GiB、ESSD 50Gib。

腾讯云轻量应用服务器，2vCPU、2GiB、ESSD 40Gib。

#### 2.软件环境

操作系统：Linux

开发架构：Scrapy + pyWeb

## 二、环境搭建

### （一）创建新用户

为方便系统管理，建议创建一个新用户，作为爬虫开发用户。最简单增加用户的方法仅需要两步：

#### 1.增加用户web

~~~bash
useradd web
~~~
#### 2.设置密码(webpass@1981)

~~~bash
passwd web
~~~

~~~
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
~~~
### （二）搭建爬虫开发环境

Scrapy需要python版本在3.9以上，如果检查服务器python版本低于3.9，需要首先升级python。

执行命令：
~~~bash 
yum install python3.11 
~~~

命令执行完毕后，系统中将安装python3.11版本。

为了避免Scrapy安装对现有系统环境的影响，采用"虚拟机"方式安装Scrapy。

#### 1.创建虚拟机环境

执行命令：
~~~bash
python3.11 -m venv spiderEnv
~~~
最后一个参数为虚拟环境名称，命令执行完毕后将在当前目录下创建spiderEnv文件夹。

#### 2.进入（退出）虚拟机环境

执行命令：
~~~bash
source \~/spiderEnv/bin/activate
~~~
命令执行完毕后，系统进入spiderEnv虚拟环境。

执行命令：
~~~bash
deactivate
~~~
命令执行完毕后，系统退出spiderEnv虚拟环境。

#### 3.安装Scrapy

执行命令：pip install scrapy

#### 4.创建爬虫工程

执行命令：scrapy startproject mySpider

命令执行完毕后，将在当前目录下生成mySpider目录，该目录即为爬虫工程的原型目录。

### （二）搭建mysql数据库

对于爬虫获取的数据，计划通过数据库进行保存，采用了mysql数据库。

Mysql安装方式，我参照mysql官网Installing MySQL on Linux Using the MySQL
Yum Repository中的相关内容。

#以下内容为阿里云服务器中的操作方式。

#### 1.选择mysql安装包，我选择的是安装包文件是：

mysql84-community-release-el10-2.noarch.rpm

#### 2.安装mysql资源库

执行命令：yum localinstall mysql84\*\*

该命令执行完毕后，将在服务器的yum资源库中安装mysql的资源地址。

执行命令：yum
repolist，可以显示当前服务器中已经安装的资源库列表，可以看到mysql相关资源已经安装。

#### 3.安装mysql

执行命令：yum install mysql

提示：通过mysql官网下载的安装包，默认的linux版本是Redhat或者Fedola，使用阿里云Linux可能导致相关的版本号不正确，从而无法下载mysql安装相关文件。我的解决办法是直接更改对应的配置文件：

/etc/yum.repo.d/mysql-community.repo

将该文件中的\$releasever直接替换为8。这个版本号可以直接从yum的下载地址中查到。

错误应对：阿里云使用命令yum install
mysql-server安装时，系统提示错误信息：

Error: Transaction test error:

file /usr/lib64/mysql/libmysqlclient.so.21 from install of
mysql-community-libs-8.0.44-1.el7.x86_64 conflicts with file from
package mysql-community-libs-compat-8.4.7-1.el8.x86_64

处理方式：

（1）清理已安装的mysql相关软件包

rpm -qa \| grep mysql \| xargs rpm -e \--nodeps

（2）重新在系统中安装mysql源（RHEL8/CentOS8）

yum install

https://dev.mysql.com/get/mysql80-community-release-el8-1.noarch.rpm

#### 4.启动mysql服务

执行命令：systemctl start mysqld

#### 5.登录mysql数据库

（1）首次登录数据库，查看系统临时密码。

grep \'temporary password\' /var/log/mysqld.log

（2）使用临时密码登录。使用以下命令，输入临时密码。mysql -u root -p

（3）修改登录密码。

alter user \'root\'@localhost identified by \'Pswdx@1981\';

（4）创建用户www，用于爬虫开发。

create user \'www\'@\'%\' identified by \'Pswdx@1981\';

（5）为用户www授权。

#以下内容为腾讯云服务器中的操作方式。

1.安装mysql源

yum install

<https://dev.mysql.com/get/mysql80-community-release-el8-1.noarch.rpm>

2.安装mysql client/server

yum install mysql #安装client

yum install mysql-community-server #安装server

[小贴士：安装过程中出现"GPG check failed"]{.mark}

[GPG key不匹配导致，通过下面命令临时解决。]{.mark}

[yum install mysql-community-server \--nogpgcheck]{.mark}

3.启动mysql服务

systemctl start mysqld

### （三）搭建mongodb数据库

#### 1.创建MongoDB官方源文件

sudo tee /etc/yum.repos.d/mongodb-org-7.0.repo \<\< \'EOF\'

\# 适用于OpenCloudOS/CentOS/RHEL 8及兼容系统

\[mongodb-org-7.0\]

name=MongoDB Repository 7.0

baseurl=https://repo.mongodb.org/yum/redhat/8/mongodb-org/7.0/x86_64/

gpgcheck=1

enabled=1

gpgkey=https://www.mongodb.org/static/pgp/server-7.0.asc

\# 可选：添加测试版本源

\[mongodb-org-testing\]

name=MongoDB Testing Repository

baseurl=https://repo.mongodb.org/yum/redhat/8/mongodb-org/testing/x86_64/

gpgcheck=1

enabled=0

gpgkey=https://www.mongodb.org/static/pgp/server-7.0.asc

EOF

#### 2.针对OpenCloudOS的兼容性调整

\# 如果OpenCloudOS版本不是8，需要手动调整（先查看版本）

cat /etc/os-release \| grep -E \"\^(VERSION_ID\|PLATFORM_ID)\"

\# 如果系统版本是9或其它，修改baseurl中的版本号

\# 将上面配置中的 \.../redhat/8/\... 改为
\.../redhat/9/\...（根据实际版本）

sudo sed -i \'s\|/redhat/8/\|/redhat/9/\|g\'
/etc/yum.repos.d/mongodb-org-7.0.repo

#### 3.验证并安装

\# 导入GPG密钥（备用方法，如果自动导入失败）

sudo rpm \--import https://www.mongodb.org/static/pgp/server-7.0.asc

\# 清理并重建缓存

sudo yum clean all

sudo yum makecache

\# 查看MongoDB源是否可用

yum repolist \| grep -i mongo

\# 搜索MongoDB软件包

yum search mongodb-org

\# 安装MongoDB社区版

sudo yum install -y mongodb-org

#### 4.安装后配置

\# 启动MongoDB服务

sudo systemctl start mongod

sudo systemctl enable mongod

sudo systemctl status mongod

\# 查看版本确认安装成功

mongod \--version

\# 创建管理员用户（安全必需）

mongosh \--eval \"

use admin

db.createUser({

user: \'admin\',

pwd: \'mongopass@1981\',

roles: \[{ role: \'root\', db: \'admin\' }\]

})

\"

\# 启用身份验证（编辑配置文件）

sudo vi /etc/mongod.conf

\# 添加或修改：

\# security:

\# authorization: enabled

\# 重启服务

sudo systemctl restart mongod

#### 5.python环境配置

Pip install pymongo

## 三、编写爬虫

在创建的Scrapy爬虫工程mySpider中可以针对不同的任务创建多个不同的爬虫。

### （一）人民网爬虫

#### 1.创建爬虫

scrapy genspider s_people http://www.people.com.cn

#### 2.创建爬取数据结构（Item）

主要涉及items.py文件，在文件中增加以下代码：

class UrlItem(scrapy.Item):

\# URL Item: title, url

title = scrapy.Field()

url = scrapy.Field()

date = scrapy.Field()

该类继承自scrapy.Item，其中的每一个元素均为scrapy.Field()实例。

需要注意的是在进行赋值时需要采用以下方式：item\['title'\]='title'。而不能使用item.title='title'。

#### 3.在爬虫中解析网页、保存数据

使用XPath对网页进行解析，相关内容和方式不在本篇详述。对于解析出的元素保存为Item结构。主要涉及爬虫类中的parse函数。例如：

item = UrlItem()

item\[\'title\'\] = \"No Title\" if len(topline_title) \< 1 else
topline_title\[0\];

item\[\'url\'\] = \"No Url\" if len(topline_url) \< 1 else
topline_url\[0\];

item\[\'date\'\] = s_date

yield item #该语句将获取的数据交由pipeline进行存储

#### 4.将数据保存至数据库

主要涉及SpeoplePipeline类

class SpeoplePipeline:

#初始化数据库连接，注意quote_plus函数的使用

def \_\_init\_\_(self):

#client = pymongo.MongoClient(host=settings.MONGODB_HOST,
port=settings.MONGODB_PORT)

self.client =
pymongo.MongoClient(f\"mongodb://{settings.MONGODB_USER}:{quote_plus(settings.MONGODB_PASS)}@{settings.MONGODB_HOST}:{settings.MONGODB_PORT}\")

self.db = self.client\[settings.MONGODB_DBNAME\]

self.doc = self.db\[settings.MONGODB_DOC_PEOPLE\]

#将数据保存入数据库

def process_item(self, item, spider):

info = dict(item)

self.doc.insert_one(info)

return item

#关闭数据库连接

def \_\_del\_\_(self):

if self.client:

self.client.close()

#### 5.运行爬虫

scrapy crawl s_people

注意：该条命令须在scrapy.cfg所在目录下执行，可以通过scrapy
list命令查看可以使用的爬虫。

## 附录（参考资料）

### 1.  Scrapy官方文档（https://www.scrapy.org/）

### 小知识：python格式化输出

Python 3.6+ 项目：一律使用 f-string，最简洁高效

\# f-string 核心语法

f\"{变量}\" \# 基本

f\"{表达式}\" \# 表达式

f\"{变量:格式}\" \# 格式化

f\"{变量!转换}\" \# 转换(!r=repr, !s=str, !a=ascii)

\# 常用格式说明符

\# :.2f - 2位小数

\# :, - 千位分隔符

\# :\<10 - 左对齐宽度10

\# :\>10 - 右对齐宽度10

\# :\^10 - 居中对齐宽度10

\# :% - 百分比

\# :x - 十六进制

\# :#x - 带0x前缀的十六进制

\# :08b - 8位二进制前导零

\# str.format() 等效

\"{:.2f}\".format(3.14159) == f\"{3.14159:.2f}\"

\"{:\<10}\".format(\"text\") == f\"{\'text\':\<10}\"

\"{:,}\".format(1000) == f\"{1000:,}\"

### 小知识：VI最常用的15个命令（80%场景）

i → 插入

Esc → 退出插入模式

:wq → 保存退出

dd → 删除行

yy → 复制行

p → 粘贴

/text → 搜索

u → 撤销

:set nu → 显示行号

:q! → 强制退出

gg → 到文件头

G → 到文件尾

:n → 到第n行

Ctrl+f/Ctrl+b → 翻页

:%s/old/new/g → 全局替换

### 小知识：MONGODB

一、用户权限管理

使用mongosh可以进行mongodb数据库管理，默认情况下登录当前服务器test数据库，默认情况没有用户登录，将导致很多命令无权执行。如下命令：

mongosh

此种情况下，可创建一个管理员用户，命令如下：

1.切换到admin数据库

use admin

2.创建用户admin，设置密码，用户权限为root

db.createUser({user:\"admin\", pwd:\"mongopass@1981\",
roles:\[{role:\"root\", db:\"admin\"}\])

3.使用admin用户登录数据库

mongosh -u admin -p mongopass@1981 admin

二、创建表

db.createCollection(\"docPeople\")\
相关信息

1.  OS用户信息

    root swdx@1981

    web webpass@1981

2.  mongodb用户信息

    admin mongopass@1981
