---
title: Docker集中化web管理平台
date: 2018-12-26 01:45:50
tags:
---
*--本次部署参考文档：https://www.cnblogs.com/kevingrace/p/6867820.html*

**关闭防火墙**
```
[root@localhost ~]# firewall-cmd --state
not running
[root@localhost ~]# systemctl stop firewalld.service
```

**安装docker** *并修改docker配置文件，在文件末尾添加下面一行，进行docker加速设置*
```
[root@localhost ~]# yum -y install docker
[root@localhost ~]# vim /etc/sysconfig/docker
ADD_REGISTRY='--add-registry xxx.mirror.aliyuncs.com'
```
**启动docker服务**
```
[root@localhost ~]# systemctl start docker
```

**下载docker相关镜像**
```
[root@localhost ~]# docker pull rethinkdb
[root@localhost ~]# docker images
[root@localhost ~]# docker pull microbox/etcd
[root@localhost ~]# docker pull shipyard/docker-proxy
[root@localhost ~]# docker pull swarm
[root@localhost ~]# docker pull dockerclub/shipyard
```

**执行官方一键部署脚本**
```
[root@localhost ~]# vi shipyard-deploy     #文件已上传github，将里面的文件内容复制即可;本文末尾也有
[root@localhost ~]# chmod 755 shipyard-deploy
此处注意，默认web访问路径是http://ip:8080，如果要修改默认端口，则执行：
[root@localhost ~]# sed -i 's/8080/8283/g' shipyard-deploy        //端口随意不冲突即可，此处是改为8283
[root@localhost ~]# sh shipyard-deploy              //执行脚本
```
执行完毕后，通过docker ps命令就可以看到相应的shipyard容器已经创建好了，然后浏览器访问

**访问**
http://172.20.100.162:8283
初始用户名/密码：admin/shipyard


**添加节点**
Shipyard添加其他节点主机(centos7.X系统的主机)的步骤如下：
1.上传部署脚本shipyard-deploy到被加入的节点，如172.20.100.115
2.在被添加的节点上执行操作，注意下面etcd地址要写成shipyard部署机的ip地址
```
[root@localhost ~]# scp shipyard-deploy 172.20.100.115:/home/           #将脚本传给115 
#在172.20.100.115上执行                  
[root@myself home]# cat shipyard-deploy| ACTION=node DISCOVERY=etcd://172.20.100.162:4001 bash
[root@myself home]# docker ps       #查看 
然后后登录http://172.16.60.218:8283就可以看到新添加的节点了
```
**删除节点**
```
[root@myself home]#cat shipyard-deploy |ACTION=remove bash -s          #在要删除的节点上操作，如删除115节点
```
**<font color=red>如果在初期没有更改端口，后来在使用过程中想修改了，则更改完端口后，先删除Shipyard环境，然后再重新部署即可，不会对之前的操作有任何影响。**</font>
```
[root@localhost ~]# sed -i 's/8080/8283/g' shipyard-deploy 
[root@localhost ~]# cat shipyard-deploy |ACTION=remove bash                //删除Shipyard环境
[root@localhost ~]# sh shipyard-deploy                                     //执行部署脚本
```



