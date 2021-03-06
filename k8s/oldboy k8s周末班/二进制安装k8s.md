# 第一章：kubernetes概述

> 官网：[https://kubernetes.io](https://kubernetes.io/)
> GitHub：https://github.com/kubernetes/kubernetes
> 由来：谷歌的Borg系统，后经Go语言重写并捐给CNCF基金会开源
> 含义：词根源于希腊语：舵手/飞行员。K8S -> K12345678S
> 重要作用：开源的容器编排框架工具(生态极其丰富)
> 学习的意义：解决跑裸docker的若干痛点

kubernetes优势：

```shell
自动装箱，水平扩展，自我修复
服务发现和负载均衡
自动发布(默认滚动发布模式)和回滚
集中化配置管理和秘钥管理
存储编排
任务批量处理运行
....
```

# 第二章：kubernetes快速入门

## 1.四组基本概念

### 1.Pod/Pod控制器

Pod

```shell
Pod是K8S里能够被运行的做小的逻辑单元(原子单元)
1个Pod里面可以运行多个容器，他们共享UTS+NET+IPC名称空间
可以把Pod理解成豌豆荚，而同一个Pod内的每个容器是一颗颗豌豆
一个Pod里运行多个容器，又叫：边车(SideCar)模式
```

Pod控制器

```shell
Pod控制器是Pod启动的一种模板，用来保证在K8S里启动的Pod应始终按照人们的预期运行(副本数、生命周期、健康状态检查...)

K8S内提供了众多的Pod控制器，常用的有以下几种:
    Deployment
    DaemonSet
    ReplicaSet
    StatefulSet
    Job
    Cronjob
```

### 2.Name/Namespace

Name

```shell
由于K8S内部，使用“资源”来定义每一种逻辑概念(功能)故每种“资源”，都应该有自己的“名称”
“资源”有api版本(apiVersion)类别(Kind)、元数据(matadata)、定义清单(spec)、状态(status)等配置信息
“名称”通常定义在“资源”的“元数据”信息里
```

Namespace

```shell
随着项目增多、人员增加、集群规模的扩大，需要一种能够隔离K8S内各种“资源”的方法，这就是名称空间
名称空间可以理解为K8S内部的虚拟集群组
不同名称空间内的“资源”，名称可以相同，相同名称空间内的同种“资源”，“名称”不能相同
合理的使用K8S的名称空间，使得集群管理员能够更好的对交付到K8S里的服务进行分类管理和浏览
K8S里默认存在的名称空间有：default、kube-system、kube-public
查询K8S里特定“资源”要带上相应的名称空间
```

### 3.Label/Label选择器

Label

```shell
标签是k8s特色的管理方式，便于分类管理资源对象。
一个标签可以对应多个资源，一个资源也可以有多个标签，它们是多对多的关系。
一个资源拥有多个标签，可以实现不同维度的管理。
标签的组成：key=value
与标签类似的，还有一种“注解”(annotations)
```

Label选择器

```shell
给资源打上标签后，可以使用标签选择器过滤指定的标签
标签选择器目前有两个：基于等值关系(等于、不等于)和基于集合关系(属于、不属于，存在)
许多资源支持内嵌标签选择器字段
    matchLabels
    matchExpressions
```

### 4.Service/Ingress

Service

```shell
在K8S的世界里，虽然每个Pod都会被分配一个单独的IP地址，但这个IP地址会随着Pod的销毁而消失
Service(服务)就是用来解决这个问题的核心概念
一个Service可以看作一组提供相同服务的Pod的对外访问接口
Service作用于哪些Pod是通过标签选择器来定义的
```

Ingress

```shell
Ingress是K8S集群里工作在OSI网络参考模型下，第7层的应用，对外暴露的接口
Service只能进行L4流量调度，表现形式是ip+port
Ingress则可以调度不同业务域，不同URL访问路径的业务流量
```

## 2.K8S的组成

```shell
核心组件
    配置存储中心 -> etcd服务
    主控(Master)节点
        kube-apiserver服务
        kube-controller-manager服务
        kube-scheduler服务
    运算(node)节点
        kube-kubelet服务
        kube-proxy服务

CLI客户端
    kubectl

核心附件
    CNI网络插件 -> flannel/calico
    服务发现用插件 -> coredns
    服务暴露用插件 -> traefik
    GUI管理插件 -> Dashboard
```

apiserver：

```shell
    提供了集群管理的REST API接口(包括鉴权、数据校验及集群状态变更)
    负责其他模块之间的数据交互，承担通信枢纽功能
    是资源配额控制的入口
    提供完备的集群安全机制
```

controller-manager：

```shell
    由一系列控制器组成，通过apiserver监控整个集群的状态，并确保集群处于预期的工作状态
    Node Controller                    节点控制
    Deployment Controller           pod控制器
    Service Controller                服务控制器
    Volume Controller                   存储卷控制器
    Endpoint Controller             接入点控制器
    Garbage Controller              垃圾控制器
    Namespace Controller            名称空间控制器
    Job Controller                    任务控制器
    Resource quta Controller        资源配额控制器
    ...    
```

scheduler:

```shell
    主要功能是接收调度pod到适合的运算节点上
    预算策略(predict)
    优选策略(priorities)
```

kubelet:

```shell
    简单的说，kubelet的主要功能就是定时从某个地方获取节点上pod的期望状态(运行什么容器、运行副本数量、网络或者存储如何配置等等)，并调用对应的容器平台接口达到这个状态。
    定时汇报当前节点的状态给apiserver，以供调度的时候使用
    镜像和容器的清理工作，保证节点上的镜像不会占满磁盘空间，退出的容器不会占用太多资源
```

kube-proxy：

```shell
    是K8S在每个节点上运行网络代理，service资源的载体
    建立了pod网络和集群网络的关系(clusterip -> podip)
    常用的三种流量调度模式：
        Userspace(废弃)
        Iptables(濒临泛滥)
        Ipvs(推荐)

    负责建立和删除包括更新调度规则、通知apiserver自己的更新，或者从apiserver的调度规则变化来更新自己的。
```

## 3.K8S三条网络详解

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_9f8b90aeb7332c704fcad1ca417d1b6e_r.png)

# 第三章：实验部署集群架构详解

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_f5d8c5028b894e4a2c1ca111eb6b7d27_r.png)

常见的K8S安装部署方式：

```shell
1. Minikube单节点微型K8S(仅供学习、预览使用)
2. 二进制安装部署(生产首选，新手推荐)
3. 使用Kuberadmin进行部署，K8S的部署工具，跑在K8S里(相对简单，熟手推荐)
```

# 第四章：部署k8s集群前准备工作

### 1.准备虚拟机

- 5台vm，每台4C8G

| 主机名               | 角色                     | ip              | 配置 | 部署服务                                                     |
| :------------------- | :----------------------- | :-------------- | :--- | :----------------------------------------------------------- |
| hdss184-241.host.com | k8s1代理节点1            | 192.168.184.241 | 4C8G | bind9,nginx(四层代理),keeplived,supervisor                   |
| hdss184-242.host.com | k8s1代理节点2            | 192.168.184.242 | 4C8G | etcd,nginx(四层代理),keeplived,supervisor                    |
| hdss184-243.host.com | k8s运算节点1             | 192.168.184.243 | 4C8G | etcd,kube-apiserver,kube-controller-manager,kube-scheduler,kube-kubelet,kube-proxy,supervisor |
| hdss184-244.host.com | k8s运算节点2             | 192.168.184.244 | 4C8G | etcd,kube-apiserver,kube-controller-manager,kube-scheduler,kube-kubelet,kube-proxy,supervisor |
| hdss184-245.host.com | k8s运维节点(docker仓库） | 192.168.184.245 | 4C8G | 证书服务，docker私有仓库harbor，nginx代理harbor，pause       |

查看系统版本

```shell
~]# uname -a
Linux hdss184.host.com 3.10.0-693.21.1.el7.x86_64 #1 SMP Wed Mar 7 19:03:37 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```

### 2.调整操作系统

所有机器上：

1.设置主机名

```shell
~]# hostnamectl set-hostname hdss184-241.host.com
~]# hostnamectl set-hostname hdss184-242.host.com
~]# hostnamectl set-hostname hdss184-243.host.com
~]# hostnamectl set-hostname hdss184-244.host.com
~]# hostnamectl set-hostname hdss184-245.host.com
```

2.关闭selinux和关闭防火墙

```shell
~]# sed -i 's#SELINUX=enforcing#SELINUX=disabled#g' /etc/selinux/config
~]# setenforce 0

~]# systemctl stop firewalld
~]# systemctl  disable firewalld
```

3.安装epel-release

```shell
~]# yum install -y epel-release
```

4.安装必工具

```shell
~]# yum install wget net-tools telnet tree nmap sysstat lrzsz dos2unix bind-utils -y
```

### 3.DNS服务初始化

hdss184-241.host.com上：

#### 1.安装bind9软件

```shell
[root@hdss184-241 ~]# yum install bind -y
=====================================================================================================================================
 Package                                  Arch                     Version                              Repository              Size
=====================================================================================================================================
Installing:
 bind                                     x86_64                   32:9.11.4-9.P2.el7                   base                   2.3 M
```

#### 2.配置bind9

主配置文件

```shell
[root@hdss184-241 ~]# vim /etc/named.conf

listen-on port 53 { 192.168.184.241; };
allow-query     { any; };
forwarders      { 192.168.184.2; };      #向上查询(增加一条)
dnssec-enable no;
dnssec-validation no;

[root@hdss184-241 ~]# named-checkconf   #检查配置文件
```

区域配置文件

```shell
[root@hdss184-241 ~]# vim /etc/named.rfc1912.zones

[root@hdss184-241 ~]# tail -12 /etc/named.rfc1912.zones

zone "host.com" IN {
        type  master;
        file  "host.com.zone";
        allow-update { 192.168.6.241; };
};

zone "od.com" IN {
        type  master;
        file  "od.com.zone";
        allow-update { 192.168.6.241; };
};
```

配置区域数据文件

- 配置主机域数据文件

```shell
[root@hdss184-241 ~]# vim /var/named/host.com.zone
$ORIGIN host.com.
$TTL 600    ; 10 minutes
@       IN SOA    dns.host.com. dnsadmin.host.com. (
                2019111201 ; serial
                10800      ; refresh (3 hours)
                900        ; retry (15 minutes)
                604800     ; expire (1 week)
                86400      ; minimum (1 day)
                )
            NS   dns.host.com.
$TTL 60    ; 1 minute
dns                A    192.168.6.241
hdss184-241          A    192.168.6.241
hdss184-242          A    192.168.6.242
hdss184-243          A    192.168.6.243
hdss184-244          A    192.168.6.244
hdss184-245          A    192.168.6.245
[root@hdss184-241 ~]# vim /var/named/od.com.zone
$ORIGIN od.com.
$TTL 600    ; 10 minutes
@           IN SOA    dns.od.com. dnsadmin.od.com. (
                2019111201 ; serial
                10800      ; refresh (3 hours)
                900        ; retry (15 minutes)
                604800     ; expire (1 week)
                86400      ; minimum (1 day)
                )
                NS   dns.od.com.
$TTL 60    ; 1 minute
dns                A    192.168.6.241
```

#### 3.启动bind9

```shell
[root@hdss184-241 ~]# named-checkconf
[root@hdss184-241 ~]# systemctl start named
[root@hdss184-241 ~]# systemctl enable named
```

#### 4.检查

```shell
[root@hdss184-245 ~]# dig -t A hdss184-244.host.com @192.168.184.241 +short
192.168.184.244
[root@hdss184-245 ~]# dig -t A hdss184-241.host.com @192.168.184.241 +short
192.168.184.241
[root@hdss184-245 ~]# dig -t A hdss184-242.host.com @192.168.184.241 +short
192.168.184.242
[root@hdss184-245 ~]# dig -t A hdss184-243.host.com @192.168.184.241 +short
192.168.184.243
[root@hdss184-245 ~]# dig -t A hdss184-244.host.com @192.168.184.241 +short
192.168.184.244
[root@hdss184-245 ~]# dig -t A hdss184-245.host.com @192.168.184.241 +short
192.168.184.245
```

#### 5.配置dns客户端

-- Linux主机上

```shell
 # vi  /etc/sysconfig/network-scripts/ifcfg-ens33
 修改 DNS1=192.168.184.241
 
 # systemctl  restart network
 
~]# cat /etc/resolv.conf 
# Generated by NetworkManager
search host.com
nameserver 192.168.184.241



```

- windows主机上

> 网络和共享中心 -> 网卡设置 -> 设置DNS服务器
> 如有必要，还应设置虚拟网卡的接口地跃点数为：10

#### 6.检查

```shell
[root@hdss184-245 ~]# ping hdss184-241
PING hdss184-241.host.com (192.168.6.241) 56(84) bytes of data.
64 bytes from node1.98yz.cn (192.168.6.241): icmp_seq=1 ttl=64 time=0.213 ms
^C
--- hdss184-241.host.com ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.213/0.213/0.213/0.000 ms
[root@hdss184-245 ~]# ping hdss184-241.host.com
PING hdss184-241.host.com (192.168.6.241) 56(84) bytes of data.
64 bytes from node1.98yz.cn (192.168.6.241): icmp_seq=1 ttl=64 time=0.136 ms
^C
--- hdss184-241.host.com ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.136/0.136/0.136/0.000 ms
[C:\Users\mcake]$ ping dns.od.com

正在 Ping dns.od.com [192.168.6.241] 具有 32 字节的数据:
来自 192.168.6.241 的回复: 字节=32 时间<1ms TTL=63
来自 192.168.6.241 的回复: 字节=32 时间<1ms TTL=63
```

### 4.准备自签证书

运维主机hdss184-245.host.com上：

#### 1.安装CFSSL

- 证书签发工具CFSSL:R1.2

[cfssl下载地址](https://pkg.cfssl.org/R1.2/cfssl_linux-amd64)

[cfssl-json下载地址](https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64)

[cfssl-certinfo下载地址](https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64)

```shell
[root@hdss184-245 ~]# wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 -O /usr/bin/cfssl
[root@hdss184-245 ~]# wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64 -O /usr/bin/cfssl-json
[root@hdss184-245 ~]# wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64 -O /usr/bin/cfssl-certinfo
[root@hdss184-245 ~]# chmod +x /usr/bin/cfssl*
```

#### 2.创建生成CA证书签名请求(csr)的JSON配置文件

```shell
[root@hdss184-245 ~]# mkdir /opt/certs
[root@hdss184-245 ~]# vim /opt/certs/ca-csr.json
[root@hdss184-245 ~]# cat /opt/certs/ca-csr.json
{
    "CN": "OldboyEdu",
    "hosts": [
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "beijing",
            "L": "beijing",
            "O": "od",
            "OU": "ops"
        }
    ],
    "ca": {
        "expiry": "175200h"
    }
}
```

> CN:Common Name，浏览器使用该字段验证网站是否合法，一般写的是域名。非常重要。
> C：Country。国家
> ST：State，州，省
> L：Locality，城区，城市
> O：Organization Name，组织名称，公司名称
> OU：Organization Unit Name。组织单位名称，公司部门

#### 3.生成CA证书和私钥

```shell
 cd /opt/certs

[root@hdss184-245 certs]# cfssl gencert -initca ca-csr.json | cfssl-json -bare ca
2019/11/12 16:31:15 [INFO] generating a new CA key and certificate from CSR
2019/11/12 16:31:15 [INFO] generate received request
2019/11/12 16:31:15 [INFO] received CSR
2019/11/12 16:31:15 [INFO] generating key: rsa-2048
2019/11/12 16:31:16 [INFO] encoded CSR
2019/11/12 16:31:16 [INFO] signed certificate with serial number 165156553242987548447967502951541624956409280173
[root@hdss184-245 certs]# ll
total 16
-rw-r--r-- 1 root root  993 Nov 12 16:31 ca.csr
-rw-r--r-- 1 root root  328 Nov 12 16:06 ca-csr.json
-rw------- 1 root root 1679 Nov 12 16:31 ca-key.pem
-rw-r--r-- 1 root root 1346 Nov 12 16:31 ca.pem
```

### 5.部署docker环境

在hdss184-243、hdss184-244、hdss184-245上：

#### 1.安装

```shell
[root@hdss184-243 ~]# curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun

[root@hdss184-244 ~]# curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun

[root@hdss184-245 ~]# curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

#### 2.配置

```shell
[root@hdss184-243 ~]# mkdir -p /etc/docker /data/docker
[root@hdss184-243 ~]# vim /etc/docker/daemon.json

{
  "graph": "/data/docker",
  "storage-driver": "overlay2",
  "insecure-registries": ["registry.access.redhat.com","quay.io","harbor.od.com"],
  "registry-mirrors": ["https://q2gr04ke.mirror.aliyuncs.com"],
  "bip": "172.6.243.1/24",
  "exec-opts": ["native.cgroupdriver=systemd"],
  "live-restore": true
}
[root@hdss184-244 ~]#  mkdir -p /etc/docker /data/docker
[root@hdss184-244 ~]# vim /etc/docker/daemon.json

{
  "graph": "/data/docker",
  "storage-driver": "overlay2",
  "insecure-registries": ["registry.access.redhat.com","quay.io","harbor.od.com"],
  "registry-mirrors": ["https://q2gr04ke.mirror.aliyuncs.com"],
  "bip": "172.6.244.1/24",
  "exec-opts": ["native.cgroupdriver=systemd"],
  "live-restore": true
}
[root@hdss184-245 ~]#  mkdir -p /etc/docker /data/docker
[root@hdss184-245 ~]# vim /etc/docker/daemon.json

{
  "graph": "/data/docker",
  "storage-driver": "overlay2",
  "insecure-registries": ["registry.access.redhat.com","quay.io","harbor.od.com"],
  "registry-mirrors": ["https://q2gr04ke.mirror.aliyuncs.com"],
  "bip": "172.6.245.1/24",
  "exec-opts": ["native.cgroupdriver=systemd"],
  "live-restore": true
}
```

#### 3.启动

在hdss184-243、hdss184-244、hdss184-245上操作

```shell
~]# systemctl start docker
~]# systemctl enable docker
```



### 6、部署docker镜像私有仓库harbor

hdss184-245上：

#### 1.下载软件二进制包并解压

[harbor官方github地址](https://github.com/goharbor/harbor)

[harbor下载地址](https://storage.googleapis.com/harbor-releases/release-1.8.0/harbor-offline-installer-v1.8.3.tgz)

```shell
[root@hdss184-245 ~]# mkdir -p /opt/src/harbor
[root@hdss184-245 ~]# cd /opt/src/harbor
[root@hdss184-245 harbor]# wget https://storage.googleapis.com/harbor-releases/release-1.8.0/harbor-offline-installer-v1.8.3.tgz
[root@hdss184-245 harbor]# tar xvf harbor-offline-installer-v1.8.3.tgz -C /opt
harbor/harbor.v1.8.3.tar.gz
harbor/prepare
harbor/LICENSE
harbor/install.sh
harbor/harbor.yml

[root@hdss184-245 harbor]# mv /opt/harbor /opt/harbor-v1.8.3
[root@hdss184-245 harbor]# ln -s /opt/harbor-v1.8.3 /opt/harbor
```

#### 2.配置

```shell
/opt/harbor/harbor.yml

hostname: harbor.od.com
http:
  port: 180
harbor_admin_password: Harbor12345
data_volume: /data/harbor
log:
  level: info
  rotate_count: 50
  rotate_size: 200M
  location: /data/harbor/logs

mkdir -p /data/harbor/logs
```

#### 3.安装docker-compose

运维主机hdss184-245.host.com上：

```shell
[root@hdss184-245 harbor]# yum install docker-compose -y
[root@hdss184-245 harbor]# rpm -qa docker-compose
docker-compose-1.18.0-4.el7.noarch
```

#### 4.安装harbor

```shell
[root@hdss184-245 harbor]# ll
total 569632
-rw-r--r-- 1 root root 583269670 Sep 16 11:53 harbor.v1.8.3.tar.gz
-rw-r--r-- 1 root root      4526 Nov 13 11:35 harbor.yml
-rwxr-xr-x 1 root root      5088 Sep 16 11:53 install.sh
-rw-r--r-- 1 root root     11347 Sep 16 11:53 LICENSE
-rwxr-xr-x 1 root root      1654 Sep 16 11:53 prepare
[root@hdss184-245 harbor]# sh install.sh
```

#### 5.检查harbor启动情况

```shell
[root@hdss184-245 harbor]# docker-compose ps
      Name                     Command               State             Ports          
--------------------------------------------------------------------------------------
harbor-core         /harbor/start.sh                 Up                               
harbor-db           /entrypoint.sh postgres          Up      5432/tcp                 
harbor-jobservice   /harbor/start.sh                 Up                               
harbor-log          /bin/sh -c /usr/local/bin/ ...   Up      127.0.0.1:1514->10514/tcp
harbor-portal       nginx -g daemon off;             Up      80/tcp                   
nginx               nginx -g daemon off;             Up      0.0.0.0:180->80/tcp      
redis               docker-entrypoint.sh redis ...   Up      6379/tcp                 
registry            /entrypoint.sh /etc/regist ...   Up      5000/tcp                 
registryctl         /harbor/start.sh                 Up
```

#### 6.配置harbor的dns内网解析

在hdss184-241上：

```shell
[root@hdss184-241 ~]# /var/named/od.com.zone
harbor             A    192.168.6.245

# 注意serial前滚一个序号
```

重启named

```shell
[root@hdss184-241 ~]# systemctl restart named
```

测试

```shell
[root@hdss184-241 ~]# dig -t A harbor.od.com +short
192.168.6.245
```

#### 7.安装nginx并配置

用nginx代理180端口：

```shell
[root@hdss184-245 harbor]# yum install nginx -y
[root@hdss184-245 harbor]# rpm -qa nginx
nginx-1.16.1-1.el7.x86_64

[root@hdss184-245 harbor]# vim /etc/nginx/conf.d/harbor.od.com.conf
server {
    listen       80;
    server_name  harbor.od.com;

    client_max_body_size 1000m;

    location / {
        proxy_pass http://127.0.0.1:180;
    }
}

[root@hdss184-245 harbor]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@hdss184-245 harbor]# systemctl start nginx
[root@hdss184-245 harbor]# systemctl enable nginx
```

#### 8.浏览器打开[http://harbor.od.com](http://harbor.od.com/)

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_912067b201416e737952e930e79212e1_r.png)

> 账号为admin 密码是Harbor12345

#### 9.检查

1.登录harbor，创建public仓库
![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_07c23944a8431db55a23c99f478faa44_r.png)

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_c18cf349e136767c800c57a09c79ab59_r.png)

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_eafeec65828a52590657bdcea5eb659c_r.png)

2.从docker.io下载镜像nginx:1.7.9

```shell
[root@hdss184-245 ~]# docker pull nginx:1.7.9
1.7.9: Pulling from library/nginx
Image docker.io/library/nginx:1.7.9 uses outdated schema1 manifest format. Please upgrade to a schema2 image for better future compatibility. More information at https://docs.docker.com/registry/spec/deprecated-schema-v1/
a3ed95caeb02: Pull complete 
6f5424ebd796: Pull complete 
d15444df170a: Pull complete 
e83f073daa67: Pull complete 
a4d93e421023: Pull complete 
084adbca2647: Pull complete 
c9cec474c523: Pull complete 
Digest: sha256:e3456c851a152494c3e4ff5fcc26f240206abac0c9d794affb40e0714846c451
Status: Downloaded newer image for nginx:1.7.9
docker.io/library/nginx:1.7.9
```

3.打tag

```shell
[root@hdss184-245 ~]# docker images|grep 1.7.9
nginx                           1.7.9                      84581e99d807        4 years ago         91.7MB
[root@hdss184-245 ~]# docker tag 84581e99d807 harbor.od.com/public/nginx:v1.7.9
```

4.登录私有仓库，并推送镜像nginx

```shell
[root@hdss184-245 ~]# docker login harbor.od.com
Username: admin
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
[root@hdss184-245 ~]# docker push harbor.od.com/public/nginx:v1.7.9
The push refers to repository [harbor.od.com/public/nginx]
5f70bf18a086: Pushed 
4b26ab29a475: Pushed 
ccb1d68e3fb7: Pushed 
e387107e2065: Pushed 
63bf84221cce: Pushed 
e02dce553481: Pushed 
dea2e4984e29: Pushed 
v1.7.9: digest: sha256:b1f5935eb2e9e2ae89c0b3e2e148c19068d91ca502e857052f14db230443e4c2 size: 3012
```

5.查看仓库

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_f43bd912bcc3eb48f776b460fe54736f_r.png)





# 第五章：部署主控节点服务

## 1.部署etcd集群

### 1.集群规划

| 主机名               | 角色        | ip              |
| :------------------- | :---------- | :-------------- |
| hdss184-242.host.com | etcd lead   | 192.168.184.242 |
| hdss184-243.host.com | etcd foolow | 192.168.184.243 |
| hdss184-244.host.com | etcd foolow | 192.168.184.244 |

注意：这里部署文档以`hdss184-242.host.com`主机为例，另外两台主机安装部署方法类似

### 2.创建基于根证书的config配置文件

```shell
[root@hdss184-245 certs]# cd /opt/certs/
[root@hdss184-245 certs]# vim /opt/certs/ca-config.json

{
    "signing": {
        "default": {
            "expiry": "175200h"
        },
        "profiles": {
            "server": {
                "expiry": "175200h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth"
                ]
            },
            "client": {
                "expiry": "175200h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "client auth"
                ]
            },
            "peer": {
                "expiry": "175200h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth",
                    "client auth"
                ]
            }
        }
    }
}
```

> 证书类型
> client certificate：客户端使用，用于服务端认证客户端，例如etcdctl、etcd proxy、fleetctl、docker客户端
> server certificate：服务器端使用，客户端已验证服务端身份，例如docker服务端、kube-apiserver
> peer certificate：双向证书，用于etcd集群成员间通信

### 3.创建生成自签证书签名请求(csr)的JSON配置文件

运维主机`hdss184-245.host.com`上：

```shell
[root@hdss184-245 certs]# vi etcd-peer-csr.json

