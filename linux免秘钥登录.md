---
title: linux免秘钥登录
date: 2019-02-15 13:46:28
password: passw0rd
tags:
---
//单台免秘钥登录
```
ssh-keygen    //生成密钥
ssh-copy-id   root@10.0.0.1      //推到远程主机，此过程需要输入密码
ssh 10.0.0.1   //测试
```

//比如3台主机相互免密码登录，就需要在3台上面都执行以上命令，新的秘钥会追加到相应文件，不会影响原来的。即：
```
1.在3台上都生成秘钥  ssh-keygen
2.推送到其他两台 ssh-copy-id   root@10.0.0.2
3.测试
```
//比如几十台都通过免秘钥登陆呢？这时候就需要ansible了。
```
vim  /etc/ansible/hosts 
[jenkins]·
172.20.100.126
172.20.100.217
[jjaaa]
172.20.100.218
172.20.100.219
执行下列语句(all是所有的，也可指定单个组，如下jjaaa)，如果ip的密码都是一样的，执行的时候需输入一次密码
ansible all -m authorized_key -a "user=root key='{{ lookup('file', '/root/.ssh/id_rsa.pub')}}' path='/root/.ssh/authorized_keys' manage_dir=no" --ask-pass -c paramiko
ansible jjaaa -m authorized_key -a "user=root key='{{ lookup('file', '/root/.ssh/id_rsa.pub')}}' path='/root/.ssh/authorized_keys' manage_dir=no" --ask-pass -c paramiko


[jenkins]
172.20.100.126 ansible_ssh_pass="old123456"
172.20.100.217 ansible_ssh_pass="123456
如果密码不同  需要自定义
ansible all -m authorized_key -a "user=root key='{{ lookup('file', '/root/.ssh/id_rsa.pub')}}' path='/root/.ssh/authorized_keys' manage_dir=no" --ask-pass -c paramiko


[jenkins]
172.20.100.126
172.20.100.217
172.20.100.157 ansible_ssh_pass="ley123456"
如果密码不同  需要自定义，

看到这样的，"Are you sure you want to continue connecting (yes/no)?"。。。。直接回车就可以了  
```