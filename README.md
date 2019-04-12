# zookeeper-ansible
1. 通过ansible自动化安装zookeeper的docker集群
2. 确保zookeeper集群节点数为2N+1
3. 目前仅支持Centos7.x版本
4. zookeeper 镜像版本为 docker.io/zookeeper:3.4.13  
5. 因为集群间各容器使用随机端口进行心跳检查,存在从容器中访问其他节点容器不通的情况。本文直接使用host网络模式解决节点间不通的问题

## Configuration  vars/zk.yml
-----------------------------
1、更新vars/zk.yml中的 mount_path 宿主机挂载目录
# env path
mount_path: "/data/zookeeper"   #zookeeper宿主机挂载目录

2、指定vars/zk.yml中容器启动的端口
# zookeeper port
client_port: 2181
leader_port: 2888
vote_port: 3888
jmx_port: 11911
random_port: "30001-65006"  #heartbeat port

3、指定用户、组
# zookeeper user                   
user: "zookeeper"
group: "zookeeper"

4、指定主机组名 [与host中配置一致]
# host group
zk_hosts: zookeeper   # the group define in hosts

---------------------------
#主机组文件
1、hosts中指定集群机器
[zookeeper]
192.168.xx.xx
192.168.xx.xx
192.168.xx.xx

[zookeeper:vars]
ansible_ssh_user=xxx
ansible_ssh_pass=xxx
ansible_ssh_port=xxx




## Install
1. check  zookeeper.yml
2. install_zk, config_zk, add_user, open_firewall 修改为false即可取消执行

----------------------

- hosts: zookeeper

  user: root

  vars_files:

    - "vars/zk.yml"

  vars:
    config_zk: true       # config zookeeper

    install_zk: true      # install zookeeper

    add_user: true        # add zookeeper user 

    open_firewall: false   # open firewall

  roles:

    - user                # add user

    - zookeeper           # zookeeper

```
3. 安装zookeeper集群

```
ansible-playbook -i hosts zookeeper.yml

```



docker zookeeper
environment:
  ZOO_MY_ID: 1
  ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888

mount:
  /data
  /conf
  /datalog


  docker run -d -v /data/zookeeper/data:/data -v /data/zookeeper/conf/:/conf -v /data/zookeeper/datalog:/datalog -e ZOO_SERVERS="server.1=ip:2888:3888" -e ZOO_MY_ID=1 -p 2181:2181 -p 2888:2888 -p 3888:3888 --name zook1 docker.io/zookeeper:3.4.13

  检查节点状态
  echo ruok|nc host_ip 2181  -> imok

  echo stats|nc host_ip 2181
  Zookeeper version: 3.4.13-2d71af4dbe22557fda74f9a9b4309b15a7487f03, built on 06/29/2018 04:05 GMT
Clients:
 /172.17.0.1:47970[0](queued=0,recved=1,sent=0)

Latency min/avg/max: 0/0/0
Received: 1
Sent: 0
Connections: 1
Outstanding: 0
Zxid: 0x0
Mode: standalone
Node count: 4

主机组中各主机需要安装docker及playbook使用的docker-py模块
yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
yum install docker-ce docker-ce-cli containerd.io

#安装pip
python install python-pip -y

#安装docker-py module
pip install docker-py