{
    "CN": "k8s-etcd",
    "hosts": [
        "192.168.6.241",
        "192.168.6.242",
        "192.168.6.243",
        "192.168.6.244"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "beijing",
            "L": "beijing",
            "O": "od",
            "OU": "ops"
        }
    ]
}
```

### 4.生成etcd证书和私钥

```shell
[root@hdss184-245 certs]# cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=peer etcd-peer-csr.json|cfssl-json -bare etcd-peer
2019/11/13 17:00:03 [INFO] generate received request
2019/11/13 17:00:03 [INFO] received CSR
2019/11/13 17:00:03 [INFO] generating key: rsa-2048
2019/11/13 17:00:04 [INFO] encoded CSR
2019/11/13 17:00:04 [INFO] signed certificate with serial number 69997016866371968425072677347883174107938471757
2019/11/13 17:00:04 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
websites. For more information see the Baseline Requirements for the Issuance and Management
of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
specifically, section 10.2.3 ("Information Requirements").
```

### 5.检查生成的证书、私钥

```shell
[root@hdss184-245 certs]# ll
total 36
-rw-r--r-- 1 root root  836 Nov 13 16:32 ca-config.json
-rw-r--r-- 1 root root  993 Nov 12 16:31 ca.csr
-rw-r--r-- 1 root root  328 Nov 12 16:06 ca-csr.json
-rw------- 1 root root 1679 Nov 12 16:31 ca-key.pem
-rw-r--r-- 1 root root 1346 Nov 12 16:31 ca.pem
-rw-r--r-- 1 root root 1062 Nov 13 17:00 etcd-peer.csr
-rw-r--r-- 1 root root  379 Nov 13 16:34 etcd-peer-csr.json
-rw------- 1 root root 1679 Nov 13 17:00 etcd-peer-key.pem
-rw-r--r-- 1 root root 1428 Nov 13 17:00 etcd-peer.pem
```

### 6.创建etcd用户

在hdss184-242机器上：

```shell
[root@hdss184-242 ~]# useradd -s /sbin/nologin  -M etcd
```

### 7.下载软件，解压，做软连接

[etcd下载地址](https://github.com/etcd-io/etcd/tags)

[这里使用的是etcd-v3.1.20](https://github.com/etcd-io/etcd/releases/download/v3.1.20/etcd-v3.1.20-linux-amd64.tar.gz)

在hdss184-242机器上：

```shell
[root@hdss184-242 ~]# cd /opt/src/
[root@hdss184-242 src]# wget https://github.com/etcd-io/etcd/releases/download/v3.1.20/etcd-v3.1.20-linux-amd64.tar.gz
[root@hdss184-242 src]# tar xf etcd-v3.1.20-linux-amd64.tar.gz -C /opt
[root@hdss184-242 src]# cd /opt/
[root@hdss184-242 opt]# mv etcd-v3.1.20-linux-amd64/ etcd-v3.1.20
[root@hdss184-242 opt]# ln -s /opt/etcd-v3.1.20/ /opt/etcd
[root@hdss184-242 src]# ls -l /opt/
total 0
lrwxrwxrwx 1 root   root   18 Nov 13 17:30 etcd -> /opt/etcd-v3.1.20/
drwxr-xr-x 3 478493 89939 123 Oct 11  2018 etcd-v3.1.20
drwxr-xr-x 2 root   root   45 Nov 13 17:27 src
```

### 8.创建目录，拷贝证书，私钥

在hdss184-242机器上：

- 创建目录

```shell
[root@hdss184-242 src]# mkdir -p /opt/etcd/certs /data/etcd /data/logs/etcd-server
```

- 拷贝证书

```shell
[root@hdss184-242 src]# cd /opt/etcd/certs
[root@hdss184-242 certs]# scp root@hdss184-245:/opt/certs/ca.pem /opt/etcd/certs/
root@hdss184-245's password: 
ca.pem                                                                                             100% 1346   133.4KB/s   00:00    
[root@hdss184-242 certs]# scp -P52113 hdss184-245:/opt/certs/etcd-peer.pem /opt/etcd/certs/
root@hdss184-245's password: 
etcd-peer.pem                                                                                      100% 1428   208.6KB/s   00:00    
[root@hdss184-242 certs]# scp -P52113 hdss184-245:/opt/certs/etcd-peer-key.pem /opt/etcd/certs/
root@hdss184-245's password: 
etcd-peer-key.pem
```

> 将运维主机上生成的ca.pem,etcd-peer-key.pem,etcd-peer.pem拷贝到/ope/etcd/certs目录中，注意私钥权限600

- 修改权限

```shell
/opt/etcd/certs

[root@hdss184-242 certs]# chown -R etcd.etcd /opt/etcd/certs /data/etcd /data/logs/etcd-server
[root@hdss184-242 certs]# ls -l
total 12
-rw-r--r-- 1 etcd etcd 1346 Nov 13 17:45 ca.pem
-rw------- 1 etcd etcd 1679 Nov 13 17:46 etcd-peer-key.pem
-rw-r--r-- 1 etcd etcd 1428 Nov 13 17:45 etcd-peer.pem
```

### 9.创建etcd服务启动脚本

在hdss184-242机器上：

```shell
[root@hdss184-242 certs]# vim /opt/etcd/etcd-server-startup.sh

#!/bin/sh
./etcd --name etcd-server-184-242 \
       --data-dir /data/etcd/etcd-server \
       --listen-peer-urls https://192.168.184.242:2380 \
       --listen-client-urls https://192.168.184.242:2379,http://127.0.0.1:2379 \
       --quota-backend-bytes 8000000000 \
       --initial-advertise-peer-urls https://192.168.184.242:2380 \
       --advertise-client-urls https://192.168.184.242:2379,http://127.0.0.1:2379 \
       --initial-cluster  etcd-server-184-242=https://192.168.184.242:2380,etcd-server-184-243=https://192.168.184.243:2380,etcd-server-184-244=https://192.168.184.244:2380 \
       --ca-file ./certs/ca.pem \
       --cert-file ./certs/etcd-peer.pem \
       --key-file ./certs/etcd-peer-key.pem \
       --client-cert-auth  \
       --trusted-ca-file ./certs/ca.pem \
       --peer-ca-file ./certs/ca.pem \
       --peer-cert-file ./certs/etcd-peer.pem \
       --peer-key-file ./certs/etcd-peer-key.pem \
       --peer-client-cert-auth \
       --peer-trusted-ca-file ./certs/ca.pem \
       --log-output stdout
```

注意：etcd集群各主机的启动脚本略有不同，部署其他节点是需要注意。

### 10.调整权限

在hdss184-242机器上：

```shell
[root@hdss184-242 certs]# cd ../
[root@hdss184-242 etcd]# chmod +x etcd-server-startup.sh 
[root@hdss184-242 etcd]# ll etcd-server-startup.sh 
-rwxr-xr-x 1 root root 1013 Nov 14 08:52 etcd-server-startup.sh
```

### 11.安装supervisor软件

在hdss184-242机器上：

```shell
[root@hdss184-242 etcd]# yum install supervisor -y
[root@hdss184-242 etcd]# systemctl start supervisord.service
[root@hdss184-242 etcd]# systemctl enable supervisord.service
```

### 12.创建etcd-server的启动配置

在hdss184-242机器上：

```shell
[root@hdss184-242 etcd]# vim /etc/supervisord.d/etcd-server.ini
[root@hdss184-242 etcd]# cat /etc/supervisord.d/etcd-server.ini
[program:etcd-server-184-242]
command=/opt/etcd/etcd-server-startup.sh                        ; the program (relative uses PATH, can take args)
numprocs=1                                                      ; number of processes copies to start (def 1)
directory=/opt/etcd                                             ; directory to cwd to before exec (def no cwd)
autostart=true                                                  ; start at supervisord start (default: true)
autorestart=true                                                ; retstart at unexpected quit (default: true)
startsecs=30                                                    ; number of secs prog must stay running (def. 1)
startretries=3                                                  ; max # of serial start failures (default 3)
exitcodes=0,2                                                   ; 'expected' exit codes for process (default 0,2)
stopsignal=QUIT                                                 ; signal used to kill process (default TERM)
stopwaitsecs=10                                                 ; max num secs to wait b4 SIGKILL (default 10)
user=etcd                                                       ; setuid to this UNIX account to run the program
redirect_stderr=true                                            ; redirect proc stderr to stdout (default false)
killasgroup=true                                                ; kill all process in a group
stopasgroup=true                                                ; stop all process in a group
stdout_logfile=/data/logs/etcd-server/etcd.stdout.log           ; stdout log path, NONE for none; default AUTO
stdout_logfile_maxbytes=64MB                                    ; max # logfile bytes b4 rotation (default 50MB)
stdout_logfile_backups=4                                        ; # of stdout logfile backups (default 10)
stdout_capture_maxbytes=1MB                                     ; number of bytes in 'capturemode' (default 0)
stdout_events_enabled=false                                     ; emit events on stdout writes (default false)
```

注意：etcd集群各主机启动配置略有不同，配置其他节点时注意修改。

### 13.启动etcd服务并检查

在hdss184-242机器上：

```shell
[root@hdss184-242 etcd]# supervisorctl update
etcd-server-6-242: added process group

[root@hdss184-242 etcd]# supervisorctl status
etcd-server-6-242                RUNNING   pid 10375, uptime 0:00:43
[root@hdss184-242 etcd]# netstat -lntup|grep etcd
tcp        0      0 192.168.6.242:2379      0.0.0.0:*               LISTEN      10376/./etcd
tcp        0      0 127.0.0.1:2379          0.0.0.0:*               LISTEN      10376/./etcd
tcp        0      0 192.168.6.242:2380      0.0.0.0:*               LISTEN      10376/./etcd 
```

### 14.安装部署启动检查所有集群规划的etcd服务

在hdss184-243机器上：

```shell
[root@hdss184-243 ~]# useradd -s /sbin/nologin -M etcd
[root@hdss184-243 ~]# mkdir /opt/src
[root@hdss184-243 ~]# cd /opt/src
[root@hdss184-243 src]# wget https://github.com/etcd-io/etcd/releases/download/v3.1.20/etcd-v3.1.20-linux-amd64.tar.gz
[root@hdss184-243 src]# tar xf etcd-v3.1.20-linux-amd64.tar.gz -C /opt
[root@hdss184-243 src]# cd /opt/
[root@hdss184-243 opt]# mv etcd-v3.1.20-linux-amd64/ etcd-v3.1.20
[root@hdss184-243 opt]# ln -s /opt/etcd-v3.1.20/ /opt/etcd
total 0
drwx--x--x  4 root   root   28 Nov 13 10:33 containerd
lrwxrwxrwx  1 root   root   18 Nov 14 09:28 etcd -> /opt/etcd-v3.1.20/
drwxr-xr-x  3 478493 89939 123 Oct 11  2018 etcd-v3.1.20
drwxr-xr-x. 2 root   root    6 Sep  7  2017 rh
drwxr-xr-x  2 root   root   45 Nov 14 09:26 src
[root@hdss184-243 opt]# rm -fr rh containerd/ containerd/
[root@hdss184-243 opt]# mkdir -p /opt/etcd/certs /data/etcd /data/logs/etcd-server
[root@hdss184-243 opt]# cd /opt/etcd/certs
[root@hdss184-243 certs]# scp  root@hdss184-245.host.com:/opt/certs/ca.pem /opt/etcd/certs/
[root@hdss184-243 certs]# scp  root@hdss184-245.host.com:/opt/certs/etcd-peer.pem /opt/etcd/certs/
[root@hdss184-243 certs]# scp  root@hdss184-245.host.com:/opt/certs/etcd-peer-key.pem /opt/etcd/certs/
[root@hdss184-243 certs]#  chown -R etcd.etcd /opt/etcd/certs /data/etcd /data/logs/etcd-server
[root@hdss184-243 certs]# vim /opt/etcd/etcd-server-startup.sh
[root@hdss184-243 certs]# cat /opt/etcd/etcd-server-startup.sh
#!/bin/sh
./etcd --name etcd-server-184-243 \
       --data-dir /data/etcd/etcd-server \
       --listen-peer-urls https://192.168.184.243:2380 \
       --listen-client-urls https://192.168.184.243:2379,http://127.0.0.1:2379 \
       --quota-backend-bytes 8000000000 \
       --initial-advertise-peer-urls https://192.168.184.243:2380 \
       --advertise-client-urls https://192.168.184.243:2379,http://127.0.0.1:2379 \
       --initial-cluster  etcd-server-184-242=https://192.168.184.242:2380,etcd-server-184-243=https://192.168.184.243:2380,etcd-server-184-244=https://192.168.184.244:2380 \
       --ca-file ./certs/ca.pem \
       --cert-file ./certs/etcd-peer.pem \
       --key-file ./certs/etcd-peer-key.pem \
       --client-cert-auth  \
       --trusted-ca-file ./certs/ca.pem \
       --peer-ca-file ./certs/ca.pem \
       --peer-cert-file ./certs/etcd-peer.pem \
       --peer-key-file ./certs/etcd-peer-key.pem \
       --peer-client-cert-auth \
       --peer-trusted-ca-file ./certs/ca.pem \
       --log-output stdout
[root@hdss184-243 certs]# chmod +x /opt/etcd/etcd-server-startup.sh
[root@hdss184-243 certs]# yum install supervisor -y
[root@hdss184-243 certs]# systemctl start supervisord.service
[root@hdss184-243 certs]# systemctl enable supervisord.service
[root@hdss184-243 certs]# vim /etc/supervisord.d/etcd-server.ini
[root@hdss184-243 certs]# cat /etc/supervisord.d/etcd-server.ini
[program:etcd-server-6-243]
command=/opt/etcd/etcd-server-startup.sh                        ; the program (relative uses PATH, can take args)
numprocs=1                                                      ; number of processes copies to start (def 1)
directory=/opt/etcd                                             ; directory to cwd to before exec (def no cwd)
autostart=true                                                  ; start at supervisord start (default: true)
autorestart=true                                                ; retstart at unexpected quit (default: true)
startsecs=30                                                    ; number of secs prog must stay running (def. 1)
startretries=3                                                  ; max # of serial start failures (default 3)
exitcodes=0,2                                                   ; 'expected' exit codes for process (default 0,2)
stopsignal=QUIT                                                 ; signal used to kill process (default TERM)
stopwaitsecs=10                                                 ; max num secs to wait b4 SIGKILL (default 10)
user=etcd                                                       ; setuid to this UNIX account to run the program
redirect_stderr=true                                            ; redirect proc stderr to stdout (default false)
killasgroup=true                                                ; kill all process in a group
stopasgroup=true                                                ; stop all process in a group
stdout_logfile=/data/logs/etcd-server/etcd.stdout.log           ; stdout log path, NONE for none; default AUTO
stdout_logfile_maxbytes=64MB                                    ; max # logfile bytes b4 rotation (default 50MB)
stdout_logfile_backups=4                                        ; # of stdout logfile backups (default 10)
stdout_capture_maxbytes=1MB                                     ; number of bytes in 'capturemode' (default 0)
stdout_events_enabled=false                                     ; emit events on stdout writes (default false)
[root@hdss184-243 certs]# supervisorctl update
etcd-server-6-243: added process group
[root@hdss184-243 certs]# supervisorctl status
etcd-server-6-243                STARTING  
[root@hdss184-243 certs]# netstat -lntup|grep etcd
tcp        0      0 192.168.6.243:2379      0.0.0.0:*               LISTEN      12113/./etcd        
tcp        0      0 127.0.0.1:2379          0.0.0.0:*               LISTEN      12113/./etcd        
tcp        0      0 192.168.6.243:2380      0.0.0.0:*               LISTEN      12113/./etcd
```

在hdss184-244机器上：

```shell
[root@hdss184-244 ~]# useradd -s /sbin/nologin  -M etcd
[root@hdss184-244 ~]# mkdir /opt/src
[root@hdss184-244 ~]# cd /opt/src
[root@hdss184-244 src]# wget https://github.com/etcd-io/etcd/releases/download/v3.1.20/etcd-v3.1.20-linux-amd64.tar.gz
[root@hdss184-244 src]# useradd -s /sbin/nologin  -M etcd
useradd: user 'etcd' already exists
[root@hdss184-244 src]# tar xf etcd-v3.1.20-linux-amd64.tar.gz -C /opt
[root@hdss184-244 src]# cd /opt/
[root@hdss184-244 opt]# mv etcd-v3.1.20-linux-amd64/ etcd-v3.1.20
[root@hdss184-244 opt]# ln -s /opt/etcd-v3.1.20/ /opt/etcd
[root@hdss184-244 opt]# mkdir -p /opt/etcd/certs /data/etcd /data/logs/etcd-server
[root@hdss184-244 opt]#  cd /opt/etcd/certs
[root@hdss184-244 certs]# scp  root@hdss184-245:/opt/certs/ca.pem /opt/etcd/certs/
[root@hdss184-244 certs]# scp  root@hdss184-245:/opt/certs/etcd-peer.pem /opt/etcd/certs/
[root@hdss184-244 certs]# scp  root@hdss184-245:/opt/certs/etcd-peer-key.pem /opt/etcd/certs/
[root@hdss184-244 certs]# chown -R etcd.etcd /opt/etcd/certs /data/etcd /data/logs/etcd-server
[root@hdss184-244 certs]# vim /opt/etcd/etcd-server-startup.sh
[root@hdss184-244 etcd]# cat /opt/etcd/etcd-server-startup.sh
#!/bin/sh
./etcd --name etcd-server-184-244 \
       --data-dir /data/etcd/etcd-server \
       --listen-peer-urls https://192.168.184.244:2380 \
       --listen-client-urls https://192.168.184.244:2379,http://127.0.0.1:2379 \
       --quota-backend-bytes 8000000000 \
       --initial-advertise-peer-urls https://192.168.184.244:2380 \
       --advertise-client-urls https://192.168.184.244:2379,http://127.0.0.1:2379 \
       --initial-cluster  etcd-server-184-242=https://192.168.184.242:2380,etcd-server-184-243=https://192.168.184.243:2380,etcd-server-184-244=https://192.168.184.244:2380 \
       --ca-file ./certs/ca.pem \
       --cert-file ./certs/etcd-peer.pem \
       --key-file ./certs/etcd-peer-key.pem \
       --client-cert-auth  \
       --trusted-ca-file ./certs/ca.pem \
       --peer-ca-file ./certs/ca.pem \
       --peer-cert-file ./certs/etcd-peer.pem \
       --peer-key-file ./certs/etcd-peer-key.pem \
       --peer-client-cert-auth \
       --peer-trusted-ca-file ./certs/ca.pem \
       --log-output stdout
[root@hdss184-244 certs]# cd ../
[root@hdss184-244 etcd]# chmod +x etcd-server-startup.sh 
[root@hdss184-244 etcd]# yum install supervisor -y
[root@hdss184-244 etcd]# systemctl start supervisord.service
[root@hdss184-244 etcd]# systemctl enable supervisord.service
[root@hdss184-244 etcd]#  vim /etc/supervisord.d/etcd-server.ini
[root@hdss184-244 etcd]# cat /etc/supervisord.d/etcd-server.ini
[program:etcd-server-6-244]
command=/opt/etcd/etcd-server-startup.sh                        ; the program (relative uses PATH, can take args)
numprocs=1                                                      ; number of processes copies to start (def 1)
directory=/opt/etcd                                             ; directory to cwd to before exec (def no cwd)
autostart=true                                                  ; start at supervisord start (default: true)
autorestart=true                                                ; retstart at unexpected quit (default: true)
startsecs=30                                                    ; number of secs prog must stay running (def. 1)
startretries=3                                                  ; max # of serial start failures (default 3)
exitcodes=0,2                                                   ; 'expected' exit codes for process (default 0,2)
stopsignal=QUIT                                                 ; signal used to kill process (default TERM)
stopwaitsecs=10                                                 ; max num secs to wait b4 SIGKILL (default 10)
user=etcd                                                       ; setuid to this UNIX account to run the program
redirect_stderr=true                                            ; redirect proc stderr to stdout (default false)
killasgroup=true                                                ; kill all process in a group
stopasgroup=true                                                ; stop all process in a group
stdout_logfile=/data/logs/etcd-server/etcd.stdout.log           ; stdout log path, NONE for none; default AUTO
stdout_logfile_maxbytes=64MB                                    ; max # logfile bytes b4 rotation (default 50MB)
stdout_logfile_backups=4                                        ; # of stdout logfile backups (default 10)
stdout_capture_maxbytes=1MB                                     ; number of bytes in 'capturemode' (default 0)
stdout_events_enabled=false                                     ; emit events on stdout writes (default false)
[root@hdss184-244 etcd]# supervisorctl update
[root@hdss184-244 etcd]# supervisorctl status
etcd-server-6-244                RUNNING   pid 11748, uptime 0:00:33
[root@hdss184-244 etcd]#  netstat -lntup|grep etcd
tcp        0      0 192.168.6.244:2379      0.0.0.0:*               LISTEN      11749/./etcd        
tcp        0      0 127.0.0.1:2379          0.0.0.0:*               LISTEN      11749/./etcd        
tcp        0      0 192.168.6.244:2380      0.0.0.0:*               LISTEN      11749/./etcd
[root@hdss184-244 etcd]# ./etcdctl cluster-health
member 4244d625c76d5482 is healthy: got healthy result from http://127.0.0.1:2379
member aa911af67b8285a2 is healthy: got healthy result from http://127.0.0.1:2379
member c751958d48e7e127 is healthy: got healthy result from http://127.0.0.1:2379
cluster is healthy
```

### 15.检查集群状态

**三个etcd节点都起来后**

在hdss184-242机器上：

```shell
[root@hdss184-242 etcd]# ./etcdctl cluster-health
member 4244d625c76d5482 is healthy: got healthy result from http://127.0.0.1:2379
member aa911af67b8285a2 is healthy: got healthy result from http://127.0.0.1:2379
member c751958d48e7e127 is healthy: got healthy result from http://127.0.0.1:2379
cluster is healthy
[root@hdss184-242 etcd]# ./etcdctl member list
4244d625c76d5482: name=etcd-server-6-242 peerURLs=https://192.168.6.242:2380 clientURLs=http://127.0.0.1:2379,https://192.168.6.242:2379 isLeader=true
aa911af67b8285a2: name=etcd-server-6-243 peerURLs=https://192.168.6.243:2380 clientURLs=http://127.0.0.1:2379,https://192.168.6.243:2379 isLeader=false
c751958d48e7e127: name=etcd-server-6-244 peerURLs=https://192.168.6.244:2380 clientURLs=http://127.0.0.1:2379,http
```



## 2.部署kube-apiserver集群

### 1.集群规划

| 主机名               | 角色           | ip            |
| :------------------- | :------------- | :------------ |
| hdss184-243.host.com | kube-apiserver | 192.168.6.243 |
| hdss184-244.host.com | kube-apiserver | 192.168.6.244 |
| hdss184-241.host.com | 4层负载均衡    | 192.168.6.241 |
| hdss184-242.host.com | 4层负载均衡    | 192.168.6.242 |

注意：这里`192.168.6.241`和`192.168.6.242`使用nginx做4层负载均衡器，用keepalived跑一个vip：`192.168.6.66`，代理两个kube-apiserver，实现高可用

这里部署文档以`hdss184-243.host.com`主机为例，另外一台运算节点安装部署方法类似

### 2.下载软件，解压，做软链

`hdss184-243.host.com`主机上：

[kubernetes官方Github地址](https://github.com/kubernetes/kubernetes)
[kubernetes下地址](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.15.md#downloads-for-v1154)

```shell
[root@hdss184-243 src]# cd /opt/src/
[root@hdss184-243 src]# wget http://down.sunrisenan.com/k8s/kubernetes/kubernetes-server-linux-amd64-v1.15.2.tar.gz

[root@hdss184-243 src]# tar xf kubernetes-server-linux-amd64-v1.15.2.tar.gz -C /opt/
[root@hdss184-243 src]# cd /opt/

[root@hdss184-243 opt]# mv kubernetes/ kubernetes-v1.15.2
[root@hdss184-243 opt]# ln -s /opt/kubernetes-v1.15.2/ /opt/kubernetes
```

**删除源码**

```shell
[root@hdss184-243 opt]# cd kubernetes
[root@hdss184-243 kubernetes]# rm -f kubernetes-src.tar.gz 
[root@hdss184-243 kubernetes]# ll
total 1180
drwxr-xr-x 2 root root       6 Aug  5 18:01 addons
-rw-r--r-- 1 root root 1205293 Aug  5 18:01 LICENSES
drwxr-xr-x 3 root root      17 Aug  5 17:57 server
```

**删除docker镜像**

```shell
[root@hdss184-243 kubernetes]# cd server/bin
[root@hdss184-243 bin]# ll
total 1548800
-rwxr-xr-x 1 root root  43534816 Aug  5 18:01 apiextensions-apiserver
-rwxr-xr-x 1 root root 100548640 Aug  5 18:01 cloud-controller-manager
-rw-r--r-- 1 root root         8 Aug  5 17:57 cloud-controller-manager.docker_tag
-rw-r--r-- 1 root root 144437760 Aug  5 17:57 cloud-controller-manager.tar
-rwxr-xr-x 1 root root 200648416 Aug  5 18:01 hyperkube
-rwxr-xr-x 1 root root  40182208 Aug  5 18:01 kubeadm
-rwxr-xr-x 1 root root 164501920 Aug  5 18:01 kube-apiserver
-rw-r--r-- 1 root root         8 Aug  5 17:57 kube-apiserver.docker_tag
-rw-r--r-- 1 root root 208390656 Aug  5 17:57 kube-apiserver.tar
-rwxr-xr-x 1 root root 116397088 Aug  5 18:01 kube-controller-manager
-rw-r--r-- 1 root root         8 Aug  5 17:57 kube-controller-manager.docker_tag
-rw-r--r-- 1 root root 160286208 Aug  5 17:57 kube-controller-manager.tar
-rwxr-xr-x 1 root root  42985504 Aug  5 18:01 kubectl
-rwxr-xr-x 1 root root 119616640 Aug  5 18:01 kubelet
-rwxr-xr-x 1 root root  36987488 Aug  5 18:01 kube-proxy
-rw-r--r-- 1 root root         8 Aug  5 17:57 kube-proxy.docker_tag
-rw-r--r-- 1 root root  84282368 Aug  5 17:57 kube-proxy.tar
-rwxr-xr-x 1 root root  38786144 Aug  5 18:01 kube-scheduler
-rw-r--r-- 1 root root         8 Aug  5 17:57 kube-scheduler.docker_tag
-rw-r--r-- 1 root root  82675200 Aug  5 17:57 kube-scheduler.tar
-rwxr-xr-x 1 root root   1648224 Aug  5 18:01 mounter
[root@hdss184-243 bin]# rm -f *.tar
[root@hdss184-243 bin]# rm -f *_tag
[root@hdss184-243 bin]# ll
total 884636
-rwxr-xr-x 1 root root  43534816 Aug  5 18:01 apiextensions-apiserver
-rwxr-xr-x 1 root root 100548640 Aug  5 18:01 cloud-controller-manager
-rwxr-xr-x 1 root root 200648416 Aug  5 18:01 hyperkube
-rwxr-xr-x 1 root root  40182208 Aug  5 18:01 kubeadm
-rwxr-xr-x 1 root root 164501920 Aug  5 18:01 kube-apiserver
-rwxr-xr-x 1 root root 116397088 Aug  5 18:01 kube-controller-manager
-rwxr-xr-x 1 root root  42985504 Aug  5 18:01 kubectl
-rwxr-xr-x 1 root root 119616640 Aug  5 18:01 kubelet
-rwxr-xr-x 1 root root  36987488 Aug  5 18:01 kube-proxy
-rwxr-xr-x 1 root root  38786144 Aug  5 18:01 kube-scheduler
-rwxr-xr-x 1 root root   1648224 Aug  5 18:01 mounter
```

### 3.签发client证书

在运维机`hdss184.245.host.com`上：

#### 1.创建生成证书请求（csr）的JSON配置文件

```shell
vim /opt/certs/client-csr.json

