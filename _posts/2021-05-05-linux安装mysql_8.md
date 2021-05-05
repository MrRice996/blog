---
layout: post
title: linux安装mysql_8
summary: linux安装mysql_8的记录及遇到的一些问题
featured-img: sleek
mathjax: true
---

### 流程

**1.wget下载**
```
wget https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.20-linux-glibc2.12-x86_64.tar.xz
```

**2.解压**
```
tar -xvf mysql-8.0.20-linux-glibc2.12-x86_64
```

**3.移动并重命名**
```no-highlight
个人习惯会将执行文件放到/usr/local 目录下
mv mysql-8.0.20-linux-glibc2.12-x86_64 /usr/local/mysql
```

**4.创建mysql组**
```no-highlight
groupadd mysql
```

**5.创建用户mysql**
```no-highlight
useradd -r -g mysql mysql （useradd -r 创建系统用户 -g 为用户分配组）
```

**6.配置**
```no-highlight
/etc/my.cnf
```

**7.初始化mysql**
```no-highlight
/usr/local/mysql/bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql/ --datadir=/usr/local/mysql/data/
```

**8.初始化后会有个随机密码**
```no-highlight
[Server] A temporary password is generated for root@localhost: nj91U8dyk6-I
例如这个就是：nj91U8dyk6-I
```

**9.复制服务文件**
```no-highlight
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql
```

**10.启动mysql**
```no-highlight
service mysql start
```

**11.登录mysql，修改密码，设置权限等**
```no-highlight
登录：
mysql -u root -p 输入前面获得的初始密码

修改密码：
alter user user() identified by "xxx";

设置密码不过期：
ALTER USER 'root'@'localhost' PASSWORD EXPIRE NEVER;

给root开启远程访问权限：
use mysql select User，authentication_string，Host from user;

配置远程访问：
update user set host = '%' where user = 'root';

刷新权限：
flush privileges;
```


### 遇到的问题:

**1.在初始化mysql的时候，初始化失败**    

```no-highlight
初始化失败：
./mysqld: error while loading shared libraries: libaio.so.1: cannot open shared object file: No such file or directory

解决：
安装缺少的东西，yum install -y libaio-devel.x86_64
```

**2.mysql安装目录默认是/usr/local/mysql，如果更改了路径，或者mysql名字带版本号时（mysql-8.0.20）**
```no-highlight
收到如下错误：
./support-files/mysql.server: line 276: cd: /usr/local/mysql: No such file or directoryStarting MySQLCouldn't find MySQL`
server (/usr/local/mysql/[FAILED]ld_safe)        
   
解决：
把mysql移动到默认的路径或者修改配置文件/mysql/support-files/mysql.server将basedir和datadir修改成当前的路径
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
```

**3.bash: mysql: 未找到命令**
```no-highlight
添加软连接 ln -s /usr/local/mysql/bin/mysql /usr/bin
```

**4.新装的mysql没设置密码时，修改密码会报错**
```no-highlight
修改MySQL密码 修改root密码
%表示所有host都可以访问 ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'xxx';

报错： 
You must reset your password using ALTER USER statement before executing this statement. 
在初始化的时候有个密码，只是相当于临时密码，直接执行设置密码即可:
alter user user() identified by "xxx";
```

**5.找不到mysqld服务**
```no-highlight
systemctl status mysqld 
报错信息：
Unit mysqld.service could not be found. 

解决：
找到mysql.server文件，这个文件据说与mysqld文件一模一样，只是文件名不同
find / -name mysql.server 

复制mysql.server文件到/etc/init.d/目录下，重命名为mysqld 
cp /usr/local/mysql/support-files/mysql.server

/etc/init.d/mysqld systemctl status 
mysqld这个不行就用这个service mysqld status
```

### 拓展

**查看版本**
```no-highlight
mysql --version
```

**启动服务**
```no-highlight
systemctl start mysqld
```

**停止服务**
```no-highlight
systemctl stop mysqld
```

**重启服务**
```no-highlight
systemctl restart mysqld
```

**查看服务状态**
```no-highlight
systemctl status mysqld
```
