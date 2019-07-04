---
title: scp和rsync
date: 2019-06-07 16:20:15
tags:
---
scp和rsync都用于（本地或远程）文件拷贝传输
区别：
scp相当于复制粘贴，文件存在则覆盖。每一次都会同步所有文件。
rsync是复制。文件存在会直接跳过。第一次会同步所有文件，后面只同步修改过的文件。
//scp
```
语法：本地远程都需安装yum -y install openssh-clients
scp 172.20.100.11:/home/www.txt /data   拷贝远程文件到本地/data目录下
scp /data/a.txt 172.20.100.11:/home/     拷贝本地文件到远程/home目录下
如果是传输目录的话，要加-r
scp -r 172.20.100.11:/home/www  /data  
```
//rsync
```
rsync语法举例：本地远程都需安装 yum -y install rsync
rsync -av --progress web-mobile/ web-mobile-yaoji/ --exclude 'customer_config.json'
---除customer_config.json文件外，同步web-mobile下所有东西到web-mobile-yaoji目录下
-a：归档模式，以递归方式拷贝文件，同时保留文件属性
-v:显示详细信息
--progress：显示数据传输的进度信息
--exclude：排除指定文件或目录

[ ! -d /tmp/myrpms ] && mkdir -p /tmp/myrpms           #目录不存在则创建

rsync -avz root@192.168.0.100:/home/tarunika/rpmpkgs /tmp/myrpms     把远程主机100下的的文件拷贝到本地
rsync -av a/ 172.20.100.155:/home/a/       把本地a目录下的东西同步到远程主机155上的/home/a目录下 
rsync -av ./redis.tar.gz /home/wwtest         如果wwtest不是目录，这句话的意思就是把当前目录下的redis文件拷贝到home目录下，并命名为wwtest。 如果是目录，则会把redis放到该目录下，文件名不变。  

可以创建密码文件rsync.pass，然后使用--password-file指定此文件，这样就不用每次都输密码了。
echo "123" > rsync.pass   #服务器端用户tom的密码
rsync -avz --delete --password-file=rsync.pass tom@172.20.100.152::wwwroot /dest
```

###exclude
du -sh /* 
du -sh /* --exclude="proc"          排除目录proc

如果是多个后缀类型需要被排除可以在后面添加，无限制如下：
tar -cvf test.tgz test/ --exclude *.txt --exclude *.jpg