{
    "CN": "k8s-node",
    "hosts": [
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "beijing",
            "L": "beijing",
            "O": "od",
            "OU": "ops"
        }
    ]
}
```

#### 2.生成client证书和私钥

```shell
# cd /opt/certs/
[root@hdss184-245 certs]# cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=client client-csr.json |cfssl-json -bare client
2019/11/14 13:59:24 [INFO] generate received request
2019/11/14 13:59:24 [INFO] received CSR
2019/11/14 13:59:24 [INFO] generating key: rsa-2048
2019/11/14 13:59:24 [INFO] encoded CSR
2019/11/14 13:59:24 [INFO] signed certificate with serial number 71787071397684874048844497862502145400133190813
2019/11/14 13:59:24 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
websites. For more information see the Baseline Requirements for the Issuance and Management
of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
specifically, section 10.2.3 ("Information Requirements").
```

#### 3.检查生成的证书和私钥

```shell
[root@hdss184-245 certs]# ll client*
-rw-r--r-- 1 root root  993 Nov 14 13:59 client.csr
-rw-r--r-- 1 root root  280 Nov 14 13:59 client-csr.json
-rw------- 1 root root 1679 Nov 14 13:59 client-key.pem
-rw-r--r-- 1 root root 1363 Nov 14 13:59 client.pem
```

### 4.签发kube-apiserver证书

在运维机`hdss184.245.host.com`上：

#### 1.创建生成证书签名请求（csr）的josn配置文件

```shell
[root@hdss184-245 certs]# vim /opt/certs/apiserver-csr.json
[root@hdss184-245 certs]# cat /opt/certs/apiserver-csr.json
{
    "CN": "k8s-apiserver",
    "hosts": [
        "127.0.0.1",
        "10.96.0.1",
        "kubernetes.default",
        "kubernetes.default.svc",
        "kubernetes.default.svc.cluster",
        "kubernetes.default.svc.cluster.local",
        "192.168.184.66",
        "192.168.184.243",
        "192.168.184.244",
        "192.168.184.245"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "beijing",
            "L": "beijing",
            "O": "od",
            "OU": "ops"
        }
    ]
} 
```

#### 2.生成kube-apiserver证书和私钥

```shell
[root@hdss184-245 certs]# cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=server apiserver-csr.json |cfssl-json -bare apiserver
2019/11/14 14:10:01 [INFO] generate received request
2019/11/14 14:10:01 [INFO] received CSR
2019/11/14 14:10:01 [INFO] generating key: rsa-2048
2019/11/14 14:10:02 [INFO] encoded CSR
2019/11/14 14:10:02 [INFO] signed certificate with serial number 531358145467350237994138515547646071524442824033
2019/11/14 14:10:02 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
websites. For more information see the Baseline Requirements for the Issuance and Management
of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
specifically, section 10.2.3 ("Information Requirements").
```

#### 3.检查生成的证书和私钥

```shell
[root@hdss184-245 certs]# ll apiserver*
-rw-r--r-- 1 root root 1249 Nov 14 14:10 apiserver.csr
-rw-r--r-- 1 root root  581 Nov 14 14:09 apiserver-csr.json
-rw------- 1 root root 1679 Nov 14 14:10 apiserver-key.pem
-rw-r--r-- 1 root root 1598 Nov 14 14:10 apiserver.pem
```

### 5.拷贝证书至各个运算节点，并创建配置

在运维机`hdss184.243.host.com`上：

拷贝证书

```shell
[root@hdss184-243 bin]# pwd
/opt/kubernetes/server/bin
[root@hdss184-243 bin]# mkdir cert
[root@hdss184-243 bin]# scp root@hdss184-245:/opt/certs/apiserver-key.pem /opt/kubernetes/server/bin/cert/
[root@hdss184-243 bin]# scp root@hdss184-245:/opt/certs/apiserver.pem /opt/kubernetes/server/bin/cert/
[root@hdss184-243 bin]# scp root@hdss184-245:/opt/certs/ca-key.pem /opt/kubernetes/server/bin/cert/
[root@hdss184-243 bin]# scp root@hdss184-245:/opt/certs/ca.pem /opt/kubernetes/server/bin/cert/  
[root@hdss184-243 bin]# scp root@hdss184-245:/opt/certs/client-key.pem /opt/kubernetes/server/bin/cert/
[root@hdss184-243 bin]# scp root@hdss184-245:/opt/certs/client.pem /opt/kubernetes/server/bin/cert/
```

创建配置文件

```shell
[root@hdss184-243 bin]# mkdir conf
[root@hdss184-243 bin]# vi conf/audit.yaml
[root@hdss184-243 bin]# cat conf/audit.yaml 
apiVersion: audit.k8s.io/v1beta1 # This is required.
kind: Policy
# Don't generate audit events for all requests in RequestReceived stage.
omitStages:
  - "RequestReceived"
rules:
  # Log pod changes at RequestResponse level
  - level: RequestResponse
    resources:
    - group: ""
      # Resource "pods" doesn't match requests to any subresource of pods,
      # which is consistent with the RBAC policy.
      resources: ["pods"]
  # Log "pods/log", "pods/status" at Metadata level
  - level: Metadata
    resources:
    - group: ""
      resources: ["pods/log", "pods/status"]

  # Don't log requests to a configmap called "controller-leader"
  - level: None
    resources:
    - group: ""
      resources: ["configmaps"]
      resourceNames: ["controller-leader"]

  # Don't log watch requests by the "system:kube-proxy" on endpoints or services
  - level: None
    users: ["system:kube-proxy"]
    verbs: ["watch"]
    resources:
    - group: "" # core API group
      resources: ["endpoints", "services"]

  # Don't log authenticated requests to certain non-resource URL paths.
  - level: None
    userGroups: ["system:authenticated"]
    nonResourceURLs:
    - "/api*" # Wildcard matching.
    - "/version"

  # Log the request body of configmap changes in kube-system.
  - level: Request
    resources:
    - group: "" # core API group
      resources: ["configmaps"]
    # This rule only applies to resources in the "kube-system" namespace.
    # The empty string "" can be used to select non-namespaced resources.
    namespaces: ["kube-system"]

  # Log configmap and secret changes in all other namespaces at the Metadata level.
  - level: Metadata
    resources:
    - group: "" # core API group
      resources: ["secrets", "configmaps"]

  # Log all other resources in core and extensions at the Request level.
  - level: Request
    resources:
    - group: "" # core API group
    - group: "extensions" # Version of group should NOT be included.

  # A catch-all rule to log all other requests at the Metadata level.
  - level: Metadata
    # Long-running requests like watches that fall under this rule will not
    # generate an audit event in RequestReceived.
    omitStages:
      - "RequestReceived"
```

### 6.创建启动脚本

在运维机`hdss184.243.host.com`上：

```shell
[root@hdss184-243 bin]# vim /opt/kubernetes/server/bin/kube-apiserver.sh
[root@hdss184-243 bin]# cat /opt/kubernetes/server/bin/kube-apiserver.sh
#!/bin/bash
./kube-apiserver \
  --apiserver-count 2 \
  --audit-log-path /data/logs/kubernetes/kube-apiserver/audit-log \
  --audit-policy-file ./conf/audit.yaml \
  --authorization-mode RBAC \
  --client-ca-file ./cert/ca.pem \
  --requestheader-client-ca-file ./cert/ca.pem \
  --enable-admission-plugins NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota \
  --etcd-cafile ./cert/ca.pem \
  --etcd-certfile ./cert/client.pem \
  --etcd-keyfile ./cert/client-key.pem \
  --etcd-servers https://192.168.184.242:2379,https://192.168.184.243:2379,https://192.168.184.244:2379 \
  --service-account-key-file ./cert/ca-key.pem \
  --service-cluster-ip-range 10.96.0.0/22 \
  --service-node-port-range 3000-29999 \
  --target-ram-mb=1024 \
  --kubelet-client-certificate ./cert/client.pem \
  --kubelet-client-key ./cert/client-key.pem \
  --log-dir  /data/logs/kubernetes/kube-apiserver \
  --tls-cert-file ./cert/apiserver.pem \
  --tls-private-key-file ./cert/apiserver-key.pem \
  --v 2
```

### 7.调整权限和目录

在运维机`hdss184.243.host.com`上：

```shell
[root@hdss184-243 bin]# chmod +x /opt/kubernetes/server/bin/kube-apiserver.sh
[root@hdss184-243 bin]# mkdir -p /data/logs/kubernetes/kube-apiserver
```

### 8.创建supervisor配置

在运维机`hdss184.243.host.com`上：

```shell
[root@hdss184-243 bin]# vi /etc/supervisord.d/kube-apiserver.ini
[root@hdss184-243 bin]# cat /etc/supervisord.d/kube-apiserver.ini
[program:kube-apiserver-6-243]
command=/opt/kubernetes/server/bin/kube-apiserver.sh            ; the program (relative uses PATH, can take args)
numprocs=1                                                      ; number of processes copies to start (def 1)
directory=/opt/kubernetes/server/bin                            ; directory to cwd to before exec (def no cwd)
autostart=true                                                  ; start at supervisord start (default: true)
autorestart=true                                                ; retstart at unexpected quit (default: true)
startsecs=30                                                    ; number of secs prog must stay running (def. 1)
startretries=3                                                  ; max # of serial start failures (default 3)
exitcodes=0,2                                                   ; 'expected' exit codes for process (default 0,2)
stopsignal=QUIT                                                 ; signal used to kill process (default TERM)
stopwaitsecs=10                                                 ; max num secs to wait b4 SIGKILL (default 10)
user=root                                                       ; setuid to this UNIX account to run the program
redirect_stderr=true                                            ; redirect proc stderr to stdout (default false)
killasgroup=true                                                ; kill all process in a group
stopasgroup=true                                                ; stop all process in a group
stdout_logfile=/data/logs/kubernetes/kube-apiserver/apiserver.stdout.log        ; stderr log path, NONE for none; default AUTO
stdout_logfile_maxbytes=64MB                                    ; max # logfile bytes b4 rotation (default 50MB)
stdout_logfile_backups=4                                        ; # of stdout logfile backups (default 10)
stdout_capture_maxbytes=1MB                                     ; number of bytes in 'capturemode' (default 0)
stdout_events_enabled=false                                     ; emit events on stdout writes (default false)
```

### 9.启动服务并检查

```shell
[root@hdss184-243 bin]# supervisorctl update
kube-apiserver-6-243: added process group
[root@hdss184-243 bin]# supervisorctl status
etcd-server-6-243                RUNNING   pid 12112, uptime 5:06:23
kube-apiserver-6-243             RUNNING   pid 12824, uptime 0:00:46

[root@hdss184-243 bin]# netstat -lntup|grep kube-apiser
tcp        0      0 127.0.0.1:8080          0.0.0.0:*               LISTEN      12825/./kube-apiser 
tcp6       0      0 :::6443                 :::*                    LISTEN      12825/./kube-apiser 
```

### 10.安装部署启动检查所有集群规划机器

```shell
[root@hdss184-244 src]# tar xf kubernetes-server-linux-amd64-v1.15.2.tar.gz -C /opt/
[root@hdss184-244 src]# cd /opt/
[root@hdss184-244 opt]# mv kubernetes kubernetes-v1.15.2
[root@hdss184-244 opt]# ln -s /opt/kubernetes-v1.15.2/ /opt/kubernetes
[root@hdss184-244 opt]# cd kubernetes
[root@hdss184-244 kubernetes]# rm -f kubernetes-src.tar.gz 
[root@hdss184-244 kubernetes]# cd server/bin
[root@hdss184-244 bin]# rm -f *.tar
[root@hdss184-244 bin]# rm -f *_tag
[root@hdss184-244 bin]# mkdir cert
[root@hdss184-244 bin]# scp  root@hdss184-245:/opt/certs/apiserver-key.pem /opt/kubernetes/server/bin/cert/
[root@hdss184-244 bin]# scp -P52113 hdss184-245:/opt/certs/apiserver.pem /opt/kubernetes/server/bin/cert/
[root@hdss184-244 bin]# scp -P52113 hdss184-245:/opt/certs/ca-key.pem /opt/kubernetes/server/bin/cert/
[root@hdss184-244 bin]# scp -P52113 hdss184-245:/opt/certs/ca.pem /opt/kubernetes/server/bin/cert/  
[root@hdss184-244 bin]# scp -P52113 hdss184-245:/opt/certs/client-key.pem /opt/kubernetes/server/bin/cert/
[root@hdss184-244 bin]# scp -P52113 hdss184-245:/opt/certs/client.pem /opt/kubernetes/server/bin/cert/
[root@hdss184-244 bin]# mkdir conf
[root@hdss184-244 bin]# vi conf/audit.yaml
[root@hdss184-244 bin]# cat conf/audit.yaml 
apiVersion: audit.k8s.io/v1beta1 # This is required.
kind: Policy
# Don't generate audit events for all requests in RequestReceived stage.
omitStages:
  - "RequestReceived"
rules:
  # Log pod changes at RequestResponse level
  - level: RequestResponse
    resources:
    - group: ""
      # Resource "pods" doesn't match requests to any subresource of pods,
      # which is consistent with the RBAC policy.
      resources: ["pods"]
  # Log "pods/log", "pods/status" at Metadata level
  - level: Metadata
    resources:
    - group: ""
      resources: ["pods/log", "pods/status"]

  # Don't log requests to a configmap called "controller-leader"
  - level: None
    resources:
    - group: ""
      resources: ["configmaps"]
      resourceNames: ["controller-leader"]

  # Don't log watch requests by the "system:kube-proxy" on endpoints or services
  - level: None
    users: ["system:kube-proxy"]
    verbs: ["watch"]
    resources:
    - group: "" # core API group
      resources: ["endpoints", "services"]

  # Don't log authenticated requests to certain non-resource URL paths.
  - level: None
    userGroups: ["system:authenticated"]
    nonResourceURLs:
    - "/api*" # Wildcard matching.
    - "/version"

  # Log the request body of configmap changes in kube-system.
  - level: Request
    resources:
    - group: "" # core API group
      resources: ["configmaps"]
    # This rule only applies to resources in the "kube-system" namespace.
    # The empty string "" can be used to select non-namespaced resources.
    namespaces: ["kube-system"]

  # Log configmap and secret changes in all other namespaces at the Metadata level.
  - level: Metadata
    resources:
    - group: "" # core API group
      resources: ["secrets", "configmaps"]

  # Log all other resources in core and extensions at the Request level.
  - level: Request
    resources:
    - group: "" # core API group
    - group: "extensions" # Version of group should NOT be included.

  # A catch-all rule to log all other requests at the Metadata level.
  - level: Metadata
    # Long-running requests like watches that fall under this rule will not
    # generate an audit event in RequestReceived.
    omitStages:
      - "RequestReceived"
[root@hdss184-244 bin]# vi /opt/kubernetes/server/bin/kube-apiserver.sh
[root@hdss184-244 bin]# cat /opt/kubernetes/server/bin/kube-apiserver.sh
#!/bin/bash
./kube-apiserver \
  --apiserver-count 2 \
  --audit-log-path /data/logs/kubernetes/kube-apiserver/audit-log \
  --audit-policy-file ./conf/audit.yaml \
  --authorization-mode RBAC \
  --client-ca-file ./cert/ca.pem \
  --requestheader-client-ca-file ./cert/ca.pem \
  --enable-admission-plugins NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota \
  --etcd-cafile ./cert/ca.pem \
  --etcd-certfile ./cert/client.pem \
  --etcd-keyfile ./cert/client-key.pem \
  --etcd-servers https://192.168.184.242:2379,https://192.168.184.243:2379,https://192.168.184.244:2379 \
  --service-account-key-file ./cert/ca-key.pem \
  --service-cluster-ip-range 10.96.0.0/22 \
  --service-node-port-range 3000-29999 \
  --target-ram-mb=1024 \
  --kubelet-client-certificate ./cert/client.pem \
  --kubelet-client-key ./cert/client-key.pem \
  --log-dir  /data/logs/kubernetes/kube-apiserver \
  --tls-cert-file ./cert/apiserver.pem \
  --tls-private-key-file ./cert/apiserver-key.pem \
  --v 2
[root@hdss184-244 bin]# chmod +x /opt/kubernetes/server/bin/kube-apiserver.sh
[root@hdss184-244 bin]# mkdir -p /data/logs/kubernetes/kube-apiserver
[root@hdss184-244 bin]# vi /etc/supervisord.d/kube-apiserver.ini
[root@hdss184-244 bin]# cat /etc/supervisord.d/kube-apiserver.ini
[program:kube-apiserver-184-244]
command=/opt/kubernetes/server/bin/kube-apiserver.sh            ; the program (relative uses PATH, can take args)
numprocs=1                                                      ; number of processes copies to start (def 1)
directory=/opt/kubernetes/server/bin                            ; directory to cwd to before exec (def no cwd)
autostart=true                                                  ; start at supervisord start (default: true)
autorestart=true                                                ; retstart at unexpected quit (default: true)
startsecs=30                                                    ; number of secs prog must stay running (def. 1)
startretries=3                                                  ; max # of serial start failures (default 3)
exitcodes=0,2                                                   ; 'expected' exit codes for process (default 0,2)
stopsignal=QUIT                                                 ; signal used to kill process (default TERM)
stopwaitsecs=10                                                 ; max num secs to wait b4 SIGKILL (default 10)
user=root                                                       ; setuid to this UNIX account to run the program
redirect_stderr=true                                            ; redirect proc stderr to stdout (default false)
killasgroup=true                                                ; kill all process in a group
stopasgroup=true                                                ; stop all process in a group
stdout_logfile=/data/logs/kubernetes/kube-apiserver/apiserver.stdout.log        ; stderr log path, NONE for none; default AUTO
stdout_logfile_maxbytes=64MB                                    ; max # logfile bytes b4 rotation (default 50MB)
stdout_logfile_backups=4                                        ; # of stdout logfile backups (default 10)
stdout_capture_maxbytes=1MB                                     ; number of bytes in 'capturemode' (default 0)
stdout_events_enabled=false                                     ; emit events on stdout writes (default false)
[root@hdss184-244 bin]# supervisorctl update
kube-apiserver-6-244: added process group
[root@hdss184-244 bin]# supervisorctl status
etcd-server-6-244                RUNNING   pid 11748, uptime 5:10:52
kube-apiserver-6-244             RUNNING   pid 12408, uptime 0:00:43
```

### 11.配四层反向代理

#### 1.部署nginx

在`hdss184-241`和`hdss184-242`上：

```shell
 ~]# yum install nginx -y
```

#### 2.配置4层代理

在`hdss184-241`和`hdss184-242`上：

```shell
 ~]# vim /etc/nginx/nginx.conf
#放在http外面
stream {
    upstream kube-apiserver {
        server 192.168.184.243:6443     max_fails=3 fail_timeout=30s;
        server 192.168.184.244:6443     max_fails=3 fail_timeout=30s;
    }
    server {
        listen 7443;
        proxy_connect_timeout 2s;
        proxy_timeout 900s;
        proxy_pass kube-apiserver;
    }
}
```

#### 3.启动nginx

在`hdss184-241`和`hdss184-242`上：

```shell
 ~]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
 ~]# systemctl start nginx
 ~]# systemctl enable nginx
```

#### 4.部署keepalived服务

在`hdss184-241`和`hdss184-242`上：

```shell
[root@hdss184-241 ~]# yum install keepalived -y
```

#### 5.配置keepalived服务

在`hdss184-241`和`hdss184-242`上：

```shell
[root@hdss184-241 ~]# vi /etc/keepalived/check_port.sh
[root@hdss184-241 ~]# chmod +x /etc/keepalived/check_port.sh
[root@hdss184-241 ~]# cat /etc/keepalived/check_port.sh
#!/bin/bash
#keepalived 监控端口脚本
#使用方法：
#在keepalived的配置文件中
#vrrp_script check_port {#创建一个vrrp_script脚本,检查配置
#    script "/etc/keepalived/check_port.sh 6379" #配置监听的端口
#    interval 2 #检查脚本的频率,单位（秒）
#}
CHK_PORT=$1
if [ -n "$CHK_PORT" ];then
        PORT_PROCESS=`ss -lnt|grep $CHK_PORT|wc -l`
        if [ $PORT_PROCESS -eq 0 ];then
                echo "Port $CHK_PORT Is Not Used,End."
                exit 1
        fi
else
        echo "Check Port Cant Be Empty!"
fi
```

在`hdss184-241`上：

配置keepalived主：

```shell
[root@hdss184-241 ~]# vim /etc/keepalived/keepalived.conf 
[root@hdss184-241 ~]# cat /etc/keepalived/keepalived.conf
! Configuration File for keepalived

global_defs {
   router_id 192.168.184.241

}

vrrp_script chk_nginx {
    script "/etc/keepalived/check_port.sh 7443"
    interval 2
    weight -20
}

vrrp_instance VI_1 {
    state MASTER
    interface ens33
    virtual_router_id 251
    priority 100
    advert_int 1
    mcast_src_ip 192.168.184.241
    nopreempt

    authentication {
        auth_type PASS
        auth_pass 11111111
    }
    track_script {
         chk_nginx
    }
    virtual_ipaddress {
        192.168.184.66
    }
}
```

在`hdss184-242`上：

配置keepalived备：

```shell
[root@hdss184-242 ~]# vim /etc/keepalived/keepalived.conf 
[root@hdss184-242 ~]# cat /etc/keepalived/keepalived.conf
! Configuration File for keepalived
global_defs {
    router_id 192.168.184.242
}
vrrp_script chk_nginx {
    script "/etc/keepalived/check_port.sh 7443"
    interval 2
    weight -20
}
vrrp_instance VI_1 {
    state BACKUP
    interface ens33
    virtual_router_id 251
    mcast_src_ip 192.168.184.242
    priority 90
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 11111111
    }
    track_script {
        chk_nginx
    }
    virtual_ipaddress {
        192.168.184.66
    }
}
```

#### 6.启动keepalived

在`hdss184-241`和`hdss184-242`上：

```shell
 ~]# systemctl start keepalived.service 
 ~]# systemctl enable keepalived.service
```

#### 7.检查VIP

```shell
[root@hdss184-241 ~]# ip a |grep -A 5 ens
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP qlen 1000
    inet 192.168.6.241/24 brd 192.168.6.255 scope global eth0
    inet 192.168.6.66/32 scope global eth0
```

### 12.启动代理并检查

```shell
[root@hdss184-243 bin]# netstat -lntup|grep kube-apiser
tcp        0      0 127.0.0.1:8080          0.0.0.0:*               LISTEN      12825/./kube-apiser 
tcp6       0      0 :::6443                 :::*                    LISTEN      12825/./kube-apiser 

[root@hdss184-244 bin]# netstat -lntup|grep kube-apiser
tcp        0      0 127.0.0.1:8080          0.0.0.0:*               LISTEN      12409/./kube-apiser 
tcp6       0      0 :::6443                 :::*                    LISTEN      12409/./kube-apiser 

[root@hdss184-241 ~]# netstat -lntup|grep 7443
tcp        0      0 0.0.0.0:7443            0.0.0.0:*               LISTEN      12936/nginx: master

[root@hdss184-242 ~]# netstat -lntup|grep 7443
tcp        0      0 0.0.0.0:7443            0.0.0.0:*               LISTEN      11254/nginx: master
```





## 3.部署controller-manager

### 1.集群规划

| 主机名               | 角色               | ip            |
| :------------------- | :----------------- | :------------ |
| hdss184-243.host.com | controller-manager | 192.168.6.243 |
| hdss184-244.host.com | controller-manager | 192.168.6.244 |

注意：这里部署文档以`hdss184-243.host.com`主机为例，另外一台运算节点安装部署方法类似

### 2.创建启动脚本

hdss184-243上和hdss184-244：

```shell
[root@hdss184-243 bin]# cat /opt/kubernetes/server/bin/kube-controller-manager.sh
#!/bin/sh
./kube-controller-manager \
  --cluster-cidr 172.6.0.0/16 \
  --leader-elect true \
  --log-dir /data/logs/kubernetes/kube-controller-manager \
  --master http://127.0.0.1:8080 \
  --service-account-private-key-file ./cert/ca-key.pem \
  --service-cluster-ip-range 10.96.0.0/22 \
  --root-ca-file ./cert/ca.pem \
  --v 2
```

### 3.调整文件权限，创建目录

hdss184-243上和hdss184-244：

```shell
[root@hdss184-243 bin]# chmod +x /opt/kubernetes/server/bin/kube-controller-manager.sh
[root@hdss184-243 bin]# mkdir -p /data/logs/kubernetes/kube-controller-manager
```

### 4.创建supervisor配置

hdss184-243上和hdss184-244：

```shell
[root@hdss184-243 bin]# vi /etc/supervisord.d/kube-conntroller-manager.ini
[root@hdss184-243 bin]# cat /etc/supervisord.d/kube-conntroller-manager.ini
[program:kube-controller-manager-184.243]
command=/opt/kubernetes/server/bin/kube-controller-manager.sh                     ; the program (relative uses PATH, can take args)
numprocs=1                                                                        ; number of processes copies to start (def 1)
directory=/opt/kubernetes/server/bin                                              ; directory to cwd to before exec (def no cwd)
autostart=true                                                                    ; start at supervisord start (default: true)
autorestart=true                                                                  ; retstart at unexpected quit (default: true)
startsecs=30                                                                      ; number of secs prog must stay running (def. 1)
startretries=3                                                                    ; max # of serial start failures (default 3)
exitcodes=0,2                                                                     ; 'expected' exit codes for process (default 0,2)
stopsignal=QUIT                                                                   ; signal used to kill process (default TERM)
stopwaitsecs=10                                                                   ; max num secs to wait b4 SIGKILL (default 10)
user=root                                                                         ; setuid to this UNIX account to run the program
redirect_stderr=true                                                              ; redirect proc stderr to stdout (default false)
killasgroup=true                                                                  ; kill all process in a group
stopasgroup=true                                                                  ; stop all process in a group
stdout_logfile=/data/logs/kubernetes/kube-controller-manager/controller.stdout.log  ; stderr log path, NONE for none; default AUTO
stdout_logfile_maxbytes=64MB                                                      ; max # logfile bytes b4 rotation (default 50MB)
stdout_logfile_backups=4                                                          ; # of stdout logfile backups (default 10)
stdout_capture_maxbytes=1MB                                                       ; number of bytes in 'capturemode' (default 0)
stdout_events_enabled=false                                                       ; emit events on stdout writes (default false)
```

### 5.启动服务并检查

hdss184-243上和hdss184-244：

```shell
[root@hdss184-243 bin]# supervisorctl update
kube-controller-manager-6.243: added process group
[root@hdss184-243 bin]# supervisorctl status
etcd-server-6-243                RUNNING   pid 12112, uptime 23:36:23
kube-apiserver-6-243             RUNNING   pid 12824, uptime 18:30:46
kube-controller-manager-6.243    RUNNING   pid 14952, uptime 0:01:00
```

### 6.安装部署启动检查所有集群规划主机的kube-controller-manager服务

略



## 4.部署kube-scheduler

### 1.集群规划

| 主机名               | 角色                    | ip            |
| :------------------- | :---------------------- | :------------ |
| hdss184-243.host.com | kube-controller-manager | 192.168.6.243 |
| hdss184-244.host.com | kube-controller-manager | 192.168.6.244 |

注意：这里部署文档以`hdss184-243.host.com`主机为例，另外一台运算节点安装部署方法类似

### 2.创建启动脚本

在hdss184-243和hdss184-244上：

```shell
[root@hdss184-243 bin]# vi /opt/kubernetes/server/bin/kube-scheduler.sh
[root@hdss184-243 bin]# cat /opt/kubernetes/server/bin/kube-scheduler.sh
#!/bin/sh
./kube-scheduler \
  --leader-elect  \
  --log-dir /data/logs/kubernetes/kube-scheduler \
  --master http://127.0.0.1:8080 \
  --v 2
