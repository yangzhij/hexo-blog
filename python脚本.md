---
title: python脚本
date: 2019-01-16 10:18:52
tags:
---

```
import os
import time
import sys
import shutil
import re
import subprocess

#交互输入   python2.7用raw_input     python3用input         .strip()  去除首尾指定的字符，默认空格或换行符
domain = input("输入要配置的主域名(例如:huihuang300.com,test101.com):").strip()
root = input("输入站点目录(例如:bokeqipai):").strip()
web_mobile = input("输入H5游戏文件夹名称(例如:web-mobile-boke):").strip()

#生成需要的域名信息
agent="agent." + domain
game="game." + domain
g="g." + domain
www="www." + domain
m="m." + domain
lobby="lobby." + domain

print("\n请确认好以上的配置信息: \n"            #\n  换行           \t  回车                # 同时打印多个变量要拼接  +
      + "主域名:%s \t"%domain
      + "门户站点目录:%s \t"%root
      + "H5游戏文件夹:%s \n"%web_mobile
      + "以下nginx配置信息为脚本自动创建 \n"
      + "代理域名配置:%s \n"%agent
      + "PC游戏域名配置:%s \n"%game
      + "H5游戏域名配置:%s \n"%g
      + "PC域名配置:%s \n"%www
      + "H5域名配置:%s \n"%m
      + "游戏大厅域名配置:%s \n\n"%lobby)
load = input("是否开始脚本配置(y/n):").strip()
if load == 'y' or load == 'Y':
    pass
else:
    sys.exit(1)

#此信息不需要更改
conf_d = "/home/nginx/conf/conf.d/"
wwwroot="/home/nginx/data/"
home="/home/nginx/data"
src_web_mobile="web-mobile"
conf=".conf"

#判断是否存在conf_d定义的文件夹
if os.path.isdir(conf_d):                 #os.path.isdir()用于判断对象是否为一个目录       os.path.isfile()用于判断对象是否为一个文件
    print(conf_d + " folder exists")
else:
    print(conf_d + " folder doesn't exist")
    sys.exit(1)

#进入conf.d文件夹内
print("walk to " + conf_d)
os.chdir(conf_d)                        #os.chdir(path)    os.chdir() 方法用于改变当前工作目录到指定的路径，path 是要切换到的新路径。
print("Now dir is " + os.getcwd())                  #os.getcwd() 方法用于返回当前工作目录。


#生成agent配置文件
agent_nginx_file = agent + conf            #agent.test11.com.conf

if not os.path.exists(agent_nginx_file):               #os.path.exists()    判断文件是否存在
    print(agent_nginx_file + " File not exists")
    print("ready to create file ")

    agent_nginx_conf = '''

server {{
    listen 80;
    server_name {domain};
    root   /home/wwwroot/agentfront;
    index  index.htm index.html;


    location /api {{
        proxy_pass http://agent/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header REMOTE-HOST $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Connection  "";

        proxy_connect_timeout 600s;
        proxy_send_timeout 600s;
        proxy_read_timeout 600s;

        }}


    access_log  /var/log/nginx/{domain}.log;

}}


'''.format(domain=agent)

    f = open(agent_nginx_file,'w')
    f.write(agent_nginx_conf)
    f.close()

    print(agent_nginx_file + "create succeed !!!")

else:
    print(agent_nginx_file + " file already exists")
    pass
	

#生成gmae配置文件
game_nginx_file = game + conf
if not os.path.exists(game_nginx_file):
    print(game_nginx_file + ' file not exists')
    print(game_nginx_file + ' willd be create and update')

    game_nginx_conf = '''

server {{

    listen 80;
    server_name {domain}; 

    root /home/wwwroot/{webdir};
    index index.html index.htm; 

     
    access_log  /var/log/nginx/{domain}.log;
     
}}

'''.format(domain=game,webdir=web_mobile)

    f = open(game_nginx_file,'w')
    f.write(game_nginx_conf)
    f.close()

    print(game_nginx_file + ' update')

else:
    print(game_nginx_file + ' already exists ~')
    pass


#生成g配置文件
g_nginx_file = g + conf

if not os.path.exists(g_nginx_file):
    print(g_nginx_file + " file not exists")
    g_nginx_conf = '''


server {{
    listen 80;
    server_name {domain};  
    root  /home/wwwroot/{webdir};
    index index.htm index.html;

    access_log /var/log/nginx/{domain}.log main;

}}


'''.format(domain=g,webdir=web_mobile)


    f = open(g_nginx_file,'w')
    f.write(g_nginx_conf)
    f.close()

    print(g_nginx_file + " update")

else:
    print(g_nginx_file + " file exists~~~")
    pass



#生成门户网站配置
www_nginx_file = www + conf

if not os.path.exists(www_nginx_file):
    print(www_nginx_file + ' file not exists ')
    print(www_nginx_file + ' file ready to create  and update')

    www_nginx_conf = '''

server {{
    listen 80;
    server_name {domain} {wdomain};
    root   /home/wwwroot/{webdir}/pc/dist;
    index  index.htm index.html;


    location / {{
        root   /home/wwwroot/{webdir}/pc/dist;
        index  index.html index.htm index.php;

        if (!-e $request_filename){{
            rewrite ^/(.*)$ /index.html?s=/$1 last;
            }}
    }}



    location /api {{
        proxy_pass http://portal/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header REMOTE-HOST $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_next_upstream error timeout invalid_header http_502 http_504;


        proxy_connect_timeout 1800s;
        proxy_read_timeout 1800s;
        proxy_send_timeout 1800s;
    }}

    access_log /var/log/nginx/{webdir}.log;


}}

'''.format(domain=domain,wdomain=www,mdomain=m,webdir=root)

    f = open(www_nginx_file,'w')
    f.write(www_nginx_conf)
    f.close()

    print(www_nginx_file + ' update ')

else:
    print(www_nginx_file + ' file already exists ')
    pass



#生成h5门户网站配置
m_nginx_file = m + conf

if not os.path.exists(m_nginx_file):
    print(m_nginx_file + ' file not exists.')
    print(m_nginx_file + ' file ready to create and update')
    m_nginx_conf = '''

server {{
    listen 80;
    server_name {domain};

    root   /home/wwwroot/{webdir}/pc/dist;
    index  index.htm index.html;

    location / {{
        root   /home/wwwroot/{webdir}/pc/dist;
        index  index.htm index.html;

        if (!-e $request_filename){{
            rewrite ^/(.*)$ /index.html?s=/$1 last;
        }}


        location /api {{
            proxy_pass http://portal/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header REMOTE-HOST $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

            proxy_connect_timeout 1800s;
            proxy_read_timeout 1800s;
            proxy_send_timeout 1800s;

        }}
    
    }}

    access_log /var/log/nginx/{domain}.log;
}}

'''.format(domain=m,wdomain=www,webdir=root)

    f = open(m_nginx_file,'w')
    f.write(m_nginx_conf)
    f.close()

    print(m_nginx_file + ' update')


else:
    print(m_nginx_file + ' file already exists')
    pass



#生成lobby 配置文件
lobby_nginx_file = lobby + conf

if not os.path.exists(lobby_nginx_file):
    print(lobby_nginx_file + ' file not exisis')
    print(lobby_nginx_file + ' ready to create and update')

    lobby_nginx_conf = '''

server {{

    listen       80;
    server_name {domain};

    location / {{

    proxy_redirect off;
    proxy_pass http://lobby;
    proxy_set_header Host $host;
    proxy_set_header X-Real_IP $remote_addr;
    proxy_set_header X-Forwarded-For $remote_addr:$remote_port;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "";

    proxy_next_upstream error timeout invalid_header http_502 http_504;

    proxy_connect_timeout 1800s;
    proxy_read_timeout 1800s;
    proxy_send_timeout 1800s;

    }}

    access_log /var/log/nginx/{domain}.log;
}}



server {{

    listen 7010;
    server_name {domain};

    location / {{

        proxy_pass http://cat;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "";
        proxy_set_header X-real-ip $remote_addr;
        proxy_set_header X-Forwarded-For $remote_addr;

        proxy_connect_timeout 1800s;
        proxy_read_timeout 1800s;
        proxy_send_timeout 1800s;
    }}


    access_log /var/log/nginx/{domain}.log;
}}

'''.format(domain=lobby)

    f = open(lobby_nginx_file,'w')
    f.write(lobby_nginx_conf)
    f.close()

    print(lobby_nginx_file + ' update')

else:
    print(lobby_nginx_file + ' already exists ~')
    pass



print('walk to %s' %home)
os.chdir(home)               #os.chdir() 方法用于改变当前工作目录到指定的路径。  此处home要切换到的新路径。
print('now in '+ os.getcwd())                # os.getcwd()   获取当前目录


if os.path.isdir(wwwroot+root):
    print("存在%s文件夹"%root)
else:
    print("文件夹%s不存在"%root)
    print("创建文件夹%s"%root)
    os.mkdir(wwwroot + root)
    print("%s 文件夹创建完成"%root)



#拷贝mobile到wwwroot目录下
if os.path.exists(src_web_mobile):           #os.path.exists()    判断文件是否存在
    print(src_web_mobile + ' folder exists')
    print('ready copy to ' + web_mobile)
    try:
        shutil.copytree(src_web_mobile, wwwroot + web_mobile)
    except OSError as e:
        print(e)
        pass
    print( web_mobile + ' already copy to ' + wwwroot)

else:
    print(src_web_mobile + ' folder not exists')
    #pass



print("替换大厅信息")

js = "/res/raw-assets/CustomMade/customer_config.json"


#替换H5游戏包里面的json信息

old_base_url = re.compile(r'"HTTP_BASE_URL": "http://.*?/"')
new_base_url = '"HTTP_BASE_URL": "http://%s/"' %lobby

print(new_base_url)

old_ws_url = re.compile(r'"WEBSOCKET_BASE_URL": "ws://.*?/ws"')
new_ws_url ='"WEBSOCKET_BASE_URL": "ws://%s:7010/ws"'%lobby

print(new_ws_url)


oldurl = re.compile(r'lobby.*?\.com')

jsonfile = web_mobile + js

print(jsonfile)
if os.path.isfile(jsonfile):
    print("json 文件存在,准备替换配置 \n")
    time.sleep(1)
    print("正在替换中.....")
    with open(jsonfile,"r") as f:
        lines = f.readlines()
        #写的方式打开文件
    with open(jsonfile,"w") as f_w:
        for line in lines:
            line = re.sub(oldurl,lobby,line)                    #re.sub，实现正则的替换。
            line = re.sub(old_base_url,new_base_url,line)
            line = re.sub(old_ws_url,new_ws_url,line)
            f_w.write(line)

    print("%s 文件替换完成"%web_mobile)


else:
    print("json文件不存在,去%s 手工检查下"%web_mobile)



loads = input("配置完成 是否测试ngin配置(y/n):")
if loads == 'Y' or loads == 'y':
    print("正在测试nginx配置")
    time.sleep(1)
    cmd = "docker exec -it nginx /bin/bash -c '/usr/sbin/nginx -t'"
    test = subprocess.Popen(cmd,shell=True,stdout=subprocess.PIPE).communicate()[0]
    if 'ok' in test:
        re = input("nginx 配置测试通过 是否重加载nginx进程(y/n):")
        if re == 'y' or re == 'Y':
            cmd = "docker exec -it nginx /bin/bash -c '/usr/sbin/nginx -s reload'"
            os.system(cmd)
    else:
        print(test)
        print("检查配置 手工重启")
else:
    sys.exit(0)

	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	

```