---
title: jenkins的升级和迁移
date: 2018-12-18 17:15:46
password: passw0rd
tags:
---

**先说jenkins的迁移吧......**
比如想把一台A虚拟机上的jenkins迁移到另一台B上面：
首先在B机器上搭好Jenkins，然后启动jenkins，安装插件，该配置的都配好。然后进行如下操作：

1、首先确认Jenkins的job存放目录。
<font color="#FF0000"> 注意：jenkins默认的主目录存放在当前用户家目录路径下的.jenkins目录中。如jenkins使用root用户启动，则主目录为/root/.jenkins</font> 
本例中Jenkins A的工作目录为/root/.jenkins，Jenkins B的工作目录为/root/.jenkins。

2、接下来迁移jobs目录，压缩A机器上Jenkins工作目录下的jobs目录，并拷贝到B：
```
cd  /root/.jenkins
tar  -czvf  jobs.tar.gz  jobs
scp  jobs.tar  172.20.100.115:/root/.jenkins/
```
3、然后在B服务器上解压jobs.tar.gz，解压完后最好重启下B上的jenkins服务
```
tar -zxvf  jobs.tar.gz
```

当时做迁移查阅资料时，网上说迁移的时候可以直接把老服务器jenkins主目录数据整个拷贝过去，也可以单独拷贝jenkins主目录下的config.xml文件以及jobs、users、workspace、plugins四个目录（这是主要的迁移数据）,但是我做的时候就只拷贝了下jobs目录，当然也只是需要拷贝项目，拷贝过来后，在新的jenkins上就可以看到所有原来A上的服务了，然后再手动创建视图，把对应的服务勾选到相应的视图即可。（还有一种是拆分，就是把A上的部分服务拆分到B上，一样的做法，就是把jobs里面你要拆分出去的服务打包，传到B上就可以。）


迁移注意事项：迁移可能会对版本有要求，我记得当时B上安装的是最新版的jenkins，然后A上的服务迁移过来后，点击构建总是报错，但是如果自己新创建一个任务后，构建是没问题的。  好在当时服务少，我就用了半小时把这几个服务新建了一下，测试没问题...如果多的话怎么办呀，一来是可以安装原来的版本，二来呢把A上的jenkins升级到和B一个版本，然后再拷贝，就没问题了。亲自测试过的。


**再说jenkins的升级......**<font color="#FF0000"> 真的是so esay!</font> 
1.停掉jenkins
2.备份webapp下的jenkins包， 然后删除rm jenkins.war
3.下载最新的：wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war
4.重新启动jenkins