```

### 3.调整文件权限，创建目录

在hdss184-243和hdss184-244上：

```shell
[root@hdss184-243 bin]# chmod +x /opt/kubernetes/server/bin/kube-scheduler.sh
[root@hdss184-243 bin]# mkdir -p /data/logs/kubernetes/kube-scheduler
```

### 4.创建supervisor配置

在hdss184-243和hdss184-244上：

```shell
[root@hdss184-243 bin]# vi /etc/supervisord.d/kube-scheduler.ini
[root@hdss184-243 bin]# cat /etc/supervisord.d/kube-scheduler.ini
[program:kube-scheduler-184-243]
command=/opt/kubernetes/server/bin/kube-scheduler.sh                     ; the program (relative uses PATH, can take args)
numprocs=1                                                               ; number of processes copies to start (def 1)
directory=/opt/kubernetes/server/bin                                     ; directory to cwd to before exec (def no cwd)
autostart=true                                                           ; start at supervisord start (default: true)
autorestart=true                                                         ; retstart at unexpected quit (default: true)
startsecs=30                                                             ; number of secs prog must stay running (def. 1)
startretries=3                                                           ; max # of serial start failures (default 3)
exitcodes=0,2                                                            ; 'expected' exit codes for process (default 0,2)
stopsignal=QUIT                                                          ; signal used to kill process (default TERM)
stopwaitsecs=10                                                          ; max num secs to wait b4 SIGKILL (default 10)
user=root                                                                ; setuid to this UNIX account to run the program
redirect_stderr=true                                                     ; redirect proc stderr to stdout (default false)
killasgroup=true                                                         ; kill all process in a group
stopasgroup=true                                                         ; stop all process in a group
stdout_logfile=/data/logs/kubernetes/kube-scheduler/scheduler.stdout.log ; stderr log path, NONE for none; default AUTO
stdout_logfile_maxbytes=64MB                                             ; max # logfile bytes b4 rotation (default 50MB)
stdout_logfile_backups=4                                                 ; # of stdout logfile backups (default 10)
stdout_capture_maxbytes=1MB                                              ; number of bytes in 'capturemode' (default 0)
stdout_events_enabled=false                                              ; emit events on stdout writes (default false)
```

### 5.启动服务并检查

在hdss184-243和hdss184-244上：

```shell
[root@hdss184-243 bin]# supervisorctl update
kube-scheduler-6-243: added process group
[root@hdss184-243 bin]# supervisorctl status
etcd-server-6-243                RUNNING   pid 12112, uptime 23:52:01
kube-apiserver-6-243             RUNNING   pid 12824, uptime 18:46:24
kube-controller-manager-6.243    RUNNING   pid 14952, uptime 0:16:38
kube-scheduler-6-243             RUNNING   pid 15001, uptime 0:01:39
[root@hdss184-243 bin]# ln -s /opt/kubernetes/server/bin/kubectl /usr/bin/kubectl
```

### 6.kubectl命令自动补全

各运算节点上:

```shell
~]# yum install bash-completion -y
~]# kubectl completion bash > /etc/bash_completion.d/kubectl
```

重新登录终端即可

### 7.安装部署启动检查所有集群规划主机的kube-controller-manager服务

略

## 5.检查主控节点

在hdss184-243和hdss184-244上：

```shell
[root@hdss184-243 bin]# which kubectl 
/usr/bin/kubectl
[root@hdss184-243 bin]# kubectl get cs
NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok                   
scheduler            Healthy   ok                   
etcd-1               Healthy   {"health": "true"}   
etcd-0               Healthy   {"health": "true"}   
etcd-2               Healthy   {"health": "true"} 
```





# 第六章：部署运算节点服务

## 1.部署kubelet

### 1.集群规划

| 主机名               | 角色    | ip              |
| :------------------- | :------ | :-------------- |
| hdss184-243.host.com | kubelet | 192.168.184.243 |
| hdss184-244.host.com | kubelet | 192.168.184.244 |

注意：这里部署文档以`hdss184-243.host.com`主机为例，另外一台运算节点安装部署方法类似

### 2.签发kubelet证书

运维主机hdss184-245.host.com上：

#### 1.创建生成正事签名请求（csr）的JSON配置文件

```shell
[root@hdss184-245 certs]# vi /opt/certs/kubelet-csr.json
[root@hdss184-245 certs]# cat /opt/certs/kubelet-csr.json

{
    "CN": "k8s-kubelet",
    "hosts": [
    "127.0.0.1",
    "192.168.184.66",
    "192.168.184.243",
    "192.168.184.244",
    "192.168.184.245",
    "192.168.184.246",
    "192.168.184.247",
    "192.168.184.248"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "beijing",
            "L": "beijing",
            "O": "od",
            "OU": "ops"
        }
    ]
}
```

注意：把所有有可能用到的kubulet主机全加进去

#### 2.生成kubelet证书和私钥

```shell
[root@hdss184-245 certs]# cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=server kubelet-csr.json | cfssl-json -bare kubelet
2019/11/15 09:49:58 [INFO] generate received request
2019/11/15 09:49:58 [INFO] received CSR
2019/11/15 09:49:58 [INFO] generating key: rsa-2048
2019/11/15 09:49:59 [INFO] encoded CSR
2019/11/15 09:49:59 [INFO] signed certificate with serial number 609294877015122932833154151112494803106290808681
2019/11/15 09:49:59 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
websites. For more information see the Baseline Requirements for the Issuance and Management
of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
specifically, section 10.2.3 ("Information Requirements").
```

#### 3.检查生成证书的证书、私钥

```shell
[root@hdss184-245 certs]# ll kubelet*
-rw-r--r-- 1 root root 1098 Nov 15 09:49 kubelet.csr
-rw-r--r-- 1 root root  445 Nov 15 09:47 kubelet-csr.json
-rw------- 1 root root 1675 Nov 15 09:49 kubelet-key.pem
-rw-r--r-- 1 root root 1452 Nov 15 09:49 kubelet.pem
```

### 3.拷贝证书至各运算节点，并创建配置

hdss184-243上：

#### 1.拷贝证书，私钥，注意私钥文件属性600

```shell
[root@hdss184-243 bin]#  scp  root@hdss184-245:/opt/certs/kubelet-key.pem /opt/kubernetes/server/bin/cert/
[root@hdss184-243 bin]#  scp  root@hdss184-245:/opt/certs/kubelet.pem /opt/kubernetes/server/bin/cert/
[root@hdss184-243 bin]# ll cert/
total 32
-rw------- 1 root root 1679 Nov 14 14:18 apiserver-key.pem
-rw-r--r-- 1 root root 1598 Nov 14 14:18 apiserver.pem
-rw------- 1 root root 1679 Nov 14 14:18 ca-key.pem
-rw-r--r-- 1 root root 1346 Nov 14 14:19 ca.pem
-rw------- 1 root root 1679 Nov 14 14:19 client-key.pem
-rw-r--r-- 1 root root 1363 Nov 14 14:19 client.pem
-rw------- 1 root root 1675 Nov 15 10:01 kubelet-key.pem
-rw-r--r-- 1 root root 1452 Nov 15 10:02 kubelet.pem
```

#### 2.创建配置

##### 1.set-cluster

注意：在conf目录下

```shell
[root@hdss184-243 conf]# kubectl config set-cluster myk8s \
  --certificate-authority=/opt/kubernetes/server/bin/cert/ca.pem \
  --embed-certs=true \
  --server=https://192.168.184.66:7443 \
  --kubeconfig=kubelet.kubeconfig

Cluster "myk8s" set.
```

##### 2.set-credentials

注意：在conf目录下

```shell
[root@hdss184-243 conf]# kubectl config set-credentials k8s-node \
  --client-certificate=/opt/kubernetes/server/bin/cert/client.pem \
  --client-key=/opt/kubernetes/server/bin/cert/client-key.pem \
  --embed-certs=true \
  --kubeconfig=kubelet.kubeconfig 

User "k8s-node" set.
```

##### 3.set-context

注意：在conf目录下

```shell
[root@hdss184-243 conf]# kubectl config set-context myk8s-context \
  --cluster=myk8s \
  --user=k8s-node \
  --kubeconfig=kubelet.kubeconfig

Context "myk8s-context" created.
```

##### 4.use-context

注意：在conf目录下

```shell
[root@hdss184-243 conf]# kubectl config use-context myk8s-context --kubeconfig=kubelet.kubeconfig

Switched to context "myk8s-context".
```

##### 5.k8s-node.yaml

- 创建资源配置文件

```shell
[root@hdss184-243 conf]# vim /opt/kubernetes/server/bin/conf/k8s-node.yaml
[root@hdss184-243 conf]# cat /opt/kubernetes/server/bin/conf/k8s-node.yaml 
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: k8s-node
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:node
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: k8s-node
```

- 是集群角色用户生效

```shell
[root@hdss184-243 conf]# kubectl create -f k8s-node.yaml
```

- 查看集群角色

```shell
[root@hdss184-243 conf]# kubectl get clusterrolebinding k8s-node
NAME       AGE
k8s-node   22m
[root@hdss184-243 conf]# kubectl get clusterrolebinding k8s-node -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  creationTimestamp: "2019-11-15T02:14:34Z"
  name: k8s-node
  resourceVersion: "17884"
  selfLink: /apis/rbac.authorization.k8s.io/v1/clusterrolebindings/k8s-node
  uid: e09ed617-936f-4936-8adc-d7cc9b3bce63
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:node
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: k8s-node
```

在hdss184-244上：

```shell
[root@hdss184-244 bin]# scp root@hdss184-243:/opt/kubernetes/server/bin/conf/kubelet.kubeconfig /opt/kubernetes/server/bin/conf/
```

可略

```shell
[root@hdss184-244 bin]# scp  root@hdss184-243:/opt/kubernetes/server/bin/conf/k8s-node.yaml /opt/kubernetes/server/bin/conf/
```

### 4.准备pause基础镜像

运维主机hdss184-244.host.com上：

#### 1.下载

```shell
[root@hdss184-245 certs]# docker pull kubernetes/pause
Using default tag: latest
latest: Pulling from kubernetes/pause
4f4fb700ef54: Pull complete 
b9c8ec465f6b: Pull complete 
Digest: sha256:b31bfb4d0213f254d361e0079deaaebefa4f82ba7aa76ef82e90b4935ad5b105
Status: Downloaded newer image for kubernetes/pause:latest
docker.io/kubernetes/pause:latest
```

#### 2.打标签

```shell
[root@hdss184-245 certs]# docker images|grep pause
kubernetes/pause                latest                     f9d5de079539        5 years ago         240kB
[root@hdss184-245 certs]# docker tag f9d5de079539 harbor.od.com/public/pause:latest
```

#### 3.推送私有仓库(harbor)中

```shell
[root@hdss184-245 certs]# docker push harbor.od.com/public/pause:latest
The push refers to repository [harbor.od.com/public/pause]
5f70bf18a086: Mounted from public/nginx 
e16a89738269: Pushed 
latest: digest: sha256:b31bfb4d0213f254d361e0079deaaebefa4f82ba7aa76ef82e90b4935ad5b105 size: 938
```

### 5.创建kubelet启动脚本

在hdss184-243：

```shell
[root@hdss184-243 conf]# vi /opt/kubernetes/server/bin/kubelet.sh
[root@hdss184-243 conf]# cat /opt/kubernetes/server/bin/kubelet.sh
#!/bin/sh
./kubelet \
  --anonymous-auth=false \
  --cgroup-driver systemd \
  --cluster-dns 10.96.0.2 \
  --cluster-domain cluster.local \
  --runtime-cgroups=/systemd/system.slice \
  --kubelet-cgroups=/systemd/system.slice \
  --fail-swap-on="false" \
  --client-ca-file ./cert/ca.pem \
  --tls-cert-file ./cert/kubelet.pem \
  --tls-private-key-file ./cert/kubelet-key.pem \
  --hostname-override hdss184-243.host.com \
  --image-gc-high-threshold 20 \
  --image-gc-low-threshold 10 \
  --kubeconfig ./conf/kubelet.kubeconfig \
  --log-dir /data/logs/kubernetes/kube-kubelet \
  --pod-infra-container-image harbor.od.com/public/pause:latest \
  --root-dir /data/kubelet
```

注意：kubelet集群各主机的启动脚本略有不同，部署其节点时注意修改

```
hostname-override
```

### 6.检查配置，权限，创建日志目录

在hdss184-243：

```shell
[root@hdss184-243 conf]# ls -l|grep kubelet.kubeconfig 
-rw------- 1 root root 6202 Nov 15 10:11 kubelet.kubeconfig

[root@hdss184-243 conf]# chmod +x /opt/kubernetes/server/bin/kubelet.sh
[root@hdss184-243 conf]# mkdir -p /data/logs/kubernetes/kube-kubelet /data/kubelet
```

### 7.创建supervisor配置

在hdss184-243：

```shell
[root@hdss184-243 conf]# vi /etc/supervisord.d/kube-kubelet.ini
[root@hdss184-243 conf]# cat /etc/supervisord.d/kube-kubelet.ini

[program:kube-kubelet-184-243]
command=/opt/kubernetes/server/bin/kubelet.sh     ; the program (relative uses PATH, can take args)
numprocs=1                                        ; number of processes copies to start (def 1)
directory=/opt/kubernetes/server/bin              ; directory to cwd to before exec (def no cwd)
autostart=true                                    ; start at supervisord start (default: true)
autorestart=true                                ; retstart at unexpected quit (default: true)
startsecs=30                                      ; number of secs prog must stay running (def. 1)
startretries=3                                    ; max # of serial start failures (default 3)
exitcodes=0,2                                     ; 'expected' exit codes for process (default 0,2)
stopsignal=QUIT                                   ; signal used to kill process (default TERM)
stopwaitsecs=10                                   ; max num secs to wait b4 SIGKILL (default 10)
user=root                                         ; setuid to this UNIX account to run the program
redirect_stderr=true                              ; redirect proc stderr to stdout (default false)
killasgroup=true                                  ; kill all process in a group
stopasgroup=true                                  ; stop all process in a group
stdout_logfile=/data/logs/kubernetes/kube-kubelet/kubelet.stdout.log   ; stderr log path, NONE for none; default AUTO
stdout_logfile_maxbytes=64MB                      ; max # logfile bytes b4 rotation (default 50MB)
stdout_logfile_backups=4                          ; # of stdout logfile backups (default 10)
stdout_capture_maxbytes=1MB                       ; number of bytes in 'capturemode' (default 0)
stdout_events_enabled=false                       ; emit events on stdout writes (default false)
```

### 8.启动服务并检查

- 启动服务并检查

```shell
[root@hdss184-243 conf]# supervisorctl update
kube-kubelet-6-243: added process group
[root@hdss184-243 conf]# supervisorctl status
etcd-server-6-243                RUNNING   pid 12112, uptime 1 day, 1:43:37
kube-apiserver-6-243             RUNNING   pid 12824, uptime 20:38:00
kube-controller-manager-6.243    RUNNING   pid 14952, uptime 2:08:14
kube-kubelet-6-243               RUNNING   pid 15398, uptime 0:01:25
kube-scheduler-6-243             RUNNING   pid 15001, uptime 1:53:15

[root@hdss184-243 conf]# tail -fn 200 /data/logs/kubernetes/kube-kubelet/kubelet.stdout.log
```

### 9.检查运算节点

```shell
[root@hdss184-243 conf]# kubectl get nodes
NAME                 STATUS   ROLES    AGE     VERSION
hdss184-243.host.com   Ready    <none>   16m     v1.15.2
hdss184-244.host.com   Ready    <none>   2m12s   v1.15.2
[root@hdss184-243 conf]# kubectl get nodes -o wide
NAME                 STATUS   ROLES    AGE     VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION               CONTAINER-RUNTIME
hdss184-243.host.com   Ready    <none>   17m     v1.15.2   192.168.6.243   <none>        CentOS Linux 7 (Core)   3.10.0-693.21.1.el7.x86_64   docker://19.3.4
hdss184-244.host.com   Ready    <none>   2m45s   v1.15.2   192.168.6.244   <none>        CentOS Linux 7 (Core)   3.10.0-693.21.1.el7.x86_64   docker://19.3.4
```

- 给node节点打标签

  ```shell
  [root@hdss184-243 conf]# kubectl label node hdss184-243.host.com node-role.kubernetes.io/node=
  node/hdss184-243.host.com labeled
  [root@hdss184-243 conf]# kubectl label node hdss184-243.host.com node-role.kubernetes.io/master=
  node/hdss184-243.host.com labeled
  [root@hdss184-243 conf]# kubectl label node hdss184-244.host.com node-role.kubernetes.io/node=
  node/hdss184-244.host.com labeled
  [root@hdss184-243 conf]# kubectl label node hdss184-244.host.com node-role.kubernetes.io/master=
  node/hdss184-244.host.com labeled
  ```

### 10.安装部署启动检查所有集群规划主机的kube-kubelet服务

略

### 11.检查所有运算节点

```shell
[root@hdss184-243 conf]# kubectl get nodes -o wide
NAME                 STATUS   ROLES         AGE     VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION               CONTAINER-RUNTIME
hdss184-243.host.com   Ready    master,node   20m     v1.15.2   192.168.184.243   <none>        CentOS Linux 7 (Core)   3.10.0-693.21.1.el7.x86_64   docker://19.3.4
hdss184-244.host.com   Ready    master,node   6m34s   v1.15.2   192.168.184.244   <none>        CentOS Linux 7 (Core)   3.10.0-693.21.1.el7.x86_64   docker://19.3.4
[root@hdss184-243 conf]# kubectl get nodes
NAME                 STATUS   ROLES         AGE     VERSION
hdss184-243.host.com   Ready    master,node   21m     v1.15.2
hdss184-244.host.com   Ready    master,node   6m42s   v1.15.2
```





## 2.部署kube-proxy

### 1.集群规划

| 主机名               | 角色       | ip              |
| :------------------- | :--------- | :-------------- |
| hdss184-243.host.com | kube-proxy | 192.168.184.243 |
| hdss184-244.host.com | kube-proxy | 192.168.184.244 |

注意：这里部署文档以hdss184-243.host.com主机为例，另外一台运算节点安装部署方法类似

### 2.签发kube-proxy证书

运维主机hdss184-245.host.com上：

#### 1.创建生成证书签名请求（csr）的JSON文件

```shell
[root@hdss184-245 certs]# vi /opt/certs/kube-proxy-csr.json
[root@hdss184-245 certs]# cat kube-proxy-csr.json 
{
    "CN": "system:kube-proxy",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "beijing",
            "L": "beijing",
            "O": "od",
            "OU": "ops"
        }
    ]
}
```

#### 2.生成kubelet证书和私钥

```shell
[root@hdss184-245 certs]# cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=client kube-proxy-csr.json |cfssl-json -bare kube-proxy-client
2019/11/15 12:28:23 [INFO] generate received request
2019/11/15 12:28:23 [INFO] received CSR
2019/11/15 12:28:23 [INFO] generating key: rsa-2048
2019/11/15 12:28:24 [INFO] encoded CSR
2019/11/15 12:28:24 [INFO] signed certificate with serial number 499210659443234759487015805632579178164834077987
2019/11/15 12:28:24 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
websites. For more information see the Baseline Requirements for the Issuance and Management
of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
specifically, section 10.2.3 ("Information Requirements").
```

注意：这里的clent不能与其他的通用，上面CN变了，`"CN": "system:kube-proxy",`

#### 3.检查生成证书的证书、私钥

```shell
[root@hdss184-245 certs]# ll kube-proxy*
-rw-r--r-- 1 root root 1005 Nov 15 12:28 kube-proxy-client.csr
-rw------- 1 root root 1675 Nov 15 12:28 kube-proxy-client-key.pem
-rw-r--r-- 1 root root 1375 Nov 15 12:28 kube-proxy-client.pem
-rw-r--r-- 1 root root  267 Nov 15 12:28 kube-proxy-csr.json
```

### 3.拷贝证书至各个运算节点，并创建配置

#### 1.拷贝`kube-proxy-client-key.pem`和`kube-proxy-client.pem`至运算节点

```shell
[root@hdss184-243 conf]# scp  root@hdss184-245:/opt/certs/kube-proxy-client-key.pem /opt/kubernetes/server/bin/cert/
[root@hdss184-243 conf]# scp  root@hdss184-245:/opt/certs/kube-proxy-client.pem /opt/kubernetes/server/bin/cert/
```

#### 2.创建配置

##### 1.set-cluster

注意：在conf目录下

```shell
[root@hdss184-243 conf]# kubectl config set-cluster myk8s \
  --certificate-authority=/opt/kubernetes/server/bin/cert/ca.pem \
  --embed-certs=true \
  --server=https://192.168.184.66:7443 \
  --kubeconfig=kube-proxy.kubeconfig
```

##### 2.set-credentials

注意：在conf目录下

```shell
[root@hdss184-243 conf]# kubectl config set-credentials kube-proxy \
  --client-certificate=/opt/kubernetes/server/bin/cert/kube-proxy-client.pem \
  --client-key=/opt/kubernetes/server/bin/cert/kube-proxy-client-key.pem \
  --embed-certs=true \
  --kubeconfig=kube-proxy.kubeconfig
```

##### 3.set-context

注意：在conf目录下

```shell
[root@hdss184-243 conf]# kubectl config set-context myk8s-context \
  --cluster=myk8s \
  --user=kube-proxy \
  --kubeconfig=kube-proxy.kubeconfig
```

##### 4.use-context

注意：在conf目录下

```shell
[root@hdss184-243 conf]# kubectl config use-context myk8s-context --kubeconfig=kube-proxy.kubeconfig
```



##### 244 上直接拷贝kube-proxy.kubeconfig文件，省去创建配置

```shell
[root@hdss184-244 conf]# scp root@192.168.184.243:/opt/kubernetes/server/bin/conf/kube-proxy.kubeconfig /opt/kubernetes/server/bin/conf
```



### 4.创建kube-proxy启动脚本

在hdss184-243上：

- 加载ipvs模块

```shell
[root@hdss184-243 conf]# vi /root/ipvs.sh
[root@hdss184-243 conf]# cat /root/ipvs.sh
#!/bin/bash
ipvs_mods_dir="/usr/lib/modules/$(uname -r)/kernel/net/netfilter/ipvs"
for i in $(ls $ipvs_mods_dir|grep -o "^[^.]*")
do
  /sbin/modinfo -F filename $i &>/dev/null
  if [ $? -eq 0 ];then
    /sbin/modprobe $i
  fi
done

[root@hdss184-243 bin]# sh /root/ipvs.sh 

[root@hdss184-243 bin]# lsmod |grep ip_vs
[root@hdss184-243 bin]# lsmod |grep ip_vs
ip_vs_wlc              12519  0 
ip_vs_sed              12519  0 
ip_vs_pe_sip           12697  0 
nf_conntrack_sip       33860  1 ip_vs_pe_sip
ip_vs_nq               12516  0 
ip_vs_lc               12516  0 
ip_vs_lblcr            12922  0 
ip_vs_lblc             12819  0 
ip_vs_ftp              13079  0 
ip_vs_dh               12688  0 
ip_vs_sh               12688  0 
ip_vs_wrr              12697  0 
ip_vs_rr               12600  0 
ip_vs                 141092  24 ip_vs_dh,ip_vs_lc,ip_vs_nq,ip_vs_rr,ip_vs_sh,ip_vs_ftp,ip_vs_sed,ip_vs_wlc,ip_vs_wrr,ip_vs_pe_sip,ip_vs_lblcr,ip_vs_lblc
nf_nat                 26787  3 ip_vs_ftp,nf_nat_ipv4,nf_nat_masquerade_ipv4
nf_conntrack          133387  8 ip_vs,nf_nat,nf_nat_ipv4,xt_conntrack,nf_nat_masquerade_ipv4,nf_conntrack_netlink,nf_conntrack_sip,nf_conntrack_ipv4
libcrc32c              12644  4 xfs,ip_vs,nf_nat,nf_conntrack
```

- 创建启动脚本

```shell
[root@hdss184-243 conf]# vi /opt/kubernetes/server/bin/kube-proxy.sh
[root@hdss184-243 conf]# cat /opt/kubernetes/server/bin/kube-proxy.sh
#!/bin/sh
./kube-proxy \
  --cluster-cidr 172.6.0.0/16 \
  --hostname-override hdss184-244.host.com \
  --proxy-mode=ipvs \
  --ipvs-scheduler=nq \
  --kubeconfig ./conf/kube-proxy.kubeconfig
```

注意：kube-proxy集群各主机的启动脚本略有不同，部署其他节点时注意修改。

### 5.检查配置，权限，创建日志目录

在hdss184-243上：

```shell
[root@hdss184-243 conf]# ls -l|grep kube-proxy.kubeconfig 
-rw------- 1 root root 6207 Nov 15 12:32 kube-proxy.kubeconfig

[root@hdss184-243 conf]# chmod +x /opt/kubernetes/server/bin/kube-proxy.sh
[root@hdss184-243 conf]# mkdir -p /data/logs/kubernetes/kube-proxy
```

### 6.创建supervisor配置

```shell
[root@hdss184-243 conf]# vi /etc/supervisord.d/kube-proxy.ini
[root@hdss184-243 conf]# cat /etc/supervisord.d/kube-proxy.ini
[program:kube-proxy-184-243]
command=/opt/kubernetes/server/bin/kube-proxy.sh                     ; the program (relative uses PATH, can take args)
numprocs=1                                                           ; number of processes copies to start (def 1)
directory=/opt/kubernetes/server/bin                                 ; directory to cwd to before exec (def no cwd)
autostart=true                                                       ; start at supervisord start (default: true)
autorestart=true                                                     ; retstart at unexpected quit (default: true)
startsecs=30                                                         ; number of secs prog must stay running (def. 1)
startretries=3                                                       ; max # of serial start failures (default 3)
exitcodes=0,2                                                        ; 'expected' exit codes for process (default 0,2)
stopsignal=QUIT                                                      ; signal used to kill process (default TERM)
stopwaitsecs=10                                                      ; max num secs to wait b4 SIGKILL (default 10)
user=root                                                            ; setuid to this UNIX account to run the program
redirect_stderr=true                                                 ; redirect proc stderr to stdout (default false)
killasgroup=true                                                     ; kill all process in a group
stopasgroup=true                                                     ; stop all process in a group
stdout_logfile=/data/logs/kubernetes/kube-proxy/proxy.stdout.log     ; stderr log path, NONE for none; default AUTO
stdout_logfile_maxbytes=64MB                                         ; max # logfile bytes b4 rotation (default 50MB)
stdout_logfile_backups=4                                             ; # of stdout logfile backups (default 10)
stdout_capture_maxbytes=1MB                                          ; number of bytes in 'capturemode' (default 0)
stdout_events_enabled=false                                          ; emit events on stdout writes (default false)
```

### 7.启动服务并检查

```shell
[root@hdss184-243 conf]# supervisorctl update
kube-proxy-6-243: added process group

