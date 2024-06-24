---
sidebar_position: 2
---

# Docker Compose 快速安装

使用 [Docker Compose](https://docs.docker.com/compose/) 快速部署 Cloudpods Baremetal 物理机管理服务。

## 环境准备

### 机器配置要求

- 最低配置要求: CPU 2核, 内存 4GiB, 存储 50GiB
- docker 版本: ce-23.0.2
    - docker 建议安装最新的 ce 版本，新版本已经包含 docker-compose 插件
    - docker 需要开启容器网络以及 iptables

### 安装配置 docker

:::tip 注意
如果您的环境已经安装了新版本的 docker ，可以跳过改步骤。
:::

请自行参考官方文档安装：[Install Docker Engine](https://docs.docker.com/engine/install/) 。

## 运行 Cloudpods Baremetal 服务

下载 ocboot 工具到本地。

```bash
$ git clone -b release/3.11 https://github.com/yunionio/ocboot && cd ./ocboot
```

进入 compose/baremetal 目录，该目录包含了使用 docker compose 运行 baremetal 服务相关的配置文件。

```bash
$ cd compose/baremetal
$ ls -alh docker-compose.yml
```

运行服务，注意需要设置 LISTEN_INTERFACE 和 PUBLIC_IP 两个环境变量。

- **LISTEN_INTERFACE**: 服务监听的网卡，比如 eth0 ，改网卡会负责接受 DHCP 请求。
- **PUBLIC_IP**: 服务监听的 IP 地址，为对应 LISTEN_INTERFACE 网卡上的 IP 地址，可通过 `ip addr show` 查看对应网卡上的地址。

下面命令假设 eth0 网卡上的 ip 地址为 10.168.222.205，具体设置请根据自己的环境设置。

```bash
$ LISTEN_INTERFACE=eth0 PUBLIC_IP=10.168.222.205 docker compose up
```

等服务启动完成后，就可以登陆 **https://$PUBLIC_IP** 访问前端服务，默认登陆用户密码为：admin 和 admin@123 。


## 操作说明

### 1. 服务放到后台运行

可以使用 '-d/--detach' 参数把所有服务放到后台运行，命令如下：

```bash
# 所有服务放到后台运行
$ docker compose up -d

# 服务放到后台后，可以通过 logs 自命令查看输出日志
$ docker compose logs -f
```

### 2. 登陆 climc 命令行容器

如果要使用命令行工具对平台做操作，可以使用下面的方法进入容器：

```bash
$ docker exec -ti compose-climc-1 bash
Welcome to Cloud Shell :-) You may execute climc and other command tools in this shell.
Please exec 'climc' to get started

# source 认证信息
bash-5.1# source /etc/yunion/rcadmin
bash-5.1# climc user-list
```

### 3. 查看服务配置和持久化数据

所有服务的持久化数据都是存储在 *ocboot/compose/data* 目录下面的，所有配置都是自动生成的，一般不需要手动修改，下面对各个目录做说明：

```bash
$ tree data
data
├── etc
│   ├── nginx
│   │   └── conf.d
│   │       └── default.conf    # 前端 nginx 配置
│   └── yunion
│       ├── *.conf  # cloudpods 各个服务配置
│       ├── pki     # 证书目录
│       ├── rcadmin     # 命令行认证信息
├── opt
│   └── cloud
│       └── workspace
│           └── data
│               └── glance # 镜像服务存储的镜像目录
└── var
    └── lib
        ├── influxdb    # influxdb 持久化数据目录
        └── mysql       # mysql 数据库持久化数据目录
```

### 4. 删除所有容器

所有服务的持久化数据都是存储在 *ocboot/compose/data* 目录下面的，删除容器不会丢失数据，下次直接用 *docker compose up* 重启即可，操作如下：

```bash
# 删除服务
$ docker compose down
```

## 常见问题

### 1. docker 服务没有打开 iptables 和 bridge 导致容器网路无法创建

默认情况下，启动 docker 服务是默认打开 iptables 的，如果在 */etc/docker/daemon.json* 里面设置了 "bridge: none" 和 "iptables: false" 则无法使用 docker compose 功能。

在运行 docker compose 之前请确保打开了 bridge 和 iptables 功能。