####################################**官方一键部署脚本**#############################
vi shipyard-deploy
```
#!/bin/bash
# Author 	: dockerweb By 2018-09-07
# Description	: Auto deploy shipyard 
# Email		: dockerweb@126.com

if [ "$1" != "" ] && [ "$1" = "-h" ]; then
    echo "Shipyard Deploy uses the following environment variables:"
    echo "  ACTION: this is the action to use (deploy, upgrade, node, remove)"
    echo "  DISCOVERY: discovery system used by Swarm (only if using 'node' action)"
    echo "  IMAGE: this overrides the default Shipyard image"
    echo "  PREFIX: prefix for container names"
    echo "  SHIPYARD_ARGS: these are passed to the Shipyard controller container as controller args"
    echo "  TLS_CERT_PATH: path to certs to enable TLS for Shipyard"
    echo "  PORT: specify the listen port for the controller (default: 8080)"
    echo "  IP: specify the address at which the controller or node will be available (default: eth0 ip)"
    echo "  PROXY_PORT: port to run docker proxy (default: 2375)"
    exit 1
fi

if [ -z "`which docker`" ]; then
    echo "You must have the Docker CLI installed on your \$PATH"
    echo "  See http://docs.docker.com for details"
    exit 1
fi

ACTION=${ACTION:-deploy}
IMAGE=${IMAGE:-dockerclub/shipyard:latest}
PREFIX=${PREFIX:-shipyard}
SHIPYARD_ARGS=${SHIPYARD_ARGS:-""}
TLS_CERT_PATH=${TLS_CERT_PATH:-}
CERT_PATH="/etc/shipyard"
PROXY_PORT=${PROXY_PORT:-2375}
SWARM_PORT=3375
SHIPYARD_PROTOCOL=http
SHIPYARD_PORT=${PORT:-8080}
SHIPYARD_IP=${IP}
DISCOVERY_BACKEND=etcd
DISCOVERY_PORT=4001
DISCOVERY_PEER_PORT=7001
ENABLE_TLS=0
CERT_FINGERPRINT=""
LOCAL_CA_CERT=""
LOCAL_SSL_CERT=""
LOCAL_SSL_KEY=""
LOCAL_SSL_CLIENT_CERT=""
LOCAL_SSL_CLIENT_KEY=""
SSL_CA_CERT=""
SSL_CERT=""
SSL_KEY=""
SSL_CLIENT_CERT=""
SSL_CLIENT_KEY=""

show_cert_help() {
    echo "To use TLS in Shipyard, you must have existing certificates."
    echo "The certs must be named ca.pem, server.pem, server-key.pem, cert.pem and key.pem"
    echo "If you need to generate certificates, see https://github.com/ehazlett/certm for examples."
}

check_certs() {
    if [ -z "$TLS_CERT_PATH" ]; then
        return
    fi

    if [ ! -e $TLS_CERT_PATH ]; then
        echo "Error: unable to find certificates in $TLS_CERT_PATH"
        show_cert_help
        exit 1
    fi

    if [ "$PROXY_PORT" = "2375" ]; then
        PROXY_PORT=2376
    fi
    SWARM_PORT=3376
    SHIPYARD_PROTOCOL=https
    LOCAL_SSL_CA_CERT="$TLS_CERT_PATH/ca.pem"
    LOCAL_SSL_CERT="$TLS_CERT_PATH/server.pem"
    LOCAL_SSL_KEY="$TLS_CERT_PATH/server-key.pem"
    LOCAL_SSL_CLIENT_CERT="$TLS_CERT_PATH/cert.pem"
    LOCAL_SSL_CLIENT_KEY="$TLS_CERT_PATH/key.pem"
    SSL_CA_CERT="$CERT_PATH/ca.pem"
    SSL_CERT="$CERT_PATH/server.pem"
    SSL_KEY="$CERT_PATH/server-key.pem"
    SSL_CLIENT_CERT="$CERT_PATH/cert.pem"
    SSL_CLIENT_KEY="$CERT_PATH/key.pem"
    CERT_FINGERPRINT=$(openssl x509 -noout -in $LOCAL_SSL_CERT -fingerprint -sha256 | awk -F= '{print $2;}')

    if [ ! -e $LOCAL_SSL_CA_CERT ] || [ ! -e $LOCAL_SSL_CERT ] || [ ! -e $LOCAL_SSL_KEY ] || [ ! -e $LOCAL_SSL_CLIENT_CERT ] || [ ! -e $LOCAL_SSL_CLIENT_KEY ]; then
        echo "Error: unable to find certificates"
        show_cert_help
        exit 1
    fi

    ENABLE_TLS=1
}

# container functions
start_certs() {
    ID=$(docker run \
        -ti \
        -d \
        --restart=always \
        --name $PREFIX-certs \
        -v $CERT_PATH \
        alpine \
        sh)
    if [ $ENABLE_TLS = 1 ]; then
        docker cp $LOCAL_SSL_CA_CERT $PREFIX-certs:$SSL_CA_CERT
        docker cp $LOCAL_SSL_CERT $PREFIX-certs:$SSL_CERT
        docker cp $LOCAL_SSL_KEY $PREFIX-certs:$SSL_KEY
        docker cp $LOCAL_SSL_CLIENT_CERT $PREFIX-certs:$SSL_CLIENT_CERT
        docker cp $LOCAL_SSL_CLIENT_KEY $PREFIX-certs:$SSL_CLIENT_KEY
    fi
}

remove_certs() {
    docker rm -fv $PREFIX-certs > /dev/null 2>&1
}

get_ip() {
    if [ -z "$SHIPYARD_IP" ]; then
        SHIPYARD_IP=`docker run --rm --net=host alpine ip route get 8.8.8.8 | awk '{ print $7;  }'`
    fi
}

start_discovery() {
    get_ip

    ID=$(docker run \
        -ti \
        -d \
        -p 4001:4001 \
        -p 7001:7001 \
        --restart=always \
        --name $PREFIX-discovery \
        microbox/etcd:latest -addr $SHIPYARD_IP:$DISCOVERY_PORT -peer-addr $SHIPYARD_IP:$DISCOVERY_PEER_PORT)
}

remove_discovery() {
    docker rm -fv $PREFIX-discovery > /dev/null 2>&1
}

start_rethinkdb() {
    ID=$(docker run \
        -ti \
        -d \
        --restart=always \
        --name $PREFIX-rethinkdb \
        rethinkdb)
}

remove_rethinkdb() {
    docker rm -fv $PREFIX-rethinkdb > /dev/null 2>&1
}

start_proxy() {
    TLS_OPTS=""
    if [ $ENABLE_TLS = 1 ]; then
        TLS_OPTS="-e SSL_CA=$SSL_CA_CERT -e SSL_CERT=$SSL_CERT -e SSL_KEY=$SSL_KEY -e SSL_SKIP_VERIFY=1"
    fi
    # Note: we add SSL_SKIP_VERIFY=1 to skip verification of the client
    # certificate in the proxy image.  this will pass it to swarm that
    # does verify.  this helps with performance and avoids certificate issues
    # when running through the proxy.  ultimately if the cert is invalid
    # swarm will fail to return.
    ID=$(docker run \
        -ti \
        -d \
        -p $PROXY_PORT:$PROXY_PORT \
        --hostname=$HOSTNAME \
        --restart=always \
        --name $PREFIX-proxy \
        -v /var/run/docker.sock:/var/run/docker.sock \
        -e PORT=$PROXY_PORT \
        --volumes-from=$PREFIX-certs $TLS_OPTS\
        shipyard/docker-proxy:latest)
}

remove_proxy() {
    docker rm -fv $PREFIX-proxy > /dev/null 2>&1
}

start_swarm_manager() {
    get_ip

    TLS_OPTS=""
    if [ $ENABLE_TLS = 1 ]; then
        TLS_OPTS="--tlsverify --tlscacert=$SSL_CA_CERT --tlscert=$SSL_CERT --tlskey=$SSL_KEY"
    fi

    EXTRA_RUN_OPTS=""

    if [ -z "$DISCOVERY" ]; then
        DISCOVERY="$DISCOVERY_BACKEND://discovery:$DISCOVERY_PORT"
        EXTRA_RUN_OPTS="--link $PREFIX-discovery:discovery"
    fi
    ID=$(docker run \
        -ti \
        -d \
        --restart=always \
        --name $PREFIX-swarm-manager \
        --volumes-from=$PREFIX-certs $EXTRA_RUN_OPTS \
        swarm:latest \
        m --replication --addr $SHIPYARD_IP:$SWARM_PORT --host tcp://0.0.0.0:$SWARM_PORT $TLS_OPTS $DISCOVERY)
}

remove_swarm_manager() {
    docker rm -fv $PREFIX-swarm-manager > /dev/null 2>&1
}

start_swarm_agent() {
    get_ip

    if [ -z "$DISCOVERY" ]; then
        DISCOVERY="$DISCOVERY_BACKEND://discovery:$DISCOVERY_PORT"
        EXTRA_RUN_OPTS="--link $PREFIX-discovery:discovery"
    fi
    ID=$(docker run \
        -ti \
        -d \
        --restart=always \
        --name $PREFIX-swarm-agent $EXTRA_RUN_OPTS \
        swarm:latest \
        j --addr $SHIPYARD_IP:$PROXY_PORT $DISCOVERY)
}

remove_swarm_agent() {
    docker rm -fv $PREFIX-swarm-agent > /dev/null 2>&1
}

start_controller() {
    #-v $CERT_PATH:/etc/docker:ro \
    TLS_OPTS=""
    if [ $ENABLE_TLS = 1 ]; then
        TLS_OPTS="--tls-ca-cert $SSL_CA_CERT --tls-cert=$SSL_CERT --tls-key=$SSL_KEY --shipyard-tls-ca-cert=$SSL_CA_CERT --shipyard-tls-cert=$SSL_CERT --shipyard-tls-key=$SSL_KEY"
    fi

    ID=$(docker run \
        -ti \
        -d \
        --restart=always \
        --name $PREFIX-controller \
        --link $PREFIX-rethinkdb:rethinkdb \
        --link $PREFIX-swarm-manager:swarm \
        -p $SHIPYARD_PORT:$SHIPYARD_PORT \
        --volumes-from=$PREFIX-certs \
        $IMAGE \
        --debug \
        server \
        --listen :$SHIPYARD_PORT \
        -d tcp://swarm:$SWARM_PORT $TLS_OPTS $SHIPYARD_ARGS)
}

wait_for_available() {
    set +e 
    IP=$1
    PORT=$2
    echo Waiting for Shipyard on $IP:$PORT

    docker pull ehazlett/curl > /dev/null 2>&1

    TLS_OPTS=""
    if [ $ENABLE_TLS = 1 ]; then
        TLS_OPTS="-k"
    fi

    until $(docker run --rm ehazlett/curl --output /dev/null --connect-timeout 1 --silent --head --fail $TLS_OPTS $SHIPYARD_PROTOCOL://$IP:$PORT/ > /dev/null 2>&1); do
        printf '.'
        sleep 1 
    done
    printf '\n'
}

remove_controller() {
    docker rm -fv $PREFIX-controller > /dev/null 2>&1
}

if [ "$ACTION" = "deploy" ]; then
    set -e

    check_certs

    get_ip 

    echo "Deploying Shipyard"
    echo " -> Starting Database"
    start_rethinkdb
    echo " -> Starting Discovery"
    start_discovery
    echo " -> Starting Cert Volume"
    start_certs
    echo " -> Starting Proxy"
    start_proxy
    echo " -> Starting Swarm Manager"
    start_swarm_manager
    echo " -> Starting Swarm Agent"
    start_swarm_agent
    echo " -> Starting Controller"
    start_controller

    wait_for_available $SHIPYARD_IP $SHIPYARD_PORT

    echo "Shipyard available at $SHIPYARD_PROTOCOL://$SHIPYARD_IP:$SHIPYARD_PORT"
    if [ $ENABLE_TLS = 1 ] && [ ! -z "$CERT_FINGERPRINT" ]; then
        echo "SSL SHA-256 Fingerprint: $CERT_FINGERPRINT"
    fi
    echo "Username: admin Password: shipyard"

elif [ "$ACTION" = "node" ]; then
    set -e

    if [ -z "$DISCOVERY" ]; then
        echo "You must set the DISCOVERY environment variable"
        echo "with the discovery system used with Swarm"
        exit 1
    fi

    check_certs

    echo "Adding Node"
    echo " -> Starting Cert Volume"
    start_certs
    echo " -> Starting Proxy"
    start_proxy
    echo " -> Starting Swarm Manager"
    start_swarm_manager $DISCOVERY
    echo " -> Starting Swarm Agent"
    start_swarm_agent

    echo "Node added to Swarm: $SHIPYARD_IP"
    
elif [ "$ACTION" = "upgrade" ]; then
    set -e

    check_certs

    get_ip

    echo "Upgrading Shipyard"
    echo " -> Pulling $IMAGE"
    docker pull $IMAGE

    echo " -> Upgrading Controller"
    remove_controller
    start_controller

    wait_for_available $SHIPYARD_IP $SHIPYARD_PORT

    echo "Shipyard controller updated"

elif [ "$ACTION" = "remove" ]; then
    # ignore errors
    set +e

    echo "Removing Shipyard"
    echo " -> Removing Database"
    remove_rethinkdb
    echo " -> Removing Discovery"
    remove_discovery
    echo " -> Removing Cert Volume"
    remove_certs
    echo " -> Removing Proxy"
    remove_proxy
    echo " -> Removing Swarm Agent"
    remove_swarm_agent
    echo " -> Removing Swarm Manager"
    remove_swarm_manager
    echo " -> Removing Controller"
    remove_controller

    echo "Done"
else
    echo "Unknown action $ACTION"
    exit 1
fi
```