[root@hdss184-243 conf]# supervisorctl status
etcd-server-6-243                RUNNING   pid 12112, uptime 1 day, 8:13:06
kube-apiserver-6-243             RUNNING   pid 12824, uptime 1 day, 3:07:29
kube-controller-manager-6.243    RUNNING   pid 14952, uptime 8:37:43
kube-kubelet-6-243               RUNNING   pid 15398, uptime 6:30:54
kube-proxy-6-243                 RUNNING   pid 8055, uptime 0:01:19
kube-scheduler-6-243             RUNNING   pid 15001, uptime 8:22:44
[root@hdss184-243 conf]# yum install ipvsadm -y
[root@hdss184-243 ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.96.0.1:443 nq
  -> 192.168.6.243:6443           Masq    1      0          0         
  -> 192.168.6.244:6443           Masq    1      0          0 

[root@hdss184-243 ~]# kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   24h
```





# 第七章：完成部署并验证集群

- 创建配置清单

```shell
[root@hdss184-243 conf]# cat /root/nginx-ds.yaml 
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: nginx-ds
spec:
  template:
    metadata:
      labels:
        app: nginx-ds
    spec:
      containers:
      - name: my-nginx
        image: harbor.od.com/public/nginx:v1.7.9
        ports:
        - containerPort: 80
```

- 集群运算节点登录harbor

```shell
[root@hdss184-243 conf]# docker login harbor.od.com
Username: admin  
Password: 

[root@hdss184-244 conf]# docker login harbor.od.com
Username: admin
Password:
```

- 创建pod

```shell
[root@hdss184-243 conf]# kubectl create -f nginx-ds.yaml
```

- 创建pod

```shell
[root@hdss184-243 conf]# kubectl get pods 
NAME             READY   STATUS    RESTARTS   AGE
nginx-ds-ftxpz   1/1     Running   0          2m50s
nginx-ds-wb6wt   1/1     Running   0          2m51s

[root@hdss184-243 conf]# kubectl get cs
NAME                 STATUS    MESSAGE              ERROR
scheduler            Healthy   ok                   
controller-manager   Healthy   ok                   
etcd-0               Healthy   {"health": "true"}   
etcd-1               Healthy   {"health": "true"}   
etcd-2               Healthy   {"health": "true"}
```





# 第二章：kubenetes的CNI网络插件-flannel

**没有CNI网络插件，跨节点网络不通。**

[root@hdss184-243 ~]#  kubectl get pod -o wide
NAME             READY   STATUS    RESTARTS   AGE    IP            NODE                   NOMINATED NODE   READINESS GATES
nginx-ds-qc4h5   1/1     Running   0          6h9m   172.6.244.2   hdss184-244.host.com   <none>           <none>
nginx-ds-vmkrq   1/1     Running   0          6h1m   172.6.243.2   hdss184-243.host.com   <none>           <none>
[root@hdss184-243 ~]# 
[root@hdss184-243 ~]# 
[root@hdss184-243 ~]# ping 172.6.243.2
PING 172.6.243.2 (172.6.243.2) 56(84) bytes of data.
64 bytes from 172.6.243.2: icmp_seq=1 ttl=64 time=0.182 ms
64 bytes from 172.6.243.2: icmp_seq=2 ttl=64 time=0.074 ms
64 bytes from 172.6.243.2: icmp_seq=3 ttl=64 time=0.127 ms
^C
--- 172.6.243.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1999ms
rtt min/avg/max/mdev = 0.074/0.127/0.182/0.045 ms
[root@hdss184-243 ~]# ping 172.6.244.2
PING 172.6.244.2 (172.6.244.2) 56(84) bytes of data.

^C
--- 172.6.244.2 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2001ms

[root@hdss184-243 ~]# 



kubernetes设计了网络模型，但却将它的实现交给了网络插件，CNI网络插件最主要的功能就是实现POD资源能够跨宿主机进行通信

常见的网络插件：

- flannel
- calico
- contiv
- opencontrail
- NSX-T
- Kube-router

### 1.集群规划：

| 主机名               | 角色    | ip              |
| :------------------- | :------ | :-------------- |
| hdss184-243.host.com | flannel | 192.168.184.243 |
| hdss184-244.host.com | flannel | 192.168.184.244 |

注意：这里部署文档以`hdss184-243.host.com`主机为例，另外一台运算节点安装部署方法类似

### 2.下载软件，解压，做软连接

[fiannel官方下载地址](https://github.com/coreos/flannel/releases)

hdss184-243.host.com上：

```shell
[root@hdss184-243 ~]# cd /opt/src/
[root@hdss184-243 src]# wget https://github.com/coreos/flannel/releases/download/v0.11.0/flannel-v0.11.0-linux-amd64.tar.gz

[root@hdss184-243 src]# mkdir /opt/flannel-v0.11.0
[root@hdss184-243 src]# tar xf flannel-v0.11.0-linux-amd64.tar.gz -C /opt/flannel-v0.11.0/
[root@hdss184-243 src]# ln -s /opt/flannel-v0.11.0/ /opt/flannel
```

### 3.最终目录结构

```shell
[root@hdss184-243 flannel]# tree /opt -L 2
/opt
├── containerd
│   ├── bin
│   └── lib
├── etcd -> /opt/etcd-v3.1.20/
├── etcd-v3.1.20
│   ├── certs
│   ├── Documentation
│   ├── etcd
│   ├── etcdctl
│   ├── etcd-server-startup.sh
│   ├── README-etcdctl.md
│   ├── README.md
│   └── READMEv2-etcdctl.md
├── flannel -> /opt/flannel-v0.11.0/
├── flannel-v0.11.0
│   ├── cert
│   ├── flanneld
│   ├── mk-docker-opts.sh
│   └── README.md
├── kubernetes -> /opt/kubernetes-v1.15.2/
├── kubernetes-v1.15.2
│   ├── addons
│   ├── LICENSES
│   └── server
├── rh
└── src
    ├── etcd-v3.1.20-linux-amd64.tar.gz
    ├── flannel-v0.11.0-linux-amd64.tar.gz
    └── kubernetes-server-linux-amd64-v1.15.2.tar.gz
```

### 4.拷贝证书

```shell
[root@hdss184-243 src]# mkdir /opt/flannel/cert/
[root@hdss184-243 src]# cd /opt/flannel/cert/
[root@hdss184-243 flannel]# scp  root@hdss184-245:/opt/certs/ca.pem /opt/flannel/cert/

[root@hdss184-243 flannel]# scp  root@hdss184-245:/opt/certs/client.pem /opt/flannel/cert/

[root@hdss184-243 flannel]# scp  root@hdss184-245:/opt/certs/client-key.pem /opt/flannel/cert/
```

### 5.创建配置

```shell
[root@hdss184-243 flannel]# vi subnet.env
[root@hdss184-243 flannel]# cat subnet.env
FLANNEL_NETWORK=172.6.0.0/16
FLANNEL_SUBNET=172.6.243.1/24
FLANNEL_MTU=1500
FLANNEL_IPMASQ=false
```

注意：flannel集群各主机的配置略有不同，部署其他节点时之一修改。

### 6.创建启动脚本

```shell
[root@hdss184-243 flannel]# vi flanneld.sh 
[root@hdss184-243 flannel]# cat flanneld.sh
#!/bin/sh
./flanneld \
  --public-ip=192.168.184.243 \
  --etcd-endpoints=https://192.168.184.242:2379,https://192.168.184.243:2379,https://192.168.184.244:2379 \
  --etcd-keyfile=./cert/client-key.pem \
  --etcd-certfile=./cert/client.pem \
  --etcd-cafile=./cert/ca.pem \
  --iface=ens33 \
  --subnet-file=./subnet.env \
  --healthz-port=2401
```

注意：flannel集群各主机的启动脚本略有不同，部署其他节点时注意修改

### 7.检查配置，权限，创建日志目录

```shell
[root@hdss184-243 flannel]# chmod +x /opt/flannel/flanneld.sh 
[root@hdss184-243 flannel]# mkdir -p /data/logs/flanneld
```

### 8.创建supervisor配置

```shell
[root@hdss184-243 flannel]# cat /etc/supervisord.d/flannel.ini
[program:flanneld-184-243]
command=/opt/flannel/flanneld.sh                             ; the program (relative uses PATH, can take args)
numprocs=1                                                   ; number of processes copies to start (def 1)
directory=/opt/flannel                                       ; directory to cwd to before exec (def no cwd)
autostart=true                                               ; start at supervisord start (default: true)
autorestart=true                                             ; retstart at unexpected quit (default: true)
startsecs=30                                                 ; number of secs prog must stay running (def. 1)
startretries=3                                               ; max # of serial start failures (default 3)
exitcodes=0,2                                                ; 'expected' exit codes for process (default 0,2)
stopsignal=QUIT                                              ; signal used to kill process (default TERM)
stopwaitsecs=10                                              ; max num secs to wait b4 SIGKILL (default 10)
user=root                                                    ; setuid to this UNIX account to run the program
redirect_stderr=true                                         ; redirect proc stderr to stdout (default false)
killasgroup=true                                             ; kill all process in a group
stopasgroup=true                                             ; stop all process in a group
stdout_logfile=/data/logs/flanneld/flanneld.stdout.log       ; stderr log path, NONE for none; default AUTO
stdout_logfile_maxbytes=64MB                                 ; max # logfile bytes b4 rotation (default 50MB)
stdout_logfile_backups=4                                     ; # of stdout logfile backups (default 10)
stdout_capture_maxbytes=1MB                                  ; number of bytes in 'capturemode' (default 0)
stdout_events_enabled=false                                  ; emit events on stdout writes (default false)
killasgroup=true
stopasgroup=true
```

注意：flannel集群各主机的启动脚本略有不同，部署其他节点时注意修改

supervisord管理进程的时候，默认是不kill子进程的，需要在对应的服务.ini配置文件中加以下两个配置：

```shell
killasgroup=true
stopasgroup=true
```

### 9.操作etcd，增加host-gw

```shell
[root@hdss184-243 etcd]# ./etcdctl set /coreos.com/network/config '{"Network": "172.6.0.0/16", "Backend": {"Type": "host-gw"}}'
{"Network": "172.6.0.0/16", "Backend": {"Type": "host-gw"}}

[root@hdss184-243 etcd]# ./etcdctl get /coreos.com/network/config
{"Network": "172.6.0.0/16", "Backend": {"Type": "host-gw"}}
```

Flannel的host-gw模型

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_b12d9d33897632ec412af7644b40298a_r.png)

本质就是增加静态路由：

```shell
[root@hdss184-243 flannel]# route add -net 172.6.244.0/24 gw 192.168.6.244 dev eth0

[root@hdss184-244 flannel]# route add -net 172.6.243.0/24 gw 192.168.6.244 dev eth0

[root@hdss184-243 ~]# iptables -t filter -I FORWARD -d 172.5.243.0/24 -j ACCEPT

[root@hdss184-244 ~]# iptables -t filter -I FORWARD -d 172.5.244.0/24 -j ACCEPT
```

附：flannel的其他网络模型

VxLAN模型

```shell
'{"Network": "172.6.0.0/16", "Backend": {"Type": "VxLAN"}}'
```

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_f125be65a578dc5128c832c86f3edb9e_r.png)

```shell
1、更改为VxLAN模型

1.1 拆除现有的host-gw模式

1.1.1查看当前flanneld工作模式
[root@hdss184-243 flannel]# cd /opt/etcd
[root@hdss184-243 etcd]# ./etcdctl get /coreos.com/network/config
{"Network": "172.6.0.0/16", "Backend": {"Type": "host-gw"}}

1.1.2关闭flanneld
[root@hdss184-243 flannel]# supervisorctl stop flanneld-6-243
flanneld-6-243: stopped

[root@hdss184-244 flannel]# supervisorctl stop flanneld-6-244
flanneld-6-244: stopped


1.1.3检查关闭情况
[root@hdss184-243 flannel]# ps -aux | grep flanneld
root     12784  0.0  0.0 112660   964 pts/0    S+   10:30   0:00 grep --color=auto flanneld

[root@hdss184-244 flannel]# ps -aux | grep flanneld
root     12379  0.0  0.0 112660   972 pts/0    S+   10:31   0:00 grep --color=auto flanneld

1.2拆除静态规则

1.2.1查看当前路由

[root@hdss184-243 flannel]# route -n 
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.6.254   0.0.0.0         UG    100    0        0 eth0
172.6.243.0     0.0.0.0         255.255.255.0   U     0      0        0 docker0
172.6.244.0     192.168.6.244   255.255.255.0   UG    0      0        0 eth0
192.168.6.0     0.0.0.0         255.255.255.0   U     100    0        0 eth0


[root@hdss184-244 flannel]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.6.254   0.0.0.0         UG    100    0        0 eth0
172.6.243.0     192.168.6.243   255.255.255.0   UG    0      0        0 eth0
172.6.244.0     0.0.0.0         255.255.255.0   U     0      0        0 docker0
192.168.6.0     0.0.0.0         255.255.255.0   U     100    0        0 eth0



1.2.2拆除host-gw静态路由规则：
[root@hdss184-243 flannel]# route del -net  172.6.244.0/24 gw 192.168.6.244 dev eth0     # 删除规则的方法
[root@hdss184-244 flannel]# route del -net  172.6.243.0/24 gw 192.168.6.243 dev eth0        # 删除规则的方法


1.2.3查看拆除后的路由信息
[root@hdss184-243 flannel]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.6.254   0.0.0.0         UG    100    0        0 eth0
172.6.243.0     0.0.0.0         255.255.255.0   U     0      0        0 docker0
192.168.6.0     0.0.0.0         255.255.255.0   U     100    0        0 eth0

[root@hdss184-244 flannel]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.6.254   0.0.0.0         UG    100    0        0 eth0
172.6.244.0     0.0.0.0         255.255.255.0   U     0      0        0 docker0
192.168.6.0     0.0.0.0         255.255.255.0   U     100    0        0 eth0


1.3查看后端模式，并删除host-gw模型
[root@hdss184-243 flannel]# cd /opt/etcd
[root@hdss184-243 etcd]# ./etcdctl get /coreos.com/network/config
{"Network": "172.6.0.0/16", "Backend": {"Type": "host-gw"}}

[root@hdss184-243 etcd]# ./etcdctl rm /coreos.com/network/config
Error:  x509: certificate signed by unknown authority
[root@hdss184-243 etcd]# ./etcdctl get /coreos.com/network/config
{"Network": "172.6.0.0/16", "Backend": {"Type": "host-gw"}}
[root@hdss184-243 etcd]# ./etcdctl rm /coreos.com/network/config
PrevNode.Value: {"Network": "172.6.0.0/16", "Backend": {"Type": "host-gw"}}
[root@hdss184-243 etcd]# ./etcdctl get /coreos.com/network/config
Error:  100: Key not found (/coreos.com/network/config) [13]


2、更改后端模式为VxLAN

2.1后端模式添加为VxLAN
[root@hdss184-243 etcd]# ./etcdctl set /coreos.com/network/config '{"Network": "172.6.0.0/16", "Backend": {"Type": "VxLAN"}}'
{"Network": "172.6.0.0/16", "Backend": {"Type": "VxLAN"}}

2.2查看模式
[root@hdss184-243 etcd]# ./etcdctl get /coreos.com/network/config
{"Network": "172.6.0.0/16", "Backend": {"Type": "VxLAN"}}

2.3查看flanneld（VxLan）应用前网卡信息
[root@hdss184-243 etcd]# ifconfig 
docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.6.243.1  netmask 255.255.255.0  broadcast 172.6.243.255

eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.6.243  netmask 255.255.255.0  broadcast 192.168.6.255

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0

vethba8da49: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500


2.4启动flanneld
[root@hdss184-243 etcd]# supervisorctl start flanneld-6-243
[root@hdss184-244 flannel]# supervisorctl start flanneld-6-244


2.5查看flanneld（VxLan）应用生效后网卡信息
[root@hdss184-243 etcd]# ifconfig 
docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.6.243.1  netmask 255.255.255.0  broadcast 172.6.243.255

eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.6.243  netmask 255.255.255.0  broadcast 192.168.6.255

flannel.1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1450
        inet 172.6.243.0  netmask 255.255.255.255  broadcast 0.0.0.0

lo: flainet 127.0.0.1  netmask 255.0.0.0

vethba8da49: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500



2.6查看静态路由信息    
[root@hdss184-243 flannel]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.6.254   0.0.0.0         UG    100    0        0 eth0
172.6.243.0     0.0.0.0         255.255.255.0   U     0      0        0 docker0
172.6.244.0     172.6.244.0     255.255.255.0   UG    0      0        0 flannel.1
192.168.6.0     0.0.0.0         255.255.255.0   U     100    0        0 eth0

[root@hdss184-244 flannel]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.6.254   0.0.0.0         UG    100    0        0 eth0
172.6.243.0     172.6.243.0     255.255.255.0   UG    0      0        0 flannel.1
172.6.244.0     0.0.0.0         255.255.255.0   U     0      0        0 docker0
192.168.6.0     0.0.0.0         255.255.255.0   U     100    0        0 eth0
3.恢复成host-gw模型：
3.1查看当前flanneld工作模式
[root@hdss184-243 flannel]# cd /opt/etcd
[root@hdss184-243 etcd]# ./etcdctl get /coreos.com/network/config
{"Network": "172.6.0.0/16", "Backend": {"Type": "VxLAN"}}

3.2关闭flanneld
[root@hdss184-243 flannel]# supervisorctl stop flanneld-6-243
flanneld-6-243: stopped

[root@hdss184-244 flannel]# supervisorctl stop flanneld-6-244
flanneld-6-244: stopped

3.3删除后端flanneld（VxLAN）模型
[root@hdss184-243 etcd]# ./etcdctl rm /coreos.com/network/config
PrevNode.Value: {"Network": "172.6.0.0/16", "Backend": {"Type": "VxLAN"}}

3.4查看后端当前模型
[root@hdss184-243 etcd]# ./etcdctl get /coreos.com/network/config
Error:  100: Key not found (/coreos.com/network/config) [17]

3.5后端更改host-gw模型
[root@hdss184-243 etcd]#  ./etcdctl set /coreos.com/network/config '{"Network": "172.6.0.0/16", "Backend": {"Type": "host-gw"}}'
{"Network": "172.6.0.0/16", "Backend": {"Type": "host-gw"}}
[root@hdss184-243 etcd]# ./etcdctl get /coreos.com/network/config
{"Network": "172.6.0.0/16", "Backend": {"Type": "host-gw"}}

3.6查看静态路由
[root@hdss184-243 etcd]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.6.254   0.0.0.0         UG    100    0        0 eth0
172.6.243.0     0.0.0.0         255.255.255.0   U     0      0        0 docker0
172.6.244.0     192.168.6.244   255.255.255.0   UG    0      0        0 eth0
192.168.6.0     0.0.0.0         255.255.255.0   U     100    0        0 eth0

[root@hdss184-244 flannel]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.6.254   0.0.0.0         UG    100    0        0 eth0
172.6.243.0     192.168.6.243   255.255.255.0   UG    0      0        0 eth0
172.6.244.0     0.0.0.0         255.255.255.0   U     0      0        0 docker0
192.168.6.0     0.0.0.0         255.255.255.0   U     100    0        0 eth0
```

Directrouting模型(直接路由)

```shell
'{"Network": "172.6.0.0/16", "Backend": {"Type": "VxLAN","Directrouting": true}}'
```

查找主：

```shell
[root@hdss184-243 etcd]# ./etcdctl member list
4244d625c76d5482: name=etcd-server-6-242 peerURLs=https://192.168.6.242:2380 clientURLs=http://127.0.0.1:2379,https://192.168.6.242:2379 isLeader=true
aa911af67b8285a2: name=etcd-server-6-243 peerURLs=https://192.168.6.243:2380 clientURLs=http://127.0.0.1:2379,https://192.168.6.243:2379 isLeader=false
c751958d48e7e127: name=etcd-server-6-244 peerURLs=https://192.168.6.244:2380 clientURLs=http://127.0.0.1:2379,https://192.168.6.244:2379 isLeader=false
```

### 10.启动服务并检查

```shell
[root@hdss184-243 flannel]# supervisorctl update
flanneld-6-243: added process group

[root@hdss184-243 flannel]# supervisorctl status
etcd-server-6-243                RUNNING   pid 28429, uptime 2 days, 0:10:55
flanneld-6-243                   STARTING  
kube-apiserver-6-243             RUNNING   pid 17808, uptime 18:50:14
kube-controller-manager-6.243    RUNNING   pid 17999, uptime 18:49:47
kube-kubelet-6-243               RUNNING   pid 28717, uptime 1 day, 23:25:50
kube-proxy-6-243                 RUNNING   pid 31546, uptime 1 day, 23:14:58
kube-scheduler-6-243             RUNNING   pid 28574, uptime 1 day, 23:39:57
```

### 11.安装部署启动检查所有集群规划的机器

略

### 12.再次验证集群，pod之间网络互通

```shell
[root@hdss184-243 flannel]# kubectl get pods -o wide
NAME             READY   STATUS    RESTARTS   AGE   IP            NODE                 NOMINATED NODE   READINESS GATES
nginx-ds-jrbdg   1/1     Running   0          9h    172.6.243.2   hdss184-243.host.com   <none>           <none>
nginx-ds-ttlx9   1/1     Running   0          9h    172.6.244.2   hdss184-244.host.com   <none>           <none>

[root@hdss184-243 flannel]# kubectl exec -it nginx-ds-jrbdg bash 
root@nginx-ds-jrbdg:/# ping 172.6.244.2
PING 172.6.244.2 (172.6.244.2): 48 data bytes
56 bytes from 172.6.244.2: icmp_seq=0 ttl=62 time=0.446 ms
56 bytes from 172.6.244.2: icmp_seq=1 ttl=62 time=0.449 ms
56 bytes from 172.6.244.2: icmp_seq=2 ttl=62 time=0.344 ms

[root@hdss184-244 flannel]# kubectl exec -it nginx-ds-ttlx9 bash
root@nginx-ds-ttlx9:/# ping 172.6.243.2
PING 172.6.243.2 (172.6.243.2): 48 data bytes
56 bytes from 172.6.243.2: icmp_seq=0 ttl=62 time=0.324 ms
56 bytes from 172.6.243.2: icmp_seq=1 ttl=62 time=0.286 ms
56 bytes from 172.6.243.2: icmp_seq=2 ttl=62 time=0.345 ms
[root@hdss184-243 flannel]# ping 172.6.244.2
PING 172.6.244.2 (172.6.244.2) 56(84) bytes of data.
64 bytes from 172.6.244.2: icmp_seq=1 ttl=63 time=0.878 ms
64 bytes from 172.6.244.2: icmp_seq=2 ttl=63 time=0.337 ms

[root@hdss184-243 flannel]# ping 172.6.243.2
PING 172.6.243.2 (172.6.243.2) 56(84) bytes of data.
64 bytes from 172.6.243.2: icmp_seq=1 ttl=64 time=0.062 ms
64 bytes from 172.6.243.2: icmp_seq=2 ttl=64 time=0.072 ms
64 bytes from 172.6.243.2: icmp_seq=3 ttl=64 time=0.071 ms


[root@hdss184-244 flannel]# ping 172.6.243.2
PING 172.6.243.2 (172.6.243.2) 56(84) bytes of data.
64 bytes from 172.6.243.2: icmp_seq=1 ttl=63 time=0.248 ms
64 bytes from 172.6.243.2: icmp_seq=2 ttl=63 time=0.125 ms

[root@hdss184-244 flannel]# ping 172.6.244.2
PING 172.6.244.2 (172.6.244.2) 56(84) bytes of data.
64 bytes from 172.6.244.2: icmp_seq=1 ttl=64 time=0.091 ms
64 bytes from 172.6.244.2: icmp_seq=2 ttl=64 time=0.064 ms
```

为甚么172.6.243.2和172.6.244.2容器能通信呢？

```shell
[root@hdss184-244 flannel]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.6.254   0.0.0.0         UG    100    0        0 eth0
172.6.243.0     192.168.6.243   255.255.255.0   UG    0      0        0 eth0
172.6.244.0     0.0.0.0         255.255.255.0   U     0      0        0 docker0
192.168.6.0     0.0.0.0         255.255.255.0   U     100    0        0 eth0


[root@hdss184-243 flannel]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.6.254   0.0.0.0         UG    100    0        0 eth0
172.6.243.0     0.0.0.0         255.255.255.0   U     0      0        0 docker0
172.6.244.0     192.168.6.244   255.255.255.0   UG    0      0        0 eth0
192.168.6.0     0.0.0.0         255.255.255.0   U     100    0        0 eth0
```

本质就是

```shell
[root@hdss184-243 flannel]# route add -net 172.6.244.0/24 gw 192.168.6.244 dev eth0

[root@hdss184-244 flannel]# route add -net 172.6.243.0/24 gw 192.168.6.243 dev eth0
```

### 13.在各个节点上优化iptables规则

为什么要优化？一起看下面的案例：

1.创建nginx-ds.yaml

```shell
[root@hdss184-243 ~]# cat nginx-ds.yaml 
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: nginx-ds
spec:
  template:
    metadata:
      labels:
        app: nginx-ds
    spec:
      containers:
      - name: my-nginx
        image: harbor.od.com/public/nginx:curl
        command: ["nginx","-g","daemon off;"]
        ports:
        - containerPort: 80
```

------

提示：直接用nginx:curl起不来原因是，当时打包nginx:curl 用的是bash：

```shell
docker run --rm -it  sunrisenan/nginx:v1.12.2 bash
```

1、重做nginx:curl
启动nginx，docker run -d –rm nginx:1.7.9
然后exec进去安装curl，再commit

2、仍然用之前做的nginx:curl
然后在k8s的yaml里，加入cmd指令的配置
需要显示的指明，这个镜像的CMD指令是：nginx -g “daemon off;”

```shell
command: ["nginx","-g","daemon off;"]
```

------

2.应用nginx-ds.yaml

```shell
[root@hdss184-243 ~]# kubectl apply -f nginx-ds.yaml
daemonset.extensions/nginx-ds created
```

3.可以发现，pod之间访问nginx显示的是代理的IP地址

```shell
[root@hdss184-243 ~]# kubectl get pods -o wide
NAME             READY   STATUS    RESTARTS   AGE   IP            NODE                 NOMINATED NODE   READINESS GATES
nginx-ds-86f7k   1/1     Running   0          21m   172.6.244.2   hdss184-244.host.com   <none>           <none>
nginx-ds-twfgq   1/1     Running   0          21m   172.6.243.2   hdss184-243.host.com   <none>           <none>

[root@hdss184-243 ~]# kubectl exec -it nginx-ds-86f7k /bin/sh
# curl 172.6.243.2

[root@hdss184-244 ~]# kubectl log -f nginx-ds-twfgq
log is DEPRECATED and will be removed in a future version. Use logs instead.
192.168.6.244 - - [21/Nov/2019:06:56:35 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.38.0" "-"
```

4.优化pod之间不走原地址nat

在所有需要优化的机器上安装iptables

```shell
 ~]# yum install iptables-services -y

 ~]# systemctl start iptables.service
 ~]# systemctl enable iptables.service
```

优化hdss184-243

```shell
[root@hdss184-243 ~]# iptables-save|grep -i postrouting
:POSTROUTING ACCEPT [15:912]
:KUBE-POSTROUTING - [0:0]
-A POSTROUTING -m comment --comment "kubernetes postrouting rules" -j KUBE-POSTROUTING
-A POSTROUTING -s 172.6.243.0/24 ! -o docker0 -j MASQUERADE
-A KUBE-POSTROUTING -m comment --comment "kubernetes service traffic requiring SNAT" -m mark --mark 0x4000/0x4000 -j MASQUERADE

[root@hdss184-243 ~]# iptables -t nat -D POSTROUTING -s 172.6.243.0/24 ! -o docker0 -j MASQUERADE
[root@hdss184-243 ~]# iptables -t nat -I POSTROUTING -s 172.6.243.0/24 ! -d 172.6.0.0/16 ! -o docker0 -j MASQUERADE

[root@hdss184-243 ~]# iptables-save|grep -i postrouting
:POSTROUTING ACCEPT [8:488]
:KUBE-POSTROUTING - [0:0]
-A POSTROUTING -s 172.6.243.0/24 ! -d 172.6.0.0/16 ! -o docker0 -j MASQUERADE
-A POSTROUTING -m comment --comment "kubernetes postrouting rules" -j KUBE-POSTROUTING
-A KUBE-POSTROUTING -m comment --comment "kubernetes service traffic requiring SNAT" -m mark --mark 0x4000/0x4000 -j MASQUERADE
```

优化hdss184-244

```shell
[root@hdss184-244 ~]# iptables-save|grep -i postrouting
:POSTROUTING ACCEPT [7:424]
:KUBE-POSTROUTING - [0:0]
-A POSTROUTING -m comment --comment "kubernetes postrouting rules" -j KUBE-POSTROUTING
-A POSTROUTING -s 172.6.244.0/24 ! -o docker0 -j MASQUERADE
-A KUBE-POSTROUTING -m comment --comment "kubernetes service traffic requiring SNAT" -m mark --mark 0x4000/0x4000 -j MASQUERADE

[root@hdss184-244 ~]# iptables -t nat -D POSTROUTING -s 172.6.244.0/24 ! -o docker0 -j MASQUERADE
[root@hdss184-244 ~]# iptables -t nat -I POSTROUTING -s 172.6.244.0/24 ! -d 172.6.0.0/16 ! -o docker0 -j MASQUERADE

