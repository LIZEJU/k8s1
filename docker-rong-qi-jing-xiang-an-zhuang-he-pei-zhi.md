# docker容器镜像安装和配置

## docker容器镜像安装和配置

namespace 隔离&#x20;

cgroup 资源配额

### 安装源&#x20;

wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

### 安装docker依赖

yum install -y yum-utils device-mapper-persistent-data lvm2

### 基础软件包&#x20;

yum install -y wget net-tools nfs-utils lrzsz gcc gcc-c++ make cmake libxml2-devel openssl-devel curl curl-devel unzip sudo ntp libaio-devel wget vim ncurses-devel autoconf automake zlib-devel python-devel epel-release openssh-server socat ipvsadm conntrack ntpdate

### 安装docker-ce&#x20;

yum install docker-ce docker-ce-cli containerd.io -y

docker-ce-cli 作用是 docker 命令行工具包&#x20;

containerd.io 作用是容器接口相关包&#x20;

yum info 软件包的名字，可以查看一个包的具体作用



systemctl start docker && systemctl enable docker

### 查看docker版本信息&#x20;

docker version

### 开启包转发功能和修改内核参数

```
modprobe br_netfilter echo "modprobe br_netfilter" >> /etc/profile

cat > /etc/sysctl.d/docker.conf <<EOF 
net.bridge.bridge-nf-call-ip6tables = 1 
net.bridge.bridge-nf-call-iptables = 1 
net.ipv4.ip_forward = 1 
EOF
sysctl -p /etc/sysctl.d/docker.conf
systemctl restart docker
```

### 配置镜像加速器&#x20;

```
tee /etc/docker/daemon.json << 'EOF' 
{ "registry-mirrors":["https://rsbud4vc.mirror.aliyuncs.com","https://registry.docker-cn.com","https://docker.mirrors.ustc.edu.cn","https://dockerhub.azk8s.cn","http://hub-mirror.c.163.com","http://qtid6917.mirror.aliyuncs.com","https://rncxm540.mirror.aliyuncs. com","https://e9yneuy4.mirror.aliyuncs.com"] } 
EOF
```

重启 docker 服务使配置生效

```
systemctl daemon-reload && systemctl restart docker
```

docker run -d -p 80:80 指定宿主机端口是80和容器的端口80连接，映射 -P 宿主机随机端口和容器端口80连接，映射

docker exec -it container\_id /bin/bash 进入容器

### docker 网络模式&#x20;

bridge docker默认的网络驱动

host 直接借用宿主机的网络

none 此驱动不构成网络环境，采用了none网络驱动，那么只能使用loopback网络设备，容器只能使用127.0.0.1的本机网络

container 这个新创建的容器和已经存在的一个容器共享一个network namespace ,而不是和宿主机共享。和一个指定的容器共享ip,端口范围。

docker network list 查看docker网络&#x20;

docker network inspect bridge 查看网桥的子网网络

veth pair 成对出现

### brctl

yum install bridge-utils -y

brctl show

bridge的docker0 是虚拟出来的网桥，因此无法被外部的网络访问。因此需要在运行容器时通过-p和-P 参数将容器的端口映射到宿主机的端口。

docker采用的nat的方式，将容器内部的服务监听端口与宿主机的某一个端口port进行绑定，使得宿主机外部可以将网络报文发送到容器。

### 基于dockerfile构建镜像

```
ROM centos
MAINTAINER xuegod-Pod 
RUN yum install wget -y 
RUN yum install nginx -y 
COPY index.html /usr/share/nginx/html/ 
EXPOSE 80 
ENTRYPOINT ["/usr/sbin/nginx","-g","daemon off;"]
docker build -t="xxx/yyy:v1" .
docker run -d -p 80 --name xxx xx/yy:v1
```

CMD：为启动的容器指定默认要运行的程序，程序运行结束，容器也就结束。CMD 指令指定的程 序可被 docker run 命令行参数中指定要运行的程序所覆盖。如果 Dockerfile 中如果存在多个 CMD 指令，仅最后一个生效。

ENTRYPOINT： 类似于 CMD 指令，但其不会被 docker run 的命令行参数指定的指令所覆盖， 而且这些命令行参数会被当作参数送给 ENTRYPOINT 指令指定的程序。如果 Dockerfile 中如果 存在多个 ENTRYPOINT 指令，仅最后一个生效。如果运行 docker run 时使用了 --entrypoint 选项，将覆盖 CMD 指令指定的程序。如果 Dockerfile 中如果存在多个 ENTRYPOINT 指令， 仅最后一个生效。

docker run -c 传参

HEALTHCHECK：用于指定某个程序或者指令来监控 docker 容器服务的运行状态

VOLUME：定义匿名数据卷。在启动容器时忘记挂载数据卷，会自动挂载到匿名卷。 避免重要的数据，因容器重启而丢失，这是非常致命的。 避免容器不断变大

```
#!/bin/bash 
rm -rf /run/httpd/* 
exec /usr/sbin/apachectl -D FOREGROUND #启动容器时启动服务
```
