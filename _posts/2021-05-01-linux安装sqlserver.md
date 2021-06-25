---
layout: post
title: linux安装sqlserver
summary: linux安装sqlserver的记录及拓展
featured-img: 212737.jpg
labels: [linux, 安装记录]
---

<style>
    a {
        -webkit-tap-highlight-color: rgba(0, 0, 0, 0);
        text-decoration: none;
        color: #277cea;
        -webkit-transition: all .2s linear;
        transition: all .2s linear;
        border-bottom: 1px dashed #277cea
    }
</style>

### 流程
**1.替换国内yum镜像源**
```no-highlight
1.先备份系统自带yum源配置文件/etc/yum.repos.d/CentOS-Base.repo
    mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup

2.清华镜像
    进入yum配置文件目录
    cd /etc/yum.repos.d/
    在此目录执行下面命令修改配置，把原始镜像源替换成清华镜像源。该命令会自动备份原始配置文件。
    sed -e 's|^mirrorlist=|#mirrorlist=|g' \
             -e 's|^#baseurl=http://mirror.centos.org|baseurl=https://mirrors.tuna.tsinghua.edu.cn|g' \
             -i.bak \
             /etc/yum.repos.d/CentOS-*.repo
# 注意其中的*通配符，如果只需要替换一些文件中的源，请自行增删。
# 注意，如果需要启用其中一些 repo，需要将其中的 enabled=0 改为 enabled=1。

3.阿里镜像
    wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo


4.清除yum缓存
    yum clean all

5.运行命令生成缓存
    yum makecache

6.更新
    yum -y update
```

**2.下载微软官方的sqlserver源到本地**
```no-highlight
wget -O /etc/yum.repos.d/mssql-server.repo https://packages.microsoft.com/config/rhel/7/mssql-server-2017.repo
```

**3.安装mssql-server（SQL Server软件包）**
```no-highlight
yum install -y mssql-server
```

**4.如果机器空闲内存低于2G的话，可破解内存限制**
```no-highlight
1.首先切换到 /opt/mssql/bin 目录下
    cd /opt/mssql/bin/
    
2.然后保存备份文件
    mv sqlservr sqlservr.old

3.使用python修改二进制文件，把里面的2G内存限制改为512M
    python 

    oldfile = open("sqlservr.old", "rb").read()
    
    newfile = oldfile.replace("\x00\x94\x35\x77", "\x00\x80\x84\x1e")
    
    open("sqlservr", "wb").write(newfile)
    
    exit()
```

**5.选择想要安装的sql server版本，以及设置SA用户密码**
```no-highlight
/opt/mssql/bin/mssql-conf setup

选择SQL Server版本:
    1.Evaluation (免费，无生产许可，180 天限制)
    2.Developer (免费，无生产许可)
    3.Express (免费)
    4.Web (付费版)
    5.Standard (付费版)
    6.Enterprise (付费版)
    7.Enterprise Core (付费版)
    8.我通过零售渠道购买了许可证并具有要输入的产品密钥。
按需求选择，我选择了2

询问是否接受许可条款
输入yes，回车继续下一步

选择语言
    1.Englis
    2.Deutsch
    3.Español
    4.Français
    5.Italiano
    6.日本語
    7.한국어
    8.Português
    9.Русский
    10.中文 – 简体
    11.中文 （繁体）
按需求选择，我选择了10

接下来输入超级管理员(SA)的密码，输入二次。
请确保为 SA 帐户指定强密码（最少 8 个字符，包括大写和小写字母、十进制数字和/或非字母数字符号）

输入完密码后会自动启动服务
可通过systemctl status mssql-server 查看运行状态，看到active(runing)既安装成功了

● mssql-server.service - Microsoft SQL Server Database Engine
   Loaded: loaded (/usr/lib/systemd/system/mssql-server.service; enabled; vendor preset: disabled)
   Active: active (running) since Tue 2021-05-18 15:26:06 CST; 3s ago
     Docs: https://docs.microsoft.com/en-us/sql/linux
 Main PID: 2285015 (sqlservr)
    Tasks: 1
   Memory: 311.2M
   CGroup: /system.slice/mssql-server.service
           └─2285015 /usr/local/mssql/bin/sqlservr
```

<br>

### 安装sqlserver命令行工具
**1.下载微软官方的软件包yum源**
```hgignore
wget -O  /etc/yum.repos.d/msprod.repo https://packages.microsoft.com/config/rhel/7/prod.repo
```

**2.如果以前装过mssql，则需要删除较旧的UnixODBC软件包**
```hgignore
yum remove unixODBC-utf16 unixODBC-utf16-devel
```