[root@hdss184-244 ~]# iptables-save|grep -i postrouting
:POSTROUTING ACCEPT [2:120]
:KUBE-POSTROUTING - [0:0]
-A POSTROUTING -s 172.6.244.0/24 ! -d 172.6.0.0/16 ! -o docker0 -j MASQUERADE
-A POSTROUTING -m comment --comment "kubernetes postrouting rules" -j KUBE-POSTROUTING
-A KUBE-POSTROUTING -m comment --comment "kubernetes service traffic requiring SNAT" -m mark --mark 0x4000/0x4000 -j MASQUERADE
```

> 192.168.6.243主机上的，来源是172.6.243.0/24段的docker的ip，目标ip不是172.6.0.0/16段，网络发包不从docker0桥设备出站的，才进行转换
>
> 192.168.6.244主机上的，来源是172.6.244.0/24段的docker的ip，目标ip不是172.6.0.0/16段，网络发包不从docker0桥设备出站的，才进行转换

5.把默认禁止规则删掉

```shell
root@hdss184-243 ~]# iptables-save | grep -i reject
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
[root@hdss184-243 ~]# iptables -t filter -D INPUT -j REJECT --reject-with icmp-host-prohibited
[root@hdss184-243 ~]# iptables -t filter -D FORWARD -j REJECT --reject-with icmp-host-prohibited
[root@hdss184-243 ~]# iptables-save > /etc/sysconfig/iptables


[root@hdss184-244 ~]# iptables-save | grep -i reject
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
[root@hdss184-244 ~]# iptables -t filter -D INPUT -j REJECT --reject-with icmp-host-prohibited
[root@hdss184-244 ~]# iptables -t filter -D FORWARD -j REJECT --reject-with icmp-host-prohibited
[root@hdss184-244 ~]# iptables-save > /etc/sysconfig/iptables
```

6.优化SNAT规则，各运算节点之间的各pod之间的网络通信不再出网

测试6.244

```shell
[root@hdss184-243 ~]# kubectl get pods -o wide
NAME             READY   STATUS    RESTARTS   AGE   IP            NODE                 NOMINATED NODE   READINESS GATES
nginx-ds-86f7k   1/1     Running   0          70m   172.6.244.2   hdss184-244.host.com   <none>           <none>
nginx-ds-twfgq   1/1     Running   0          70m   172.6.243.2   hdss184-243.host.com   <none>           <none>

[root@hdss184-243 ~]# curl 172.6.243.2

[root@hdss184-244 ~]# kubectl log -f nginx-ds-86f7k
log is DEPRECATED and will be removed in a future version. Use logs instead.
192.168.6.243 - - [21/Nov/2019:07:56:22 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.29.0" "-"


[root@hdss184-243 ~]# kubectl exec -it  nginx-ds-twfgq /bin/sh
# curl 172.6.244.2

[root@hdss184-244 ~]# kubectl log -f nginx-ds-86f7k
log is DEPRECATED and will be removed in a future version. Use logs instead.
172.6.243.2 - - [21/Nov/2019:07:57:17 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.38.0" "-"
```

测试6.243

```shell
[root@hdss184-244 ~]# kubectl get pods -o wide
NAME             READY   STATUS    RESTARTS   AGE   IP            NODE                 NOMINATED NODE   READINESS GATES
nginx-ds-86f7k   1/1     Running   0          81m   172.6.244.2   hdss184-244.host.com   <none>           <none>
nginx-ds-twfgq   1/1     Running   0          81m   172.6.243.2   hdss184-243.host.com   <none>           <none>

[root@hdss184-244 ~]# curl 172.6.243.2

[root@hdss184-243 ~]# kubectl log -f nginx-ds-twfgq
192.168.6.244 - - [21/Nov/2019:08:01:38 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.29.0" "-"


[root@hdss184-244 ~]# kubectl exec -it nginx-ds-86f7k /bin/sh
# curl 172.6.243.2

[root@hdss184-243 ~]# kubectl log -f nginx-ds-twfgq
172.6.244.2 - - [21/Nov/2019:08:02:52 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.38.0" "-"
```

### 14.在运算节点保存iptables规则

```shell
 [root@hdss184-243 ~]# iptables-save > /etc/sysconfig/iptables
 [root@hdss184-244 ~]# iptables-save > /etc/sysconfig/iptables
 # 上面这中保存重启系统失效，建议用下面的方式

 [root@hdss184-243 ~]# service iptables save
 [root@hdss184-244 ~]# service iptables save
```





# 第三章：kubernetes的服务发现插件–coredns

- 简单来说，服务发现就是服务(应用)之间相互定位的过程。
- 服务发现并非云计算时代独有的，传统的单体架构时代也会用到。以下应用场景下，更需要服务发现
  - 服务(应用)的动态性强
  - 服务(应用)更新发布频繁
  - 服务(应用)支持自动伸缩

- 在K8S集群里，POD的IP是不断变化的，如何“以不变应万变“呢？
  - 抽象出了Service资源，通过标签选择器，关联一组POD
  - 抽象出了集群网络，通过相对固定的“集群IP”，使服务接入点固定

- 那么如何自动关联Service资源的“名称”和“集群网络IP”，从而达到服务被集群自动发现的目的呢？
  - 考虑传统DNS的模型：hdss184-243.host.com –> 192.168.6.243
  - 能否在K8S里建立这样的模型：nginx-ds –> 10.96.0.5
- K8S里服务发现的方式–DNS
- 实现K8S里DNS功能的插件（软件）
  - Kube-dns—kubernetes-v1.2至kubernetes-v1.10
  - Coredns—kubernetes-v1.11至今
- 注意：
  - K8S里的DNS不是万能的！它应该只负责自动维护“服务名”—>“集群网络IP”之间的关系

## 1.部署K8S的内网资源配置清单http服务

> 在运维主机hdss184-245上，配置一个nginx想虚拟主机，用以提供k8s统一资源配置清单访问入口

- 配置nginx

```shell
[root@hdss184-245 ~]# cat /etc/nginx/conf.d/k8s-yaml.od.com.conf
server {
    listen 80;
    server_name k8s-yaml.od.com;

    location / {
        autoindex on;
        default_type text/plain;
        root /data/k8s-yaml;
   }
}

[root@hdss184-245 ~]# mkdir -p /data/k8s-yaml/coredns
[root@hdss184-245 ~]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@hdss184-245 ~]# nginx -s reload
```

- 配置内网DNS解析

在hdss184-241机器上：

```shell
[root@hdss184-241 ~]# vim /var/named/od.com.zone 

k8s-yaml    A    192.168.184.245
```

注意： `2019111204 ; serial` #序列号前滚

```shell
[root@hdss184-241 ~]# named-checkconf 
[root@hdss184-241 ~]# systemctl restart named
[root@hdss184-241 ~]# dig -t A k8s-yaml.od.com @192.168.184.241 +short
192.168.6.245
```

以后多有的资源配置清单统一放置在运维主机上的`/data/k8s-yaml`目录即可

## 2.部署coredns

[coredns官方Guthub](https://github.com/coredns/coredns)

[coredns官方DockerHub](https://hub.docker.com/r/coredns/coredns/tags)

### 1.准备coredns-v1.6.1镜像

在运维主机hdss184-245上：

```shell
[root@hdss184-245 ~]# docker pull coredns/coredns:1.6.1
1.6.1: Pulling from coredns/coredns
c6568d217a00: Pull complete 
d7ef34146932: Pull complete 
Digest: sha256:9ae3b6fcac4ee821362277de6bd8fd2236fa7d3e19af2ef0406d80b595620a7a
Status: Downloaded newer image for coredns/coredns:1.6.1
docker.io/coredns/coredns:1.6.1

[root@hdss184-245 ~]# docker images|grep coredns
coredns/coredns                 1.6.1                      c0f6e815079e        3 months ago        42.2MB

[root@hdss184-245 ~]# docker tag c0f6e815079e harbor.od.com/public/coredns:v1.6.1
[root@hdss184-245 ~]# docker push !$
docker push harbor.od.com/public/coredns:v1.6.1
The push refers to repository [harbor.od.com/public/coredns]
da1ec456edc8: Pushed 
225df95e717c: Pushed 
v1.6.1: digest: sha256:c7bf0ce4123212c87db74050d4cbab77d8f7e0b49c041e894a35ef15827cf938 size: 739
```

### 2.准备资源配置清单

在运维主机hdss184-245.host.com上：

```shell
[root@hdss184-245 ~]# mkdir -p /data/k8s-yaml/coredns && cd /data/k8s-yaml/coredns/
```

RBAC

```shell
[root@hdss184-245 coredns]# cat rbac.yaml 
apiVersion: v1
kind: ServiceAccount
metadata:
  name: coredns
  namespace: kube-system
  labels:
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
    addonmanager.kubernetes.io/mode: Reconcile
  name: system:coredns
rules:
- apiGroups:
  - ""
  resources:
  - endpoints
  - services
  - pods
  - namespaces
  verbs:
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
    addonmanager.kubernetes.io/mode: EnsureExists
  name: system:coredns
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:coredns
subjects:
- kind: ServiceAccount
  name: coredns
  namespace: kube-system
```

ConfigMap

```shell
[root@hdss184-245 coredns]# cat cm.yaml 
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        log
        health
        ready
        kubernetes cluster.local 10.96.0.0/22
        forward . 192.168.184.241
        cache 30
        loop
        reload
        loadbalance
       }
```

Deployment

```shell
[root@hdss184-245 coredns]# cat dp.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: coredns
  namespace: kube-system
  labels:
    k8s-app: coredns
    kubernetes.io/name: "CoreDNS"
spec:
  replicas: 1
  selector:
    matchLabels:
      k8s-app: coredns
  template:
    metadata:
      labels:
        k8s-app: coredns
    spec:
      priorityClassName: system-cluster-critical
      serviceAccountName: coredns
      containers:
      - name: coredns
        image: harbor.od.com/public/coredns:v1.6.1
        args:
        - -conf
        - /etc/coredns/Corefile
        volumeMounts:
        - name: config-volume
          mountPath: /etc/coredns
        ports:
        - containerPort: 53
          name: dns
          protocol: UDP
        - containerPort: 53
          name: dns-tcp
          protocol: TCP
        - containerPort: 9153
          name: metrics
          protocol: TCP
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 60
          timeoutSeconds: 5
          successThreshold: 1
          failureThreshold: 5
      dnsPolicy: Default
      volumes:
        - name: config-volume
          configMap:
            name: coredns
            items:
            - key: Corefile
              path: Corefile
```

Service

```shell
[root@hdss184-245 coredns]# cat svc.yaml 
apiVersion: v1
kind: Service
metadata:
  name: coredns
  namespace: kube-system
  labels:
    k8s-app: coredns
    kubernetes.io/cluster-service: "true"
    kubernetes.io/name: "CoreDNS"
spec:
  selector:
    k8s-app: coredns
  clusterIP: 10.96.0.2
  ports:
  - name: dns
    port: 53
    protocol: UDP
  - name: dns-tcp
    port: 53
  - name: metrics
    port: 9153
    protocol: TCP
```

### 3.依次执行创建

```shell
[root@hdss184-243 ~]# kubectl apply -f http://k8s-yaml.od.com/coredns/rbac.yaml
[root@hdss184-243 ~]# kubectl apply -f http://k8s-yaml.od.com/coredns/cm.yaml
[root@hdss184-243 ~]# kubectl apply -f http://k8s-yaml.od.com/coredns/dp.yaml
[root@hdss184-243 ~]# kubectl apply -f http://k8s-yaml.od.com/coredns/svc.yaml
```

### 4.检查

```shell
[root@hdss184-243 ~]# kubectl get all -n kube-system
NAME                           READY   STATUS    RESTARTS   AGE
pod/coredns-6b6c4f9648-x5zvz   1/1     Running   0          7m37s


NAME              TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
service/coredns   ClusterIP   10.96.0.2    <none>        53/UDP,53/TCP,9153/TCP   7m27s


NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/coredns   1/1     1            1           7m37s

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/coredns-6b6c4f9648   1         1         1       7m37s
[root@hdss184-243 ~]# dig -t A www.baidu.com @10.96.0.2 +short
www.a.shifen.com.
180.101.49.12
180.101.49.11

[root@hdss184-243 ~]# dig -t A hdss184-245.host.com @10.96.0.2 +short
192.168.6.245
[root@hdss184-243 ~]# kubectl create deployment nginx-dp --image=harbor.od.com/public/nginx:v1.7.9 -n kube-public
deployment.apps/nginx-dp created

[root@hdss184-243 ~]# kubectl expose deployment nginx-dp --port=80 -n kube-public
service/nginx-dp exposed

[root@hdss184-243 ~]# kubectl get service -n kube-public
NAME       TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
nginx-dp   ClusterIP   10.96.3.154   <none>        80/TCP    38s
[root@hdss184-243 ~]# dig -t A nginx-dp.kube-public.svc.cluster.local. @10.96.0.2 +short
10.96.3.154
```



# 第四章：kubernetes的服务暴露插件–traefik

- K8S的DNS实现了服务在集群“内”被自动发现，那如何是的服务在K8S集群“外”被使用和访问呢？
  - 使用NodePort型的Service
    - 注意：无法使用kube-proxy的ipvs模型，只能使用iptables模型
  - 使用Ingress资源
    - 注意：Ingress只能调度并暴露7层应用，特指http和https协议
- Ingress是K8S API的标准资源类型之一，也是一种核心资源，它其实就是一组基于域名和URL路径，把用户的请求转发至指定Service资源的规则
- 可以将集群外部的请求流量，转发至集群内部，从而实现“服务暴露”
- Ingress控制器是能够为Ingress资源监听某套接字，然后根据Ingress规则匹配机制路由调度流量的一个插件
- 说白了，Ingress没啥神秘的，就是个nginx+一段go脚本而已
- 常用的Ingress控制器的实现软件
  - Ingress-nginx
  - HAProxy
  - Traefik
  - …

## 1.使用NodePort型Service暴露服务

注意：使用这种方法暴露服务，要求kube-proxy的代理类型为：iptables

```shell
1、第一步更改为proxy-mode更改为iptables，调度方式为RR
[root@hdss184-243 ~]# cat /opt/kubernetes/server/bin/kube-proxy.sh
#!/bin/sh
./kube-proxy \
  --cluster-cidr 172.6.0.0/16 \
  --hostname-override hdss184-243.host.com \
  --proxy-mode=iptables \
  --ipvs-scheduler=rr \
  --kubeconfig ./conf/kube-proxy.kubeconfig

[root@hdss184-244 ~]# cat /opt/kubernetes/server/bin/kube-proxy.sh
#!/bin/sh
./kube-proxy \
  --cluster-cidr 172.6.0.0/16 \
  --hostname-override hdss184-243.host.com \
  --proxy-mode=iptables \
  --ipvs-scheduler=rr \
  --kubeconfig ./conf/kube-proxy.kubeconfig


2.使iptables模式生效
[root@hdss184-243 ~]# supervisorctl restart kube-proxy-184-243
kube-proxy-6-243: stopped
kube-proxy-6-243: started
[root@hdss184-243 ~]# ps -ef|grep kube-proxy
root     26694 12008  0 10:25 ?        00:00:00 /bin/sh /opt/kubernetes/server/bin/kube-proxy.sh
root     26695 26694  0 10:25 ?        00:00:00 ./kube-proxy --cluster-cidr 172.6.0.0/16 --hostname-override hdss184-243.host.com --proxy-mode=iptables --ipvs-scheduler=rr --kubeconfig ./conf/kube-proxy.kubeconfig
root     26905 13466  0 10:26 pts/0    00:00:00 grep --color=auto kube-proxy


[root@hdss184-244 ~]# supervisorctl restart kube-proxy-184-244
kube-proxy-6-244: stopped
kube-proxy-6-244kube-proxy-6-244: started
[root@hdss184-244 ~]# ps -ef|grep kube-proxy
root     25998 11030  0 10:22 ?        00:00:00 /bin/sh /opt/kubernetes/server/bin/kube-proxy.sh
root     25999 25998  0 10:22 ?        00:00:00 ./kube-proxy --cluster-cidr 172.6.0.0/16 --hostname-override hdss184-243.host.com --proxy-mode=iptables --ipvs-scheduler=rr --kubeconfig ./conf/kube-proxy.kubeconfig


[root@hdss184-243 ~]# tail -fn 11 /data/logs/kubernetes/kube-proxy/proxy.stdout.log 

[root@hdss184-244 ~]# tail -fn 11 /data/logs/kubernetes/kube-proxy/proxy.stdout.log


3.清理现有的ipvs规则
[root@hdss184-243 ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.96.0.1:443 nq
  -> 192.168.6.243:6443           Masq    1      0          0         
  -> 192.168.6.244:6443           Masq    1      0          0         
TCP  10.96.0.2:53 nq
  -> 172.6.244.3:53               Masq    1      0          0         
TCP  10.96.0.2:9153 nq
  -> 172.6.244.3:9153             Masq    1      0          0         
TCP  10.96.3.154:80 nq
  -> 172.6.243.3:80               Masq    1      0          0         
UDP  10.96.0.2:53 nq
  -> 172.6.244.3:53               Masq    1      0          0         
[root@hdss184-243 ~]# ipvsadm -D -t 10.96.0.1:443
[root@hdss184-243 ~]# ipvsadm -D -t 10.96.0.2:53
[root@hdss184-243 ~]# ipvsadm -D -t 10.96.0.2:9153
[root@hdss184-243 ~]# ipvsadm -D -t 10.96.3.154:80
[root@hdss184-243 ~]# ipvsadm -D -u 10.96.0.2:53
[root@hdss184-243 ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn


[root@hdss184-244 ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.96.0.1:443 nq
  -> 192.168.6.243:6443           Masq    1      0          0         
  -> 192.168.6.244:6443           Masq    1      1          0         
TCP  10.96.0.2:53 nq
  -> 172.6.244.3:53               Masq    1      0          0         
TCP  10.96.0.2:9153 nq
  -> 172.6.244.3:9153             Masq    1      0          0         
TCP  10.96.3.154:80 nq
  -> 172.6.243.3:80               Masq    1      0          0         
UDP  10.96.0.2:53 nq
  -> 172.6.244.3:53               Masq    1      0          0         
[root@hdss184-244 ~]# ipvsadm -D -t 10.96.0.1:443
[root@hdss184-244 ~]# ipvsadm -D -t 10.96.0.2:53
[root@hdss184-244 ~]# ipvsadm -D -t 10.96.0.2:9153
[root@hdss184-244 ~]# ipvsadm -D -t 10.96.3.154:80
[root@hdss184-244 ~]# ipvsadm -D -u 10.96.0.2:53
[root@hdss184-244 ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
```

### 1.修改nginx-ds的service资源配置清单

```shell
[root@hdss184-243 ~]# cat /root/nginx-ds-svc.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx-ds
  name: nginx-ds
  namespace: default
spec:
  ports:
  - port: 80
    protocol: TCP
    nodePort: 8000
  selector:
    app: nginx-ds
  sessionAffinity: None
  type: NodePort
[root@hdss184-243 ~]# kubectl apply -f nginx-ds-svc.yaml 
service/nginx-ds created
```

### 2.重建nginx-ds的service资源

```shell
[root@hdss184-243 ~]# cat nginx-ds.yaml 
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: nginx-ds
spec:
  template:
    metadata:
      labels:
        app: nginx-ds
    spec:
      containers:
      - name: my-nginx
        image: harbor.od.com/public/nginx:curl
        command: ["nginx","-g","daemon off;"]
        ports:
        - containerPort: 80
[root@hdss184-243 ~]# kubectl apply -f nginx-ds.yaml 
daemonset.extensions/nginx-ds created
```

### 3.查看service

```shell
[root@hdss184-243 ~]# kubectl get svc nginx-ds
NAME       TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)       AGE
nginx-ds   NodePort   10.96.1.170   <none>        80:8000/TCP   2m20s

[root@hdss184-243 ~]# netstat -lntup|grep 8000
tcp6       0      0 :::8000                 :::*                    LISTEN      26695/./kube-proxy

[root@hdss184-244 ~]# netstat -lntup|grep 8000
tcp6       0      0 :::8000                 :::*                    LISTEN      25999/./kube-proxy 
```

- nodePort本质

```shell
[root@hdss184-244 ~]# iptables-save | grep 8000
-A KUBE-FIREWALL -m comment --comment "kubernetes firewall for dropping marked packets" -m mark --mark 0x8000/0x8000 -j DROP
-A KUBE-MARK-DROP -j MARK --set-xmark 0x8000/0x8000
-A KUBE-NODEPORTS -p tcp -m comment --comment "default/nginx-ds:" -m tcp --dport 8000 -j KUBE-MARK-MASQ
-A KUBE-NODEPORTS -p tcp -m comment --comment "default/nginx-ds:" -m tcp --dport 8000 -j KUBE-SVC-T4RQBNWQFKKBCRET
```

### 4.浏览器访问

访问：http://192.168.184.243:8000/

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_219520c675b6c8faf33df7c9ebaa07bf_r.png)

访问：http://192.168.184.244:8000/

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_7103dd03286a23073e5055cd87a272c1_r.png)

### 5.还原成ipvs

删除service和pod

```shell
[root@hdss184-243 ~]# kubectl delete -f nginx-ds.yaml 
daemonset.extensions "nginx-ds" deleted

[root@hdss184-243 ~]# kubectl delete -f nginx-ds-svc.yaml 
service "nginx-ds" deleted

[root@hdss184-243 ~]# netstat -lntup|grep 8000
[root@hdss184-243 ~]# 
```

在运算节点上：

```shell
[root@hdss184-243 ~]# cat /opt/kubernetes/server/bin/kube-proxy.sh
#!/bin/sh
./kube-proxy \
  --cluster-cidr 172.6.0.0/16 \
  --hostname-override hdss184-243.host.com \
  --proxy-mode=ipvs \
  --ipvs-scheduler=nq \
  --kubeconfig ./conf/kube-proxy.kubeconfig

[root@hdss184-243 ~]# supervisorctl restart kube-proxy-184-243
kube-proxy-6-243: stopped
kube-proxy-6-243: started

[root@hdss184-243 ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.96.0.1:443 nq
  -> 192.168.6.243:6443           Masq    1      0          0         
  -> 192.168.6.244:6443           Masq    1      0          0         
TCP  10.96.0.2:53 nq
  -> 172.6.244.3:53               Masq    1      0          0         
TCP  10.96.0.2:9153 nq
  -> 172.6.244.3:9153             Masq    1      0          0         
TCP  10.96.3.154:80 nq
  -> 172.6.243.3:80               Masq    1      0          0         
UDP  10.96.0.2:53 nq
  -> 172.6.244.3:53               Masq    1      0          0   
[root@hdss184-244 ~]# cat /opt/kubernetes/server/bin/kube-proxy.sh
#!/bin/sh
./kube-proxy \
  --cluster-cidr 172.6.0.0/16 \
  --hostname-override hdss184-243.host.com \
  --proxy-mode=ipvs \
  --ipvs-scheduler=nq \
  --kubeconfig ./conf/kube-proxy.kubeconfig

[root@hdss184-244 ~]# supervisorctl restart kube-proxy-184-244
kube-proxy-6-244: stopped
kube-proxy-6-244: started

[root@hdss184-244 ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.96.0.1:443 nq
  -> 192.168.184.243:6443           Masq    1      0          0         
  -> 192.168.184.244:6443           Masq    1      0          0         
TCP  10.96.0.2:53 nq
  -> 172.6.244.3:53               Masq    1      0          0         
```

## 2.部署traefik（ingress控制器）

[traefik官方GitHub](https://github.com/containous/traefik)

[traefik官方DockerHub](https://hub.docker.com/_/traefik)

### 1.准备traefik镜像

在运维主机上hdss184-245.host.com上：

```shell
[root@hdss184-245 traefik]# docker pull traefik:v1.7.2-alpine
v1.7.2-alpine: Pulling from library/traefik
4fe2ade4980c: Pull complete 
8d9593d002f4: Pull complete 
5d09ab10efbd: Pull complete 
37b796c58adc: Pull complete 
Digest: sha256:cf30141936f73599e1a46355592d08c88d74bd291f05104fe11a8bcce447c044
Status: Downloaded newer image for traefik:v1.7.2-alpine
docker.io/library/traefik:v1.7.2-alpine

[root@hdss184-245 traefik]# docker images|grep traefik
traefik                         v1.7.2-alpine              add5fac61ae5        13 months ago       72.4MB

[root@hdss184-245 traefik]# docker tag add5fac61ae5 harbor.od.com/public/traefik:v1.7.2
[root@hdss184-245 traefik]# docker push !$
docker push harbor.od.com/public/traefik:v1.7.2
The push refers to repository [harbor.od.com/public/traefik]
a02beb48577f: Pushed 
ca22117205f4: Pushed 
3563c211d861: Pushed 
df64d3292fd6: Pushed 
v1.7.2: digest: sha256:6115155b261707b642341b065cd3fac2b546559ba035d0262650b3b3bbdd10ea size: 1157
```

### 2.准备资源配置清单

```shell
[root@hdss184-245 traefik]# cat /data/k8s-yaml/traefik/rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: traefik-ingress-controller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: traefik-ingress-controller
rules:
  - apiGroups:
      - ""
    resources:
      - services
      - endpoints
      - secrets
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - extensions
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: traefik-ingress-controller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: traefik-ingress-controller
subjects:
- kind: ServiceAccount
  name: traefik-ingress-controller
  namespace: kube-system
[root@hdss184-245 traefik]# cat /data/k8s-yaml/traefik/ds.yaml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: traefik-ingress
  namespace: kube-system
  labels:
    k8s-app: traefik-ingress
spec:
  template:
    metadata:
      labels:
        k8s-app: traefik-ingress
        name: traefik-ingress
    spec:
      serviceAccountName: traefik-ingress-controller
      terminationGracePeriodSeconds: 60
      containers:
      - image: harbor.od.com/public/traefik:v1.7.2
        name: traefik-ingress
        ports:
        - name: controller
          containerPort: 80
          hostPort: 81
        - name: admin-web
          containerPort: 8080
        securityContext:
          capabilities:
            drop:
            - ALL
            add:
            - NET_BIND_SERVICE
        args:
        - --api
        - --kubernetes
        - --logLevel=INFO
        - --insecureskipverify=true
        - --kubernetes.endpoint=https://192.168.184.66:7443
        - --accesslog
        - --accesslog.filepath=/var/log/traefik_access.log
        - --traefiklog
        - --traefiklog.filepath=/var/log/traefik.log
        - --metrics.prometheus
[root@hdss184-245 traefik]# cat /data/k8s-yaml/traefik/svc.yaml
kind: Service
apiVersion: v1
metadata:
  name: traefik-ingress-service
  namespace: kube-system
spec:
  selector:
    k8s-app: traefik-ingress
  ports:
    - protocol: TCP
      port: 80
      name: controller
    - protocol: TCP
      port: 8080
      name: admin-web
[root@hdss184-245 traefik]# cat /data/k8s-yaml/traefik/ingress.yaml 
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: traefik-web-ui
  namespace: kube-system
  annotations:
    kubernetes.io/ingress.class: traefik
spec:
  rules:
  - host: traefik.od.com
    http:
      paths:
      - path: /
        backend:
          serviceName: traefik-ingress-service
          servicePort: 8080
```

### 3.依次执行创建

```shell
[root@hdss184-243 ~]# kubectl apply -f http://k8s-yaml.od.com/traefik/rbac.yaml
[root@hdss184-243 ~]# kubectl apply -f http://k8s-yaml.od.com/traefik/ds.yaml
[root@hdss184-243 ~]# kubectl apply -f http://k8s-yaml.od.com/traefik/svc.yaml
[root@hdss184-243 ~]# kubectl apply -f http://k8s-yaml.od.com/traefik/ingress.yaml
```

## 3.解析域名

```shell
[root@hdss184-241 ~]# cat /var/named/od.com.zone 
$ORIGIN od.com.
$TTL 600    ; 10 minutes
@           IN SOA    dns.od.com. dnsadmin.od.com. (
                2019111205 ; serial
                10800      ; refresh (3 hours)
                900        ; retry (15 minutes)
                604800     ; expire (1 week)
                86400      ; minimum (1 day)
                )
                NS   dns.od.com.
$TTL 60    ; 1 minute
dns                A    192.168.184.241
harbor             A    192.168.184.245
k8s-yaml           A    192.168.184.245
traefik            A    192.168.184.66

[root@hdss184-241 ~]# named-checkconf 
[root@hdss184-241 ~]# systemctl restart named
```

## 4.配置反向代理

```shell
[root@hdss184-241 ~]# cat /etc/nginx/conf.d/od.com.conf
upstream default_backend_traefik {
    server 192.168.184.243:81    max_fails=3 fail_timeout=10s;
    server 192.168.184.244:81    max_fails=3 fail_timeout=10s;
}
server {
    server_name *.od.com;

    location / {
        proxy_pass http://default_backend_traefik;
        proxy_set_header Host       $http_host;
        proxy_set_header x-forwarded-for $proxy_add_x_forwarded_for;
    }
}
[root@hdss184-241 ~]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@hdss184-241 ~]# nginx -s reload
[root@hdss184-242 ~]# vim /etc/nginx/conf.d/od.com.conf
[root@hdss184-242 ~]# cat /etc/nginx/conf.d/od.com.conf
upstream default_backend_traefik {
    server 192.168.184.243:81    max_fails=3 fail_timeout=10s;
    server 192.168.184.244:81    max_fails=3 fail_timeout=10s;
}
server {
    server_name *.od.com;

    location / {
        proxy_pass http://default_backend_traefik;
        proxy_set_header Host       $http_host;
        proxy_set_header x-forwarded-for $proxy_add_x_forwarded_for;
    }
}
[root@hdss184-242 ~]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@hdss184-242 ~]# nginx -s reload
```

## 5.检查

```shell
[root@hdss184-244 ~]# kubectl get all -n kube-system 
NAME                           READY   STATUS    RESTARTS   AGE
pod/coredns-6b6c4f9648-x5zvz   1/1     Running   0          18h
pod/traefik-ingress-bhhkv      1/1     Running   0          17m
pod/traefik-ingress-mm2ds      1/1     Running   0          17m


NAME                              TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                  AGE
service/coredns                   ClusterIP   10.96.0.2     <none>        53/UDP,53/TCP,9153/TCP   18h
service/traefik-ingress-service   ClusterIP   10.96.3.175   <none>        80/TCP,8080/TCP          17m

NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/traefik-ingress   2         2         2       2            2           <none>          17m

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/coredns   1/1     1            1           18h

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/coredns-6b6c4f9648   1         1         1       18h
```

------

如果pod没有起来没有起来请重启docker，原因是我上面测试了nodeport，防火墙规则改变了

```shell
[root@hdss184-243 ~]# systemctl restart docker
[root@hdss184-244 ~]# systemctl restart docker
```

------

## 6.浏览器访问

**访问 http://traefik.od.com/**
![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_81f6fbe8147a8cbae19f37dcaacc63b9_r.png)





# 第五章：K8S的GUI资源管理插件-仪表篇

## 1.部署kubenetes-dashborad

[dashboard官方Github](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/dashboard)

[dashboard下载地址](https://github.com/kubernetes/dashboard/releases)

### 1.准备dashboard镜像

运维主机hdss184-245.host.com上：

```shell
[root@hdss184-245 ~]# docker pull sunrisenan/kubernetes-dashboard-amd64:v1.10.1

[root@hdss184-245 ~]# docker pull sunrisenan/kubernetes-dashboard-amd64:v1.8.3

[root@hdss184-245 ~]# docker images |grep dash
sunrisenan/kubernetes-dashboard-amd64   v1.10.1                    f9aed6605b81        11 months ago       122MB
sunrisenan/kubernetes-dashboard-amd64   v1.8.3                     0c60bcf89900        21 months ago       102MB

[root@hdss184-245 ~]# docker tag f9aed6605b81  harbor.od.com/public/kubernetes-dashboard-amd64:v1.10.1
[root@hdss184-245 ~]# docker push !$

[root@hdss184-245 ~]# docker tag 0c60bcf89900 harbor.od.com/public/kubernetes-dashboard-amd64:v1.8.3
[root@hdss184-245 ~]# docker push !$
```

### 2.准备配置清单

运维主机hdss184-245.host.com上：

- 创建目录

  ```shell
  [root@hdss184-245 ~]# mkdir -p /data/k8s-yaml/dashboard && cd /data/k8s-yaml/dashboard
  ```

- rbac

```shell
[root@hdss184-245 dashboard]# cat rbac.yaml 
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: kubernetes-dashboard
    addonmanager.kubernetes.io/mode: Reconcile
  name: kubernetes-dashboard-admin
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard-admin
  namespace: kube-system
  labels:
    k8s-app: kubernetes-dashboard
    addonmanager.kubernetes.io/mode: Reconcile
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard-admin
  namespace: kube-system
```

- Deployment

```shell
[root@hdss184-245 dashboard]# cat dp.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubernetes-dashboard
  namespace: kube-system
  labels:
    k8s-app: kubernetes-dashboard
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
spec:
  selector:
    matchLabels:
      k8s-app: kubernetes-dashboard
  template:
    metadata:
      labels:
        k8s-app: kubernetes-dashboard
      annotations:
        scheduler.alpha.kubernetes.io/critical-pod: ''
    spec:
      priorityClassName: system-cluster-critical
      containers:
      - name: kubernetes-dashboard
        image: harbor.od.com/public/kubernetes-dashboard-amd64:v1.8.3
        resources:
          limits:
            cpu: 100m
            memory: 300Mi
          requests:
            cpu: 50m
            memory: 100Mi
        ports:
        - containerPort: 8443
          protocol: TCP
        args:
          # PLATFORM-SPECIFIC ARGS HERE
          - --auto-generate-certificates
        volumeMounts:
        - name: tmp-volume
          mountPath: /tmp
        livenessProbe:
          httpGet:
            scheme: HTTPS
            path: /
            port: 8443
          initialDelaySeconds: 30
          timeoutSeconds: 30
      volumes:
      - name: tmp-volume
        emptyDir: {}
      serviceAccountName: kubernetes-dashboard-admin
      tolerations:
      - key: "CriticalAddonsOnly"
        operator: "Exists"
```

- Service

```shell
[root@hdss184-245 dashboard]# cat svc.yaml 
apiVersion: v1
kind: Service
metadata:
  name: kubernetes-dashboard
  namespace: kube-system
  labels:
    k8s-app: kubernetes-dashboard
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
spec:
  selector:
    k8s-app: kubernetes-dashboard
  ports:
  - port: 443
    targetPort: 8443
```

- ingress

```shell
[root@hdss184-245 dashboard]# cat ingress.yaml 
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: kubernetes-dashboard
  namespace: kube-system
  annotations:
    kubernetes.io/ingress.class: traefik
spec:
  rules:
  - host: dashboard.od.com
    http:
      paths:
      - backend:
          serviceName: kubernetes-dashboard
          servicePort: 443
```

### 3.依次执行创建

浏览器打开：`http://k8s-yaml.od.com/dashboard/`检查资源配置清单文件是否正确创建

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_738286ba6d1370be5fa09d275f5ac09d_r.png)

在hdss184-243.host.com机器上：

```shell
[root@hdss184-243 ~]# kubectl apply -f http://k8s-yaml.od.com/dashboard/rbac.yaml
[root@hdss184-243 ~]# kubectl apply -f http://k8s-yaml.od.com/dashboard/dp.yaml
[root@hdss184-243 ~]# kubectl apply -f http://k8s-yaml.od.com/dashboard/svc.yaml
[root@hdss184-243 ~]# kubectl apply -f http://k8s-yaml.od.com/dashboard/ingress.yaml
```

## 2.解析域名

- 添加解析记录

```shell
[root@hdss184-241 ~]# cat /var/named/od.com.zone
$ORIGIN od.com.
$TTL 600    ; 10 minutes
@           IN SOA    dns.od.com. dnsadmin.od.com. (
                2019111208 ; serial    # 向后滚动+1
                10800      ; refresh (3 hours)
                900        ; retry (15 minutes)
                604800     ; expire (1 week)
                86400      ; minimum (1 day)
                )
                NS   dns.od.com.
$TTL 60    ; 1 minute
dns                A    192.168.6.241
harbor             A    192.168.6.245
k8s-yaml           A    192.168.6.245
traefik            A    192.168.6.66
dashboard          A    192.168.6.66   # 添加这条解析
```

- 重启named并检查

```shell
[root@hdss184-241 ~]# systemctl restart named

[root@hdss184-243 ~]# dig dashboard.od.com @10.96.0.2 +short
192.168.6.66
[root@hdss184-243 ~]# dig dashboard.od.com @192.168.184.241 +short
192.168.6.66
```

## 3.浏览器访问

浏览器访问：`http://dashboard.od.com`

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_56d7bba51541a6e914564efc19910251_r.png)

## 4.配置认证

### 1.签发证书

```shell
[root@hdss184-245 certs]# (umask 077; openssl genrsa -out dashboard.od.com.key 2048)
Generating RSA private key, 2048 bit long modulus
............................+++
........+++
e is 65537 (0x10001)
[root@hdss184-245 certs]# openssl req -new -key dashboard.od.com.key -out dashboard.od.com.csr -subj "/CN=dashboard.od.com/C=CN/ST=BJ/L=Beijing/O=OldboyEdu/OU=ops"
[root@hdss184-245 certs]# openssl x509 -req -in dashboard.od.com.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out dashboard.od.com.crt -days 3650
Signature ok
subject=/CN=dashboard.od.com/C=CN/ST=BJ/L=Beijing/O=OldboyEdu/OU=ops
Getting CA Private Key

[root@hdss184-245 certs]# ll dash*
-rw-r--r-- 1 root root 1196 Nov 27 12:52 dashboard.od.com.crt
-rw-r--r-- 1 root root 1005 Nov 27 12:52 dashboard.od.com.csr
-rw------- 1 root root 1679 Nov 27 12:52 dashboard.od.com.key
```

### 2.检查证书

```shell
[root@hdss184-245 certs]# cfssl-certinfo -cert dashboard.od.com.crt 
{
  "subject": {
    "common_name": "dashboard.od.com",
    "country": "CN",
    "organization": "OldboyEdu",
    "organizational_unit": "ops",
    "locality": "Beijing",
    "province": "BJ",
    "names": [
      "dashboard.od.com",
      "CN",
      "BJ",
      "Beijing",
      "OldboyEdu",
      "ops"
    ]
  },
  "issuer": {
    "common_name": "OldboyEdu",
    "country": "CN",
    "organization": "od",
    "organizational_unit": "ops",
    "locality": "beijing",
    "province": "beijing",
    "names": [
      "CN",
      "beijing",
      "beijing",
      "od",
      "ops",
      "OldboyEdu"
    ]
  },
  "serial_number": "11427294234507397728",
  "not_before": "2019-11-27T04:52:30Z",
  "not_after": "2029-11-24T04:52:30Z",
  "sigalg": "SHA256WithRSA",
  "authority_key_id": "",
  "subject_key_id": "",
  "pem": "-----BEGIN CERTIFICATE-----\nMIIDRTCCAi0CCQCeleeP167KYDANBgkqhkiG9w0BAQsFADBgMQswCQYDVQQGEwJD\nTjEQMA4GA1UECBMHYmVpamluZzEQMA4GA1UEBxMHYmVpamluZzELMAkGA1UEChMC\nb2QxDDAKBgNVBAsTA29wczESMBAGA1UEAxMJT2xkYm95RWR1MB4XDTE5MTEyNzA0\nNTIzMFoXDTI5MTEyNDA0NTIzMFowaTEZMBcGA1UEAwwQZGFzaGJvYXJkLm9kLmNv\nbTELMAkGA1UEBhMCQ04xCzAJBgNVBAgMAkJKMRAwDgYDVQQHDAdCZWlqaW5nMRIw\nEAYDVQQKDAlPbGRib3lFZHUxDDAKBgNVBAsMA29wczCCASIwDQYJKoZIhvcNAQEB\nBQADggEPADCCAQoCggEBALeeL9z8V3ysUqrAuT7lEKcF2bi0pSuwoWfFgfBtGmQa\nQtyNaOlyemEexeUOKaIRsNlw0fgcK6HyyLkaMFsVa7q+bpYBPKp4d7lTGU7mKJNG\nNcCU21G8WZYS4jVtd5IYSmmfNkCnzY7l71p1P+sAZNZ7ht3ocNh6jPcHLMpETLUU\nDaKHmT/5iAhxmgcE/V3DUnTawU9PXF2WnICL9xJtmyErBKF5KDIEjC1GVjC/ZLtT\nvEgbH57TYgrp4PeCEAQTtgNbVJuri4awaLpHkNz2iCTNlWpLaLmV1jT1NtChz6iw\n4lDfEgS6YgDh9ZhlB2YvnFSG2eq4tGm3MKorbuMq9S0CAwEAATANBgkqhkiG9w0B\nAQsFAAOCAQEAG6szkJDIvb0ge2R61eMBVe+CWHHSE6X4EOkiQCaCi3cs8h85ES63\nEdQ8/FZorqyZH6nJ/64OjiW1IosXRTFDGMRunqkJezzj9grYzUKfkdGxTv+55IxM\ngtH3P9wM1EeNwdJCpBq9xYaPzZdu0mzmd47BP7nuyrXzkMSecC/d+vrKngEnUXaZ\n9WK3eGnrGPmeW7z5j9uVsimzNlri8i8FNBTGCDx2sgJc16MtYfGhORwN4oVXCHiS\n4A/HVSYMUeR4kGxoX9RUbf8vylRsdEbKQ20M5cbWQAAH5LNig6jERRsuylEh4uJE\nubhEbfhePgZv+mkFQ6tsuIH/5ETSV4v/bg==\n-----END CERTIFICATE-----\n"
}
```

### 3.配置nginx

在hdss184-241和hdss184-242上：

- 拷贝证书

```shell
~]# mkdir /etc/nginx/conf.d/certs

~]# scp hdss184-245:/opt/certs/dashboard.od.com.crt /etc/nginx/certs/
~]# scp hdss184-245:/opt/certs/dashboard.od.com.key /etc/nginx/certs/
```

- 配置虚拟主机dashboard.od.com.conf，走https

```shell
[root@hdss184-241 ~]# cat /etc/nginx/conf.d/dashboard.od.com.conf
server {
    listen       80;
    server_name  dashboard.od.com;

    rewrite ^(.*)$ https://${server_name}$1 permanent;
}
server {
    listen       443 ssl;
    server_name  dashboard.od.com;

    ssl_certificate "certs/dashboard.od.com.crt";
    ssl_certificate_key "certs/dashboard.od.com.key";
    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout  10m;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    location / {
        proxy_pass http://default_backend_traefik;
        proxy_set_header Host       $http_host;
        proxy_set_header x-forwarded-for $proxy_add_x_forwarded_for;
    }
}
```

- 重载nginx配置

```shell
 ~]# nginx -s reload
```

- 刷新页面检查

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_cc2c885819a8a0b49994178e10d055ba_r.png)

### 4.获取kubernetes-dashboard-admin-token

```shell
[root@hdss184-243 ~]# kubectl get secrets -n kube-system 
NAME                                     TYPE                                  DATA   AGE
coredns-token-r5s8r                      kubernetes.io/service-account-token   3      5d19h
default-token-689cg                      kubernetes.io/service-account-token   3      6d14h
kubernetes-dashboard-admin-token-w46s2   kubernetes.io/service-account-token   3      16h
kubernetes-dashboard-key-holder          Opaque                                2      14h
traefik-ingress-controller-token-nkfb8   kubernetes.io/service-account-token   3      5d1h
[root@hdss184-243 ~]# kubectl describe secret kubernetes-dashboard-admin-token-w46s2 -n kube-system |tail
Annotations:  kubernetes.io/service-account.name: kubernetes-dashboard-admin
              kubernetes.io/service-account.uid: 11fedd46-3591-4c15-b32d-5818e5aca7d8

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1346 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbi10b2tlbi13NDZzMiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjExZmVkZDQ2LTM1OTEtNGMxNS1iMzJkLTU4MThlNWFjYTdkOCIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTprdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbiJ9.HkVak9znUafeh4JTkzGRiH3uXVjcuHMTOsmz58xJy1intMn25ouC04KK7uplkAtd_IsA6FFo-Kkdqc3VKZ5u5xeymL2ccLaLiCXnlxAcVta5CuwyyO4AXeS8ss-BMKCAfeIldnqwJRPX2nzORJap3CTLU0Cswln8x8iXisA_gBuNVjiWzJ6tszMRi7vX1BM6rp6bompWfNR1xzBWifjsq8J4zhRYG9sVi9Ec3_BZUEfIc0ozFF91Jc5qCk2L04y8tHBauVuJo_ecgMdJfCDk7VKVnyF3Z-Fb8cELNugmeDlKYvv06YHPyvdxfdt99l6QpvuEetbMGAhh5hPOd9roVw
```

### 5.验证toke登录

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_ca78f8eda1b9f2313e4db8f6b8a5a94c_r.png)

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_14d5d69f3353aa8df4299059efd10383_r.png)

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_f7fded22ee708f59a805ef8051417953_r.png)

