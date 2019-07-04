---
title: centos7升级python
date: 2018-12-14 09:38:07
password: passw0rd
tags:
---

升级python为3.6，centos7自带python为Python 2.7，    直接执行即可
```
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
cd /etc/yum.repos.d/
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
yum makecache
cd /usr/local/src
wget "https://www.python.org/ftp/python/3.6.0/Python-3.6.0.tgz"
tar -zxvf  Python-3.6.0.tgz -C /tmp
cd /tmp/Python-3.6.0/
yum -y groupinstall "Development tools"
yum -y install openssl-devel bzip2-devel expat-devel gdbm-devel readline-devel sqlite-devel
yum install libffi-devel -y
./configure --prefix=/usr/local/python3
./configure --enable-optimizations
make && make install
cd /usr/bin
ln -s /usr/local/python/bin/python3.6 /usr/bin/python3
rm  python
ln -s /usr/local/bin/python3.6 /bin/python
```
python    进入python3
python2    进入python2

<font color="#FF0000"> 升级完后可能会导致yum不能用，解决方法如下：</font> 


将下列3个文件中原来的 /usr/bin/python   改成  /usr/bin/python2.7  就行 
centos6版本  的改成 python2.6
centos7版本  的改成 python2.7

1、vi /usr/bin/yum
2、vi /usr/libexec/urlgrabber-ext-down
3、vi /usr/bin/yum-config-manager