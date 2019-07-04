---
title: SVN客户端安装
date: 2018-12-28 01:56:21
tags:
---
svn客户端下载地址
https://tortoisesvn.net/downloads.zh.html
下载完毕后，直接下一步安装。安装后，右键SVN Checkout 
如果服务端IP为172.20.100.115，那么如下设置的game的访问目录就为：svn://172.20.100.115/game
点击ok后，会提示你输入用户名及密码。验证通过后，即可将项目下载到本地

**centos7安装svn**
//安装svn
```
yum -y install subversion
```
//创建一个仓库目录,并新建一个仓库项目(svnadmin create 仓库名)
```
mkdir /home/svn
cd /home/svn
svnadmin create svnrepos
```
//进入该仓库，可以看到该目录下生成了一些目录和文件
```
cd进入该仓库（cd svnrepos），编辑conf下面的三个文件 

1.权限文件  authz
-----查看github

2.用户密码文件  passwd	 
-----建立svn客户端用户以及密码，一行一个，如下。
[users]
admin = admin123
user1 = ley123qwe1
user2 = ley123qwe1

3.服务器配置文件 svnserve.conf
-----编辑svnserve.conf主配置文件，对以下几项修改如下
//释义
[general]
anon-access = none    #禁止匿名访问
auth-access = write   #授权用户有可写权限
password-db = passwd  #指定用户密码配置文件
authz-db = authz      #指定权限配置文件
//直接执行
sed -i 's/# anon-access = read/anon-access = none/g' svnserve.conf
sed -i 's/# auth-access = write/auth-access = write/g' svnserve.conf
sed -i 's/# password-db = passwd/password-db = passwd/g' svnserve.conf
sed -i 's/# authz-db = authz/authz-db = authz/g' svnserve.conf
```

**以上3个文件配置完成之后，启动svn服务**

//后台启动-d表示后台运行svn服务，-r是指定svn目录。
```
/usr/bin/svnserve -d -r /home/svn/

执行netstat -ntlp检查端口，该服务默认监听在3690端口上
```

客户端连接
```
svn://172.20.100.115/svnrepos

要求输入用户名密码，也就是passwd文件里配置的信息

登录成功后便可以在该工作目录里进行创建或编辑文件，操作完成后对该目录右键选择check commit提交，也就是将修改内容上传到SVN服务器。
```

上传完之后可以在linux上测试一下，输入以下命令，可以看到在windows上创建的文件
```
[root@localhost svnrepos]# svn checkout svn://172.20.100.115/svnrepos/ --username=admin --password=admin123
```
**svn新增一个仓库**
//就在原来的仓库目录下新建一个仓库即可
```
cd /home/svn/
svnadmin create svnadmin
cd 仓库名,编辑conf下面的三个文件，方法同上

/usr/bin/svnserve -d -r /home/svn/   //启动仓库目录，就把该目录下所有的仓库都启动了

客户端连接
svn://172.20.100.115/svnadmin
```

**SVN备份与恢复**
(可以写个脚本定时备份，并把备份后的数据同步到文件服务器，达到容灾效果)
1.备份
```
在SVN服务器上创建并进入备份目录
mkdir /data/svnbak
cd /data/svnbak
开始备份
svnadmin dump /home/svn/svnrepos > svnrepos.bak
svnadmin dump /home/svn/svnadmin > svnadmin.bak
```
2.恢复
```
将备份的文件同步到需要迁移的新服务器上的/home下(已经安装好了svn)，导入备份数据后，启动仓库
yum -y install subversion     //安装svn

mkdir /home/svn/     //创建仓库目录

svnadmin create /home/svn/svnrepos      //创建仓库实例

svnadmin load /home/svn/svnrepos < /home/svnrepos.bak     //导入备份数据

//多个仓库依次创建导入
svnadmin create /home/svn/svnadmin
svnadmin load /home/svn/svnadmin < /home/svnadmin.bak

/usr/bin/svnserve -d -r /home/svn/     //启动仓库   
```


