### 6.升级dashboard为v1.10.1

在hdss184-245.host.com上：

- 更改镜像地址：

  ```shell
  [root@hdss184-245 dashboard]# grep image dp.yaml 
        image: harbor.od.com/public/kubernetes-dashboard-amd64:v1.10.1
  ```

在hdss184-243.host.com上：

- 应用配置：

  ```shell
  [root@hdss184-243 ~]# kubectl apply -f http://k8s-yaml.od.com/dashboard/dp.yaml
  ```

### 7.dashboard 官方给的rbac-minimal

```shell
dashboard]# cat rbac-minimal.yaml 
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: kubernetes-dashboard
    addonmanager.kubernetes.io/mode: Reconcile
  name: kubernetes-dashboard
  namespace: kube-system
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
    addonmanager.kubernetes.io/mode: Reconcile
  name: kubernetes-dashboard-minimal
  namespace: kube-system
rules:
  # Allow Dashboard to get, update and delete Dashboard exclusive secrets.
- apiGroups: [""]
  resources: ["secrets"]
  resourceNames: ["kubernetes-dashboard-key-holder", "kubernetes-dashboard-certs"]
  verbs: ["get", "update", "delete"]
  # Allow Dashboard to get and update 'kubernetes-dashboard-settings' config map.
- apiGroups: [""]
  resources: ["configmaps"]
  resourceNames: ["kubernetes-dashboard-settings"]
  verbs: ["get", "update"]
  # Allow Dashboard to get metrics from heapster.
- apiGroups: [""]
  resources: ["services"]
  resourceNames: ["heapster"]
  verbs: ["proxy"]
- apiGroups: [""]
  resources: ["services/proxy"]
  resourceNames: ["heapster", "http:heapster:", "https:heapster:"]
  verbs: ["get"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: kubernetes-dashboard-minimal
  namespace: kube-system
  labels:
    k8s-app: kubernetes-dashboard
    addonmanager.kubernetes.io/mode: Reconcile
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: kubernetes-dashboard-minimal
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard
  namespace: kube-system
```

## 5.部署heapster

[heapster官方github地址](https://github.com/kubernetes-retired/heapster)

### 1.准备heapster镜像

```shell
[root@hdss184-245 ~]# docker pull sunrisenan/heapster:v1.5.4

[root@hdss184-245 ~]# docker images|grep heapster
sunrisenan/heapster                               v1.5.4                     c359b95ad38b        9 months ago        136MB

[root@hdss184-245 ~]# docker tag c359b95ad38b harbor.od.com/public/heapster:v1.5.4
[root@hdss184-245 ~]# docker push !$
```

### 2.准备资源配置清单

- rbac.yaml

```shell
[root@hdss184-245 ~]# cat /data/k8s-yaml/heapster/rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: heapster
  namespace: kube-system
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: heapster
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:heapster
subjects:
- kind: ServiceAccount
  name: heapster
  namespace: kube-system
```

- Deployment

```shell
[root@hdss184-245 ~]# cat /data/k8s-yaml/heapster/dp.yaml 
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: heapster
  namespace: kube-system
spec:
  replicas: 1
  template:
    metadata:
      labels:
        task: monitoring
        k8s-app: heapster
    spec:
      serviceAccountName: heapster
      containers:
      - name: heapster
        image: harbor.od.com/public/heapster:v1.5.4
        imagePullPolicy: IfNotPresent
        command:
        - /opt/bitnami/heapster/bin/heapster
        - --source=kubernetes:https://kubernetes.default
```

- service

```shell
[root@hdss184-245 ~]# cat /data/k8s-yaml/heapster/svc.yaml 
apiVersion: v1
kind: Service
metadata:
  labels:
    task: monitoring
    # For use as a Cluster add-on (https://github.com/kubernetes/kubernetes/tree/master/cluster/addons)
    # If you are NOT using this as an addon, you should comment out this line.
    kubernetes.io/cluster-service: 'true'
    kubernetes.io/name: Heapster
  name: heapster
  namespace: kube-system
spec:
  ports:
  - port: 80
    targetPort: 8082
  selector:
    k8s-app: heapster
```

### 3.应用资源配置清单

```shell
[root@hdss184-243 ~]# kubectl apply -f http://k8s-yaml.od.com/heapster/rbac.yaml

[root@hdss184-243 ~]# kubectl apply -f http://k8s-yaml.od.com/heapster/dp.yaml

[root@hdss184-243 ~]# kubectl apply -f http://k8s-yaml.od.com/heapster/svc.yaml
```

### 4.重启dashboard(可以不重启)

```shell
[root@hdss184-243 ~]# kubectl delete -f http://k8s-yaml.od.com/dashboard/dp.yaml

[root@hdss184-243 ~]# kubectl apply -f http://k8s-yaml.od.com/dashboard/dp.yaml
```

### 5.检查

- 主机检查

```shell
[root@hdss184-243 ~]# kubectl top node
NAME                 CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
hdss184-243.host.com   149m         3%     3643Mi          47%       
hdss184-244.host.com   130m         3%     3300Mi          42%       

[root@hdss184-243 ~]# kubectl top pod -n kube-public 
NAME                        CPU(cores)   MEMORY(bytes)   
nginx-dp-5dfc689474-dm555   0m           10Mi 
```

- 浏览器检查

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_6e4ce39904dc61713b70c334143ec1b4_r.png)





# 第六章：k8s平滑升级

第一步：观察哪台机器pod少

```shell
[root@hdss184-243 src]# kubectl get nodes
NAME                 STATUS   ROLES         AGE     VERSION
hdss184-243.host.com   Ready    master,node   6d16h   v1.15.2
hdss184-244.host.com   Ready    master,node   6d16h   v1.15.2

[root@hdss184-243 src]# kubectl get pods -n kube-system -o wide
NAME                                    READY   STATUS    RESTARTS   AGE     IP            NODE                 NOMINATED NODE   READINESS GATES
coredns-6b6c4f9648-x5zvz                1/1     Running   0          5d22h   172.6.244.3   hdss184-244.host.com   <none>           <none>
heapster-b5b9f794-s2vj9                 1/1     Running   0          76m     172.6.243.4   hdss184-243.host.com   <none>           <none>
kubernetes-dashboard-5dbdd9bdd7-qlk52   1/1     Running   0          62m     172.6.244.4   hdss184-244.host.com   <none>           <none>
traefik-ingress-bhhkv                   1/1     Running   0          5d4h    172.6.244.2   hdss184-244.host.com   <none>           <none>
traefik-ingress-mm2ds                   1/1     Running   0          5d4h    172.6.243.2   hdss184-243.host.com   <none>           <none>
```

第二步：在负载均衡了禁用7层和4层

```shell
略
```

第三步：摘除node节点

```shell
[root@hdss184-243 src]# kubectl delete node hdss184-243.host.com
node "hdss184-243.host.com" deleted
[root@hdss184-243 src]# kubectl get node
NAME                 STATUS   ROLES         AGE     VERSION
hdss184-244.host.com   Ready    master,node   6d17h   v1.15.2
```

第四步：观察运行POD情况

```shell
[root@hdss184-243 src]# kubectl get pods -n kube-system -o wide
NAME                                    READY   STATUS    RESTARTS   AGE     IP            NODE                 NOMINATED NODE   READINESS GATES
coredns-6b6c4f9648-x5zvz                1/1     Running   0          5d22h   172.6.244.3   hdss184-244.host.com   <none>           <none>
heapster-b5b9f794-dlt2z                 1/1     Running   0          15s     172.6.244.6   hdss184-244.host.com   <none>           <none>
kubernetes-dashboard-5dbdd9bdd7-qlk52   1/1     Running   0          64m     172.6.244.4   hdss184-244.host.com   <none>           <none>
traefik-ingress-bhhkv                   1/1     Running   0          5d4h    172.6.244.2   hdss184-244.host.com   <none>           <none>
```

第五步：检查dns

```shell
[root@hdss184-243 src]# dig -t A kubernetes.default.svc.cluster.local @10.96.0.2 +short
10.96.0.1
```

第六步：开始升级

```shell
[root@hdss184-243 ~]# cd /opt/src/
[root@hdss184-243 src]# wget http://down.sunrisenan.com/k8s/kubernetes/kubernetes-server-linux-amd64-v1.15.4.tar.gz
[root@hdss184-243 src]# tar xf kubernetes-server-linux-amd64-v1.15.4.tar.gz
[root@hdss184-243 src]# mv kubernetes /opt/kubernetes-v1.15.4
[root@hdss184-243 src]# cd /opt/
[root@hdss184-243 opt]# rm -f kubernetes
[root@hdss184-243 opt]# ln -s /opt/kubernetes-v1.15.4 kubernetes
[root@hdss184-243 opt]# cd /opt/kubernetes
[root@hdss184-243 kubernetes]# rm -f kubernetes-src.tar.gz 
[root@hdss184-243 kubernetes]# cd server/bin/
[root@hdss184-243 bin]# rm -f *.tar
[root@hdss184-243 bin]# rm -f *tag

[root@hdss184-243 bin]# cp -r /opt/kubernetes-v1.15.2/server/bin/cert .
[root@hdss184-243 bin]# cp -r /opt/kubernetes-v1.15.2/server/bin/conf .
[root@hdss184-243 bin]# cp -r /opt/kubernetes-v1.15.2/server/bin/*.sh .

[root@hdss184-243 bin]# systemctl restart supervisord.service 

[root@hdss184-243 bin]# kubectl get node
NAME                 STATUS   ROLES         AGE     VERSION
hdss184-243.host.com   Ready    <none>        16s     v1.15.4
hdss184-244.host.com   Ready    master,node   6d17h   v1.15.2
```

升级另一台：

```shell
[root@hdss184-244 src]# kubectl get node
NAME                 STATUS   ROLES         AGE     VERSION
hdss184-243.host.com   Ready    <none>        95s     v1.15.4
hdss184-244.host.com   Ready    master,node   6d17h   v1.15.2

[root@hdss184-244 src]# kubectl delete node hdss184-244.host.com
node "hdss184-244.host.com" deleted

[root@hdss184-244 src]# kubectl get node
NAME                 STATUS   ROLES    AGE    VERSION
hdss184-243.host.com   Ready    <none>   3m2s   v1.15.4

[root@hdss184-244 src]# kubectl get pods -n kube-system -o wide
NAME                                    READY   STATUS    RESTARTS   AGE     IP            NODE                 NOMINATED NODE   READINESS GATES
coredns-6b6c4f9648-bxqcp                1/1     Running   0          20s     172.6.243.3   hdss184-243.host.com   <none>           <none>
heapster-b5b9f794-hjx74                 1/1     Running   0          20s     172.6.243.4   hdss184-243.host.com   <none>           <none>
kubernetes-dashboard-5dbdd9bdd7-gj6vc   1/1     Running   0          20s     172.6.243.5   hdss184-243.host.com   <none>           <none>
traefik-ingress-4hl97                   1/1     Running   0          3m22s   172.6.243.2   hdss184-243.host.com   <none>           <none>

[root@hdss184-244 src]# dig -t A kubernetes.default.svc.cluster.local @10.96.0.2 +short
10.96.0.1
[root@hdss184-244 src]# wget http://down.sunrisenan.com/k8s/kubernetes/kubernetes-server-linux-amd64-v1.15.4.tar.gz
[root@hdss184-244 src]# tar xf kubernetes-server-linux-amd64-v1.15.4.tar.gz
[root@hdss184-244 src]# mv kubernetes /opt/kubernetes-v1.15.4

[root@hdss184-244 ~]# cd /opt/
[root@hdss184-244 opt]# rm -f kubernetes
[root@hdss184-244 opt]# ln -s /opt/kubernetes-v1.15.4 kubernetes
[root@hdss184-244 opt]# cd kubernetes
[root@hdss184-244 kubernetes]# rm -f kubernetes-src.tar.gz 
```





