---
layout: post
title: linux安装nginx
summary: linux安装nginx的记录及拓展
featured-img: 212737.jpg
labels: [linux, 安装记录]
---

### 流程
**1.安装各种依赖**
```no-highlight
#gcc安装，nginx源码编译需要
yum install gcc-c++

#PCRE pcre-devel 安装，nginx 的 http 模块使用 pcre 来解析正则表达式
yum install -y pcre pcre-devel

#zlib安装，nginx 使用zlib对http包的内容进行gzip
yum install -y zlib zlib-devel

#OpenSSL 安装，强大的安全套接字层密码库，nginx 不仅支持 http 协议，还支持 https（即在ssl协议上传输http）
yum install -y openssl openssl-devel
```

**2.wget下载**
```no-highlight
wget -c https://nginx.org/download/nginx-1.16.1.tar.gz
```

**3.解压**
```no-highlight
tar -zxvf nginx-1.16.1.tar.gz
```

**4.解压后进入目录**
```no-highlight
cd nginx-1.16.1
```

**5.使用ssl配置**
```no-highlight
./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module

建议配置上ssl 否则用https的时候会报错
the "ssl" parameter requires ngx_http_ssl_module in /usr/local/nginx/conf/nginx.conf
```

**6.编译安装**
```no-highlight
make
make install
```

**7.移动并重命名**
```no-highlight
mv nginx-1.16.1 /usr/local/nginx
```

**8.启动nginx**
```no-highlight
/usr/local/nginx/sbin/nginx
```

<br>

### 拓展
```no-highlight
启动服务
/usr/local/nginx/sbin/nginx
```

```no-highlight
停止，直接查找nginx进程id再使用kill命令强制杀掉进程
/usr/local/nginx/sbin/nginx -s stop

退出停止，等待nginx进程处理完任务再进行停止
/usr/local/nginx/sbin/nginx -s quit
```

```no-highlight
重新加载配置文件，修改nginx.conf后使用该命令，新配置即可生效
/usr/local/nginx/sbin/nginx -s reload
```

```no-highlight
查看nginx原有的模块
/usr/local/nginx/sbin/nginx -V

在configure arguments:后面显示的原有的configure参数如下：
--prefix=/usr/local/nginx --with-http_stub_status_module
```

```no-highlight
开机自启动
在rc.local增加启动代码即可
vim /etc/rc.local
增加一行 /usr/local/nginx/sbin/nginx 增加后保存

设置执行权限
cd /etc
chmod 755 rc.local
```

<br>

### 遇到的问题
**1.配置https的时候报错**
```no-highlight
错误：
the "ssl" parameter requires ngx_http_ssl_module in /usr/local/nginx/conf/nginx.conf

原因：
nginx缺少http_ssl_module模块，需要在已安装的nginx中添加ssl模块。

检查：
查看nginx原有的模块
/usr/local/nginx/sbin/nginx -V

在configure arguments:后面显示的原有的configure参数如下：确实没有ssl的字样
--prefix=/usr/local/nginx --with-http_stub_status_module

说明：
我的安装目录是 /usr/local/nginx
源码包在 /home/nginx/nginx-1.16.1

解决：
进入源码包
cd /home/nginx/nginx-1.16.1

重新配置ssl模块
./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module

配置完成后，运行命令make命令：
make
注意：此处不能进行make install，否则就是覆盖安装

替换已安装好的nginx包
替换之前先备份：
cp /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx.bak

停止nginx服务：
/usr/local/nginx/sbin/nginx -s stop

将刚刚编译好的nginx覆盖掉原有的nginx
cp ./objs/nginx /usr/local/nginx/sbin/

启动nginx服务
/usr/local/nginx/sbin/nginx

查看nginx原有的模块，此时显示出ssl则表示配置成功
/usr/local/nginx/sbin/nginx -V
configure arguments: --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
```