**3.安装mssql工具包和UnixODBC开发人员软件包**
```hgignore
这一步命令会出现两次询问：是否接受许可条款，都输入yes，回车确定
yum install -y mssql-tools unixODBC-devel
```

**4.添加PATH环境变量**
```hgignore
配置
echo "export PATH=$PATH:/opt/mssql-tools/bin" >> /etc/profile
重新加载
source /etc/profile
```

**4.使用sqlcmd命令连接本地的sqlserver，输入之前设置的SA密码**
```hgignore
不显示输入密码
sqlcmd -S localhost -U SA -p

显示输入密码
sqlcmd -S localhost -U SA -P 'Aa123456'

如果自定义了端口则在ip后面加上",自定义端口"
sqlcmd -S localhost,自定义端口 -U SA -P

如果成功，应会显示 sqlcmd 命令提示符：1>
```

<br>

### 拓展

```no-highlight
启动服务
systemctl start mssql-server
```

```no-highlight
停止服务
systemctl stop mssql-server
```

```no-highlight
重启服务
systemctl restart mssql-server
systemctl restart mssql-server.service
```

```no-highlight
自定义端口 10008
/usr/local/mssql/bin/mssql-conf set network.tcpport 10008
重启即可
systemctl restart mssql-server.service

如果自定义了端口则在ip后面加上",自定义端口"
sqlcmd -S localhost,自定义端口 -U SA -P
```

```no-highlight
修改密码:

停服务
systemctl stop mssql-server
设置密码
/usr/local/mssql/bin/mssql-conf set-sa-password
启动服务
systemctl start mssql-server
```

<br>

### 遇到的问题
**1.yum install -y mssql-server 的时候报错**
```no-highlight
错误:
    nothing provides python needed by mssql-server

确保有python环境的前提下，下载下来，rpm的方式安装
    输入:python -V
    显示:Python 2.7.15
    
    yum download mssql-server
    rpm -Uvh --nodeps mssql-server*rpm
```

**2.mssql-conf setup的时候，运行py脚本报错**
```no-highlight
/usr/local/mssql/bin/mssql-conf setup
/usr/local/mssql/bin/../lib/mssql-conf/mssql-conf.py: /usr/bin/python2: 解释器错误: 没有那个文件或目录

这里提到的是没有python2
确保有python环境的前提下
输入:python -V
显示:Python 2.7.15

则说明没有python2的软连接
解决，创建软连接
ln -s /usr/local/bin/python2.7  /usr/bin/python2

再运行py脚本就成功了
```
<div style="text-align:center"><a href="{{site.baseurl}}/2021/05/04/linux安装python2.7.html">python环境安装</a></div>

**3.查看状态时显示报错**
```no-highlight
systemctl status mssql-server

● mssql-server.service - Microsoft SQL Server Database Engine
   Loaded: loaded (/usr/lib/systemd/system/mssql-server.service; enabled; vendor preset: disabled)
   Active: failed (Result: exit-code) since Tue 2021-05-18 15:06:21 CST; 2s ago
     Docs: https://docs.microsoft.com/en-us/sql/linux
  Process: 2228594 ExecStart=/opt/mssql/bin/sqlservr (code=exited, status=203/EXEC)
 Main PID: 2228594 (code=exited, status=203/EXEC)
 
注意这段 ExecStart=/opt/mssql/bin/sqlservr (code=exited, status=203/EXEC)
默认启动路径是/opt/mssql/bin/sqlservr

但是我改了路径，放在了/usl/local/mssql

然后确保sqlservr文件有执行权限
chmod 777 sqlservr

所以找到 systemctl start mssql-server 脚本修改一下路径
脚本路径:/usr/lib/systemd/system/mssql-server.service
打开后找到[Service] 下的
ExecStart=/opt/mssql/bin/sqlservr

修改成我自定义的路径
ExecStart=/usr/local/mssql/bin/sqlservr

修改后 重新加载一下 systemctl
systemctl daemon-reload

再启动就好了
systemctl start mssql-server
```

**4.修改了默认路径，修改配置加载时报错**
```no-highlight
默认路径是在/opt/mssql
我mv到了/usr/local/mssql

如果修改了默认路径，只要用到了mssql-conf的地方就会报错
正在配置 SQL Server...
bash: /opt/mssql/bin/sqlservr: 没有那个文件或目录

例如:
1./opt/mssql/bin/mssql-conf setup
2.修改密码等

解决:
配置的时候会调用这个脚本
/usr/local/mssql/lib/mssql-conf/invokesqlservr.sh

修改如下:
CMDLINE="/opt/mssql/bin/sqlservr $@" 
改成自定义的路径 
CMDLINE="/usr/local/mssql/bin/sqlservr $@"
```