# 第四篇：实战交付一套dubbo微服务到K8S集群



# 第一章：Dubbo微服务概述

## 1.dubbo什么？

- dubbo是阿里巴巴SOA服务化治理方案的核心框架，每天为2000+个服务提供3000000000+次访问量支持，并被广泛应用于阿里巴巴集团的各成员站点
- dubbo是一个分布式服务框架，致力于提供高可用性能和透明化的RPC远程服务调用方案，以及SOA服务治理方案。
- 简单的说，dubbo就是一个服务框架，如果没有分布式的需求，其实是不需要用的，只是在分布式的时候，才有dubbo这样的分布式服务框架的需求，并且本质上是个服务调用的东西，说白了就是个远程服务调用的分布式框架。

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_3cc763ccfe6967147255ec9ca0c48b53_r.png)

## 2.dubbo能做什么？

- 透明化的远程方法调用，就像调用本地方法一样调用远程方法，只需要配置，没有任何API侵入。
- 软负载均衡及容错机制，可在内网替代F5等硬件负载均衡器，降低成本，减少单点。
- 服务自动注册与发现，不再需要写死服务提供方地址，注册中心基于接口名查询服务提供者的IP地址，并且能够平滑添加或删除服务提供者。

# 第二章：实验架构详解

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_151103d7cb4d3faf4e1c2c5cb996febf_r.png)

# 第三章：部署zookeeper集群

- Zookeeper是Dubbo微服务集群的注册中心
- 它的高可用机制和K8S的etcd集群一致
- 由Java编写，所以需要jdk环境

## 1.集群规划

| 主机名      | 角色                      | IP            |
| :---------- | :------------------------ | :------------ |
| hdss184-241 | k8s代理节点1,zk1,jdk      | 192.168.6.241 |
| hdss184-242 | k8s代理节点2,zk2,jdk      | 192.168.6.242 |
| hdss184-243 | k8s运算节点1,zk3,jdk      | 192.168.6.243 |
| hdss184-244 | k8s运算节点2,jenkins      | 192.168.6.244 |
| hdss184-245 | k8s运维节点（docker仓库） | 192.168.6.245 |

## 2.安装jdk1.8（3台zk角色主机）

[JDK_ALL下载地址](https://www.oracle.com/technetwork/java/archive-139210.html)

[jdk1.8下载](http://down.sunrisenan.com/oracle/jdk-8u221-linux-x64.tar.gz)

在hdss184-241机器上：

```shell
[root@hdss184-241 ~]# mkdir /opt/src
[root@hdss184-241 ~]# wget -O /opt/src/jdk-8u221-linux-x64.tar.gz http://down.sunrisenan.com/oracle/jdk-8u221-linux-x64.tar.gz
[root@hdss184-241 ~]# ls -l /opt/src/  | grep jdk
-rw-r--r-- 1 root root 195094741 Nov 28 10:44 jdk-8u221-linux-x64.tar.gz
[root@hdss184-241 ~]# mkdir /usr/java
[root@hdss184-241 ~]# tar xf /opt/src/jdk-8u221-linux-x64.tar.gz -C /usr/java
[root@hdss184-241 ~]# ln -s /usr/java/jdk1.8.0_221 /usr/java/jdk
[root@hdss184-241 ~]# vi /etc/profile
[root@hdss184-241 ~]# tail -4 /etc/profile

export JAVA_HOME=/usr/java/jdk
export PATH=$JAVA_HOME/bin:$JAVA_HOME/bin:$PATH
export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
[root@hdss184-241 ~]# source /etc/profile

[root@hdss184-241 ~]# java -version
java version "1.8.0_221"
Java(TM) SE Runtime Environment (build 1.8.0_221-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.221-b11, mixed mode)
```

注意：这里以hdss184-241为例，分别在hdss184-242，hdss184-243上部署

## 3.安装zookeeper（3台zk角色主机）

[zk下载](https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/)

[zookeeper](http://archive.apache.org/dist/zookeeper/)

### 1.解压配置

```shell
[root@hdss184-241 ~]#   wget -O /opt/src/apache-zookeeper-3.5.9-bin.tar.gz  https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.5.9/apache-zookeeper-3.5.9-bin.tar.gz
[root@hdss184-241 ~]# tar xf /opt/src/apache-zookeeper-3.5.9-bin.tar.gz -C /opt/
[root@hdss184-241 ~]# ln -s /opt/apache-zookeeper-3.5.9-bin /opt/zookeeper
[root@hdss184-241 ~]# mkdir -pv /data/zookeeper/data /data/zookeeper/logs
[root@hdss184-241 ~]# vi /opt/zookeeper/conf/zoo.cfg
[root@hdss184-241 ~]# cat /opt/zookeeper/conf/zoo.cfg
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/data/zookeeper/data
dataLogDir=/data/zookeeper/logs
clientPort=2181
server.1=zk1.od.com:2888:3888
server.2=zk2.od.com:2888:3888
server.3=zk3.od.com:2888:3888
```

注意：各节点zk配置相同

### 2.myid

hdsh6-241.host.com上：

```shell
[root@hdss184-241 ~]# echo "1" > /data/zookeeper/data/myid
```

hdsh6-242.host.com上：

```shell
[root@hdss184-242 ~]# echo "2" > /data/zookeeper/data/myid
```

hdsh6-243.host.com上：

```shell
[root@hdss184-243 ~]# echo "3" > /data/zookeeper/data/myid
```

### 3.做dns解析

hdsh6-241.host.com上：

```shell
[root@hdss184-241 ~]# vi /var/named/od.com.zone 
[root@hdss184-241 ~]# cat /var/named/od.com.zone
$ORIGIN od.com.
$TTL 600    ; 10 minutes
@           IN SOA    dns.od.com. dnsadmin.od.com. (
                2019111209 ; serial
                10800      ; refresh (3 hours)
                900        ; retry (15 minutes)
                604800     ; expire (1 week)
                86400      ; minimum (1 day)
                )
                NS   dns.od.com.
$TTL 60    ; 1 minute
dns                A    192.168.6.241
harbor             A    192.168.6.245
k8s-yaml           A    192.168.6.245
traefik            A    192.168.6.66
dashboard          A    192.168.6.66
zk1                A    192.168.184.241
zk2                A    192.168.184.242
zk3                A    192.168.184.243

[root@hdss184-241 ~]# systemctl restart named.service

[root@hdss184-241 ~]# dig -t A zk1.od.com @192.168.184.241 +short
192.168.6.241
```

### 4.依次启动

```shell
[root@hdss184-241 ~]# /opt/zookeeper/bin/zkServer.sh start

[root@hdss184-242 ~]# /opt/zookeeper/bin/zkServer.sh start

[root@hdss184-243 ~]# /opt/zookeeper/bin/zkServer.sh start
```

### 5.常用命令

- 查看当前角色

```shell
[root@hdss184-241 ~]# /opt/zookeeper/bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /opt/zookeeper/bin/../conf/zoo.cfg
Mode: follower

[root@hdss184-242 ~]# /opt/zookeeper/bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /opt/zookeeper/bin/../conf/zoo.cfg
Mode: leader

[root@hdss184-243 ~]# /opt/zookeeper/bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /opt/zookeeper/bin/../conf/zoo.cfg
Mode: follower
```





# 第四章：部署jenkins

## 1.准备镜像

[jenkins官网](https://jenkins.io/download/)

[jenkins镜像](https://hub.docker.com/_/jenkins)

在运维主机下载官网上的稳定版（这里下载2.190.3）

```shell
[root@hdss184-245 ~]# docker pull jenkins/jenkins:2.190.3
[root@hdss184-245 ~]# docker images | grep jenkins
jenkins/jenkins                                   2.190.3                    22b8b9a84dbe        7 days ago          568MB
[root@hdss184-245 ~]# docker tag 22b8b9a84dbe harbor.od.com/public/jenkins:v2.190.3
[root@hdss184-245 ~]# docker pull !$
docker push harbor.od.com/public/jenkins:v2.190.3
```

## 2.自定义Dockerfile

在运维主机hdss184-245.host.com上：

```shell
[root@hdss184-245 ~]# mkdir -p  /data/dockerfile/jenkins/
[root@hdss184-245 ~]# vi /data/dockerfile/jenkins/Dockerfile
[root@hdss184-245 ~]# cat /data/dockerfile/jenkins/Dockerfile
FROM harbor.od.com/public/jenkins:v2.190.3
USER root
RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime &&\ 
    echo 'Asia/Shanghai' >/etc/timezone
ADD id_rsa /root/.ssh/id_rsa
ADD config.json /root/.docker/config.json
ADD get-docker.sh /get-docker.sh
RUN echo "    StrictHostKeyChecking no" >> /etc/ssh/ssh_config &&\
    /get-docker.sh
```

- get-docker加速版

```shell
FROM harbor.od.com/public/jenkins:v2.190.3
USER root
RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime &&\ 
   echo 'Asia/Shanghai' >/etc/timezone
ADD id_rsa /root/.ssh/id_rsa
ADD config.json /root/.docker/config.json
ADD get-docker.sh /get-docker.sh
RUN echo "    StrictHostKeyChecking no" >> /etc/ssh/ssh_config &&\
   /get-docker.sh --mirror Aliyun   # 阿里云加速
```

这个Dockerfile里我们主要做了以下几件事

- 设置容器用户为root
- 设置容器内的时区
- 将ssh私钥加入（使用git拉取代码时要用到，配置的公钥应配置在gitlab中）
- 加入了登录自建harbor仓库的config文件
- 修改了ssh客户端的配置
- 安装一个docker的客户端

√ 1.生成ssh秘钥：

```shell
[root@hdss184-245 ~]# ssh-keygen -t rsa -b 2048 -C "b02330224@126.com" -N "" -f /root/.ssh/id_rsa

[root@hdss184-245 ~]# cat /root/.ssh/id_rsa.pub   #可以看到自己设置的邮箱
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCxz9kQZuCTd7szAtBQ/BT6kxb73vR3MyCjCvOGwxBIU1IVHLrk1u5FnUOD62o5i15ggP3DWx2KacnK+MYCcf4lKzFmRLQqWiF4iv0RqSXqNJHUO0sNdgaFGlEhbwuMLL8dcv2IZ7ukuMn276hkzx+k45iBg93zAgVw+qeVMXBdIogYiJ9JEWoOsSdG6duEO1LAoGvOCMzukmpr5Oa4/GlbXj/iU9N4w1zlNFBCVm+mzGjuHy2yg//3XORnOZjIhVTJpmfFRkzKcamgxcnXRqQcV/ukmiAW0D7YSereSBvYh7qwLpglZ+Uw1VCi/ibdDLQcE8uKTHytfFr+z4Dt3E9f b02330224@126.com
```

√ 2.拷贝文件

```shell
[root@hdss184-245 ~]# cp /root/.ssh/id_rsa /data/dockerfile/jenkins/

[root@hdss184-245 ~]# cp /root/.docker/config.json /data/dockerfile/jenkins/

[root@hdss184-245 ~]# cd /data/dockerfile/jenkins/ && curl -fsSL get.docker.com -o get-docker.sh && chmod +x get-docker.sh
```

√ 3.查看docker harbor config

```shell
[root@hdss184-245 jenkins]#cat /root/.docker/config.json
{
    "auths": {
        "harbor.od.com": {
            "auth": "YWRtaW46SGFyYm9yMTIzNDU="
        },
        "https://index.docker.io/v1/": {
            "auth": "c3VucmlzZW5hbjpseXo1MjA="
        }
    },
    "HttpHeaders": {
        "User-Agent": "Docker-Client/19.03.5 (linux)"
    }
}
```

## 3.制作自定义镜像

/data/dockerfile/jenkins

```shell
[root@hdss184-245 jenkins]# ls -l
total 28
-rw------- 1 root root   229 Nov 28 13:50 config.json
-rw-r--r-- 1 root root   394 Nov 28 13:15 Dockerfile
-rwxr-xr-x 1 root root 13216 Nov 28 13:53 get-docker.sh
-rw------- 1 root root  1679 Nov 28 13:40 id_rsa


[root@hdss184-245 jenkins]# docker build . -t harbor.od.com/infra/jenkins:v2.190.3
Sending build context to Docker daemon  19.46kB
Step 1/7 : FROM harbor.od.com/public/jenkins:v2.190.3
 ---> 22b8b9a84dbe
Step 2/7 : USER root
 ---> Running in 6347ef23acfd
Removing intermediate container 6347ef23acfd
 ---> ff18352d230e
Step 3/7 : RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime &&    echo 'Asia/Shanghai' >/etc/timezone
 ---> Running in 970da85d013e
Removing intermediate container 970da85d013e
 ---> ca63098fe359
Step 4/7 : ADD id_rsa /root/.ssh/id_rsa
 ---> 0274b5facac2
Step 5/7 : ADD config.json /root/.docker/config.json
 ---> 75d0e57592c3
Step 6/7 : ADD get-docker.sh /get-docker.sh
 ---> a0ec7cf884a4
Step 7/7 : RUN echo "    StrictHostKeyChecking no" >> /etc/ssh/ssh_config &&    /get-docker.sh
 ---> Running in cd18e5417de5
# Executing docker install script, commit: f45d7c11389849ff46a6b4d94e0dd1ffebca32c1
+ sh -c apt-get update -qq >/dev/null
+ sh -c DEBIAN_FRONTEND=noninteractive apt-get install -y -qq apt-transport-https ca-certificates curl >/dev/null
debconf: delaying package configuration, since apt-utils is not installed
+ sh -c curl -fsSL "https://download.docker.com/linux/debian/gpg" | apt-key add -qq - >/dev/null
Warning: apt-key output should not be parsed (stdout is not a terminal)
+ sh -c echo "deb [arch=amd64] https://download.docker.com/linux/debian stretch stable" > /etc/apt/sources.list.d/docker.list
+ sh -c apt-get update -qq >/dev/null
+ [ -n  ]
+ sh -c apt-get install -y -qq --no-install-recommends docker-ce >/dev/null
debconf: delaying package configuration, since apt-utils is not installed
If you would like to use Docker as a non-root user, you should now consider
adding your user to the "docker" group with something like:

  sudo usermod -aG docker your-user

Remember that you will have to log out and back in for this to take effect!

WARNING: Adding a user to the "docker" group will grant the ability to run
         containers which can be used to obtain root privileges on the
         docker host.
         Refer to https://docs.docker.com/engine/security/security/#docker-daemon-attack-surface
         for more information.
Removing intermediate container cd18e5417de5
 ---> 7170e12fccfe
Successfully built 7170e12fccfe
Successfully tagged harbor.od.com/infra/jenkins:v2.190.3
```

## 4.创建infra仓库

在Harbor页面，创建infra仓库，注意：私有仓库

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_fa310a840919e2af2f8bfbf58e9203ab_r.png)

## 5.推送镜像

```shell
[root@hdss184-245 jenkins]# docker push harbor.od.com/infra/jenkins:v2.190.3
```

√ gitee.com 添加私钥，测试jenkins镜像：

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_8c8788f678c45b04375937a4c13290d5_r.png)

```shell
[root@hdss184-245 jenkins]# docker run --rm harbor.od.com/infra/jenkins:v2.190.3 ssh -i /root/.ssh/id_rsa  -T git@gitee.com
Warning: Permanently added 'gitee.com,212.64.62.174' (ECDSA) to the list of known hosts.
Hi Sunrise! You've successfully authenticated, but GITEE.COM does not provide shell access.
```

## 6.创建kubernetes命名空间，私有仓库鉴权

在任意运算节点上：

```shell
[root@hdss184-243 ~]# kubectl create ns infra
namespace/infra created

[root@hdss184-243 ~]# kubectl create secret docker-registry harbor --docker-server=harbor.od.com --docker-username=admin --docker-password=Harbor12345 -n infra
secret/harbor created
```

## 7.准备共享存储

运维主机，以及所有运算节点上：

```shell
[root@hdss184-243 ~]# yum install nfs-utils -y

[root@hdss184-244 ~]# yum install nfs-utils -y

[root@hdss184-245 ~]# yum install nfs-utils -y
```

- 配置NFS服务

运维主机hdss184-245上：

```shell
[root@hdss184-245 ~]# cat /etc/exports
/data/nfs-volume 192.168.6.0/24(rw,no_root_squash)
```

- 启动NFS服务

运维主机hdss184-245上：

```shell
[root@hdss184-245 ~]# mkdir -p  /data/nfs-volume/jenkins_home
[root@hdss184-245 ~]# systemctl start nfs
[root@hdss184-245 ~]# systemctl enable nfs
```

## 8.准备资源配置清单

运维主机hdss184-245上：

```shell
[root@hdss184-245 ~]# mkdir /data/k8s-yaml/jenkins
```

- Deployment

```shell
[root@hdss184-245 ~]# cat /data/k8s-yaml/jenkins/dp.yaml 
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: jenkins
  namespace: infra
  labels: 
    name: jenkins
spec:
  replicas: 1
  selector:
    matchLabels: 
      name: jenkins
  template:
    metadata:
      labels: 
        app: jenkins 
        name: jenkins
    spec:
      volumes:
      - name: data
        nfs: 
          server: hdss184-245
          path: /data/nfs-volume/jenkins_home
      - name: docker
        hostPath: 
          path: /run/docker.sock
          type: ''
      containers:
      - name: jenkins
        image: harbor.od.com/infra/jenkins:v2.190.3
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8080
          protocol: TCP
        env:
        - name: JAVA_OPTS
          value: -Xmx512m -Xms512m
        volumeMounts:
        - name: data
          mountPath: /var/jenkins_home
        - name: docker
          mountPath: /run/docker.sock
      imagePullSecrets:
      - name: harbor
      securityContext: 
        runAsUser: 0
  strategy:
    type: RollingUpdate
    rollingUpdate: 
      maxUnavailable: 1
      maxSurge: 1
  revisionHistoryLimit: 7
  progressDeadlineSeconds: 600
```

- service

```shell
[root@hdss184-245 ~]# cat /data/k8s-yaml/jenkins/svc.yaml 
kind: Service
apiVersion: v1
metadata: 
  name: jenkins
  namespace: infra
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  selector:
    app: jenkins
```

- ingress

```shell
[root@hdss184-245 ~]# cat /data/k8s-yaml/jenkins/ingress.yaml 
kind: Ingress
apiVersion: extensions/v1beta1
metadata: 
  name: jenkins
  namespace: infra
spec:
  rules:
  - host: jenkins.od.com
    http:
      paths:
      - path: /
        backend: 
          serviceName: jenkins
          servicePort: 80
```

## 9.应用资源配置清单

在任意运算节点上：

```shell
[root@hdss184-243 ~]# kubectl apply -f http://k8s-yaml.od.com/jenkins/dp.yaml

[root@hdss184-243 ~]# kubectl apply -f http://k8s-yaml.od.com/jenkins/svc.yaml

[root@hdss184-243 ~]# kubectl apply -f http://k8s-yaml.od.com/jenkins/ingress.yaml
```

- 检查

```shell
[root@hdss184-243 ~]# kubectl apply -f http://k8s-yaml.od.com/jenkins/dp.yaml
deployment.extensions/jenkins created
[root@hdss184-243 ~]# kubectl get all -n infra
NAME                           READY   STATUS    RESTARTS   AGE
pod/jenkins-74f7d66687-gjgth   1/1     Running   0          56m

NAME              TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
service/jenkins   ClusterIP   10.96.2.239   <none>        80/TCP    63m

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/jenkins   1/1     1            1           56m

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/jenkins-74f7d66687   1         1         1       56m
```

## 10.解析域名

在hdss184-241上：

- 增加配置

```shell
[root@hdss184-241 ~]# cat /var/named/od.com.zone 
$ORIGIN od.com.
$TTL 600    ; 10 minutes
@           IN SOA    dns.od.com. dnsadmin.od.com. (
                2019111210 ; serial    # 滚动加一
                10800      ; refresh (3 hours)
                900        ; retry (15 minutes)
                604800     ; expire (1 week)
                86400      ; minimum (1 day)
                )
                NS   dns.od.com.
$TTL 60    ; 1 minute
dns                A    192.168.184.241
harbor             A    192.168.184.245
k8s-yaml           A    192.168.184.245
traefik            A    192.168.184.66
dashboard          A    192.168.184.66
zk1                A    192.168.184.241
zk2                A    192.168.184.242
zk3                A    192.168.184.243
jenkins            A    192.168.184.66       # 添加解析
```

- 重启，检查

```shell
[root@hdss184-241 ~]# systemctl restart named
[root@hdss184-241 ~]# dig -t A jenkins.od.com @192.168.184.241 +short
192.168.184.66
```

## 11.配置jenkins加速

- jenkins插件清华大学镜像地址

  ```shell
  [root@hdss184-245 ~]# wget -O /data/nfs-volume/jenkins_home/updates/default.json https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json
  ```

- 其他方法

操作步骤

以上的配置Json其实在Jenkins的工作目录中

```shell
$ cd {你的Jenkins工作目录}/updates  #进入更新配置位置
```

第一种方式：使用vim

```shell
$ vim default.json   #这个Json文件与上边的配置文件是相同的

这里wiki和github的文档不用改，我们就可以成功修改这个配置

使用vim的命令，如下，替换所有插件下载的url

:1,$s/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g

替换连接测试url

:1,$s/http:\/\/www.google.com/https:\/\/www.baidu.com/g

    进入vim先输入：然后再粘贴上边的：后边的命令，注意不要写两个冒号！

修改完成保存退出:wq
```

第二种方式：使用sed

```shell
$ sed -i 's/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' default.json && sed -i 's/http:\/\/www.google.com/https:\/\/www.baidu.com/g' default.json

    这是直接修改的配置文件，如果前边Jenkins用sudo启动的话，那么这里的两个sed前均需要加上sudo
```

[重启Jenkins，安装插件试试，简直超速](https://www.cnblogs.com/hellxz/p/jenkins_install_plugins_faster.html)！！

## 12.浏览器访问

浏览器访问 http://jenkins.od.com/

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_9b05da934bea1cc9b12790e4f06ba227_r.png)

## 13.页面配置jenkins

### 1.初始化密码

```shell
[root@hdss184-243 ~]# kubectl get pods -n infra
NAME                       READY   STATUS    RESTARTS   AGE
jenkins-74f7d66687-gjgth   1/1     Running   0          68m
[root@hdss184-243 ~]# kubectl exec jenkins-74f7d66687-gjgth /bin/cat /var/jenkins_home/secrets/initialAdminPassword -n infra 
59be7fd64b2b4c18a3cd927e0123f609


[root@hdss184-245 ~]# cat /data/nfs-volume/jenkins_home/secrets/initialAdminPassword 
59be7fd64b2b4c18a3cd927e0123f609
```

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_4079ad580f9908779fad463a361da95e_r.png)

### 2.跳过安装插件

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_142d1541420384969ad4a2bf66cc5c8a_r.png)

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_a33ab46e65c3d187b7443cd43a4cb8d6_r.png)

### 3.更改admin密码

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_ad3ee426b7710d427a4c2bfc652e4212_r.png)

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_fb066a63442d974943d2dc9d30dc0e6e_r.png)

### 4.使用admin登录

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_a373a959e064dce76be35505b4c88af1_r.png)

### 5.调整安全选项

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_b254e056cf8d6ccd78e8b862a36fcd97_r.png)

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_8cbb70f4591522b61501ad019b544c53_r.png)

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_dfbaa5b108f88df82246abf817183507_r.png)

### 6.安装Blue Ocean插件

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_0c43496501b1ffbc81e0de1225829eb0_r.png)

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_50c51fccc496c23857fc0d00efbe49b4_r.png)

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_111097df8cd6513890005c24dee704b6_r.png)

> 我们勾上这个允许匿名登录主要也是配合最后spinnaker

如果不允许匿名访问可进行如下操作：

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_6dc2fe58f288ecb137322e5da9f854c6_r.png)

![null](http://www.sunrisenan.com/uploads/kubernetes/images/m_8c4e6c8b6d1c0155b7859960e5dfc69c_r.png)

## 14.配置New job

- create new jobs
- Enter anitem name

> dubbo-demo

- Pipeline –> ok
- Discard old builds

> Days to keep builds：3
> Max # of builds to keep:30

- This project is parameterized

1.Add Parameter –> String Parameter

> Name：app_name
> Default Value:
> Description：project name，e.g：dubbo-demo-service

2.Add Parameter -> String Parameter

> Name : image_name
> Default Value :
> Description : project docker image name. e.g: app/dubbo-demo-service

3.Add Parameter -> String Parameter

> Name : git_repo
> Default Value :
> Description : project git repository. e.g: https://gitee.com/stanleywang/dubbo-demo-service.git

4.Add Parameter -> String Parameter

> Name : git_ver
> Default Value :
> Description : git commit id of the project.

5.Add Parameter -> String Parameter

> Name : add_tag
> Default Value :
> Description : project docker image tag, date_timestamp recommended. e.g: 190117_1920

6.Add Parameter -> String Parameter

> Name : mvn_dir
> Default Value : ./
> Description : project maven directory. e.g: ./

7.Add Parameter -> String Parameter

> Name : target_dir
> Default Value : ./target
> Description : the relative path of target file such as .jar or .war package. e.g: ./dubbo-server/target

8.Add Parameter -> String Parameter

> Name : mvn_cmd
> Default Value : mvn clean package -Dmaven.test.skip=true
> Description : maven command. e.g: mvn clean package -e -q -Dmaven.test.skip=true

9.Add Parameter -> Choice Parameter

> Name : base_image
> Default Value :
>
> - base/jre7:7u80
> - base/jre8:8u112
>   Description : project base image list in harbor.od.com.

10.Add Parameter -> Choice Parameter

> Name : maven
> Default Value :
>
> - 3.6.0-8u181
> - 3.2.5-6u025
> - 2.2.1-6u025
>   Description : different maven edition.

## 15.Pipeline Script

```shell
pipeline {
  agent any 
    stages {
      stage('pull') { //get project code from repo 
        steps {
          sh "git clone ${params.git_repo} ${params.app_name}/${env.BUILD_NUMBER} && cd ${params.app_name}/${env.BUILD_NUMBER} && git checkout ${params.git_ver}"
        }
      }
      stage('build') { //exec mvn cmd
        steps {
          sh "cd ${params.app_name}/${env.BUILD_NUMBER}  && /var/jenkins_home/maven-${params.maven}/bin/${params.mvn_cmd}"
        }
      }
      stage('package') { //move jar file into project_dir
        steps {
          sh "cd ${params.app_name}/${env.BUILD_NUMBER} && cd ${params.target_dir} && mkdir project_dir && mv *.jar ./project_dir"
        }
      }
      stage('image') { //build image and push to registry
        steps {
          writeFile file: "${params.app_name}/${env.BUILD_NUMBER}/Dockerfile", text: """FROM harbor.od.com/${params.base_image}
ADD ${params.target_dir}/project_dir /opt/project_dir"""
```