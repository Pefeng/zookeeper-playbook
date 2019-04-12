# zookeeper-ansible
1. 通过ansible自动化安装zookeeper的docker集群
2. 确保zookeeper集群节点数为2N+1
3. 目前仅支持Centos7.x版本
4. zookeeper 版本为 3.4.13  
下载链接地址：https://archive.apache.org/dist/zookeeper/zookeeper-3.4.13/zookeeper-3.4.13.tar.gz  


## Configuration

1. 下载zookeeper至本地任意文件夹
2. 更新vars/zk.yml 中的 download_path 路径
```
---
# zookeeper version
zookeeper_version: 3.4.13

# zookeeper user                   
user: "zookeeper"
group: "zookeeper"

# zookeeper data path
data_dir: zookeeper_storage
zookeeper_log_path: "{{install_dir}}/zookeeper_log"  

# zookeeper port
leader_port: 12888
vote_port: 13888
client_port: 12181
jmx_port: 11911
random_port: "30001-65006"

firewall_ports:
  - "{{ leader_port }}"
  - "{{ vote_port }}"
  - "{{ client_port }}"
  - "{{ jmx_port }}"
  - "{{ random_port }}"

# env path
install_dir: "/home/{{ user }}"
download_path: "/home/pippo/Downloads/"               # your local download path
tmp_path: "/tmp"

# host group
zk_hosts: zookeeper                                   # the group define in hosts/host

```

## Install
1. check  zookeeper.yml
2. install_zk, config_zk, start_service, add_user, open_firewall 修改为false即可取消执行
```
---

- hosts: zookeeper

  user: root

  vars_files:

    - "vars/zk.yml"

  vars:

    install_zk: true      # install zookeeper

    config_zk: true       # config zookeeper

    start_service: true   # auto start zookeeper service

    add_user: true        # add zookeeper user 

    open_firewall: true   # open firewall

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


  docker run -d -v /data/zookeeper/data:/data -v /data/zookeeper/conf/:/conf -v /data/zookeeper/datalog:/datalog -e ZOO_SERVERS="server.1=172.16.0.5:2888:3888" -e ZOO_MY_ID=1 -p 2181:2181 -p 2888:2888 -p 3888:3888 --name zook1 docker.io/zookeeper:3.4.13

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

主机组中各主机需要安装docker
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
