---
layout: post
title: linux安装python2.7
summary: linux安装python2.7的记录及拓展
featured-img: 189039.jpg
labels: [Linux, 安装记录]
---

### 流程
**1.先安装安装 GCC 包**
```no-highlight
yum install gcc openssl-devel bzip2-devel
```

**2.wget 下载**
```no-highlight
cd /home/python
wget https://www.python.org/ftp/python/2.7.15/Python-2.7.15.tgz
```

**3.解压**
```no-highlight
tar -zxvf Python-2.7.15.tgz
```

**4.安装python**
```no-highlight
cd Python-2.7.15
./configure --enable-optimizations
make altinstall
```

**5.配置环境变量**
```no-highlight
配置
echo "export PATH=$PATH:/opt/mssql-tools/bin" >> /etc/profile
或者
PATH=$PATH:/usr/src/Python-2.7.15

重新加载
source /etc/profile

查看PATH
echo $PATH

查看python的版本
python -V
```

**6.添加软连接**
```no-highlight
ln -s /usr/local/bin/python2.7  /usr/bin/python2
```