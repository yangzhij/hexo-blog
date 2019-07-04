---
title: playbook安装rabbitmq
date: 2019-02-16 23:47:50
tags:
---
//须知
```
1.前提：ansible已安装，hosts文件已配置
2.编写rabbitmq.yaml
3.ansible-playbook --syntax-check rabbitmq.yaml    进行剧本配置信息语法检查
3.执行ansible-playbook rabbitmq.yaml
4.访问172.20.100.126:15672
```
rabbitmq.yaml内容如下，注意不要有中文
```
- hosts: 172.20.100.126        //这里可以是单台主机，可以是多台，也可以是主机组，用:分隔。 all代表所有主机
  remote_user: root
  
  tasks:
        - name: installed need package         //name是对此任务进行解释，可以不写冒号后面的东西
          yum: name={{ item }} state=present
          with_items:
                - gcc
                - glibc-devel
                - make
                - ncurses-devel
                - openssl-devel
                - xmlto
                - perl
                - ntpdate

        - name: set timezone to Asia/Shanghai
          timezone:
            name: Asia/Shanghai

        - name: update time
          shell: /usr/sbin/ntpdate cn.pool.ntp.org
        
        - name: write to system  
          shell: hwclock --systohc

        - selinux: state=disabled

        - name: stop firewalld server 
          service: name=firewalld state=stopped enabled=false
        

        - name: install rabbitmq erlang package
          yum: name=https://packagecloud.io/rabbitmq/erlang/packages/el/7/erlang-20.3-1.el7.centos.x86_64.rpm/download.rpm state=present

        - name: install rabbitmq key file
          shell: rpm --import https://www.rabbitmq.com/rabbitmq-release-signing-key.asc

        - name: install socat
          yum: name=socat state=latest

        - name: install rabbitmq server
          yum: name=https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.7.8/rabbitmq-server-3.7.8-1.el7.noarch.rpm state=present

        - name: start rabbitmq server
          service: name=rabbitmq-server state=started enabled=true


        - name: enable plugins
          shell: rabbitmq-plugins enable rabbitmq_management

        - name: restart servier
          service: name=rabbitmq-server state=restarted


        - name: enable account
          shell: rabbitmqctl add_user admin admin123
          
        - name: grant account
          shell: rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"
          
        - name: setting account
          shell: rabbitmqctl set_user_tags admin administrator

```