---
title: linux一些基础
date: 2018-12-28 01:10:09
tags:
---
//java安装及配置环境变量
```
安装Java：
wget http://download.oracle.com/otn-pub/java/jdk/8u171-b13/e9e7ea248e2c4826b92b3f075a80e441/jdk-8u171-linux-x64.tar.gz
配置环境变量
vi /etc/profile
export JAVA_HOME=/usr/local/java/jdk1.8.0_171                //这个要和自己安装的路径相对应
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
source /etc/profile
```

//查询端口是否被占用，被哪个进程占用有两种方式：
```
1、netstat -anl | grep "80" 
2、lsof -i:80
终止进程的方式：kill -9 pid
```
//用tomcat进行http传输：
```
如在172.20.101.225上：
1、下载安装启动Tomcat (必要时修改端口server.xml）
2、把要传输的文件放入tomcat目录的webapps\ROOT下(可以在该目录下创建新的目录，便于区分文件。比如创建目录为wget)

下载方执行如下命令即可：
cd /home  （进哪个目录按需求来定）
wget 172.20.101.225:8080/wget/no_queue.conf
```
//centos7.0 没有netstat 和 ifconfig命令问题
```
运行yum install net-tools 就OK了
```
//解压rar
```
wget http://www.rarlab.com/rar/rarlinux-3.8.0.tar.gz
tar zxvf rarlinux-3.8.0.tar.gz -C /usr/local
ln -s /usr/local/rar/rar /usr/local/bin/rar
ln -s /usr/local/rar/unrar /usr/local/bin/unrar
```
//centos7改时间
```
timedatectl set-ntp no
timedatectl set-time "2018-10-18 09:32:25"
clock -w

或者
sudo timedatectl set-timezone Asia/Shanghai  同步上海时间
sudo systemctl enable ntp && sudo systemctl restart ntp
```


//CPU
```
查看cpu个数：
 cat /proc/cpuinfo | grep "physical id" |  uniq | wc -l  
查看cpu核数
 cat /proc/cpuinfo | grep "cpu cores" | uniq
```
//Pycharm和python安装教程
```
https://www.cnblogs.com/liyuspace/p/8033699.html  安装python3.6   Windows x86-64 executable installer
https://blog.csdn.net/ziwuzhulin/article/details/81512788
```


//查看程序占用内存
```
查看某一个进程所占用的内存，先通过ps命令找到进程id
top -p 2913

查看内存占用前10名的程序
ps aux | sort -k4,4nr | head -n 10
```


//安装npm和nodejs
```
curl -sL https://rpm.nodesource.com/setup_8.x | bash -
yum -y install nodejs
npm install gulp -g 
```

//安装Gtop
```
yum -y install git
git clone https://github.com/aksakalli/gtop.git
curl -sL https://rpm.nodesource.com/setup_8.x | bash -
yum -y install nodejs
npm install gtop -g
#使用：命令行输入gtop即可
```

//清除系统缓存
```
sync 同步缓存信息，把缓存的信息保存到磁盘
echo 3 > /proc/sys/vm/drop_caches 清空缓存

如果是程序占用的资源过多，不行就重启一下程序
```

//centos7安装pip
```
wget https://bootstrap.pypa.io/get-pip.py
python get-pip.py
```

//修改linux用户登录后默认目录
linux用户登录后默认目录是在/etc/passwd文件设置的
```
vim /etc/passwd
以下列两行为例，/root即为root用户登录后的默认目录;/home/wwwroot即为linuxaa用户登录后的默认目录
root:x:0:0:root:/root:/bin/bash
linuxaa:x:1005:1005::/home/wwwroot:/bin/bash
所以直接修改这个地方的路径就ok了
```

//打开文件乱码
```
首先查看系统对中文的支持
locale -a | grep zh_CN8
yum groupinstall chinese-support

输出样例如下
zh_CN.gbk
zh_CN.utf8

vim 只能正确识别列表中的中文编码文件，如需识别其他编码类型的中文文件，则需要做系统升级

vi ~/.bash_profile
文件末尾添加

export LANG="zh_CN.UTF-8"

export LC_ALL="zh_CN.UTF-8"
```

//jumpserver
```
http://docs.jumpserver.org/zh/docs/
```

//ssh加速
```
sed -i 's/#UseDNS yes/UseDNS no/' /etc/ssh/sshd_config
sed -i 's/GSSAPIAuthentication yes/GSSAPIAuthentication no/' /etc/ssh/sshd_config
```

数据库web管理工具，已下载到云盘
```
https://www.jianshu.com/p/51a9d5520d2a
```

python视频
```
视频网盘链接：https://pan.baidu.com/s/1KvyLDgfx6g-5o6VkLEFxKg
密码：qipx
```
//ssh 免密登录    yum -y install expect
```
vi ssh.sh
#!/usr/bin/expect
#实现ssh服务秘钥验证
expect<<EOF
spawn ssh-keygen -t rsa 
expect {
"(y/n)?" {send "\r";exp_continue}    --如果之前生成过秘钥，需要重新生成就需要加上这一条
"/id_rsa):" {send "\r";exp_continue}
"passphrase):" {send "\r";exp_continue}
"again:" {send "\r";}
}
expect eof
EOF
#获取IP及密码    --如果之前生成过秘钥，不需要重新生成就执行以下内容
host=`cat ip.txt|awk {'print $1"-"$2'}`
for i in $host
do
	ip=`echo $i|awk -F- {'print$1'}`
	pw=`echo $i|awk -F- {'print$2'}`
#将公钥上传至ssh服务器
#需保证主服务器能ssh连接上分发的服务器。在客户端服务器增加一条防火墙规则即可
expect<<EOF
spawn scp /root/.ssh/id_rsa.pub root@$ip:/root/.ssh/authorized_keys
expect "(yes/no)?"    --如果之前连过，这一条就不需要了，否则会报错
send "yes\r"
expect "password:"
send "$pw\r"
expect eof
EOF
done   


//ip.txt 格式

[root@localhost wwremy]# more ip.txt 
172.20.100.155 ley123456
172.20.100.123 admin23

```
