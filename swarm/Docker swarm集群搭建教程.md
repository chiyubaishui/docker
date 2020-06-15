1、卸载旧版本(如果安装过旧版本的话)
yum remove docker  docker-common docker-selinux docker-engine
2、安装需要的软件包
yum install -y yum-utils device-mapper-persistent-data lvm2 wget
3、设置yum源
 cd /etc/yum.repos.d/
https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo
替换国内的源：
https://download.docker.com     https://mirrors.tuna.tsinghua.edu.cn/docker-ce
4、安装docker
yum install docker-ce
5、简单配置docker
新建docker配置文件：/etc/docker/daemon.json
docker镜像加速：
{
"registry-mirrors": ["https://registry.docker-cn.com"]
}
6、启动并加入开机启动
 systemctl start docker
systemctl enable docker
7、验证安装是否成功(有client和service两部分表示docker安装启动都成功了)
docker version
Client:
 Version:           18.09.6
 API version:       1.39
 Go version:        go1.10.8
 Git commit:        481bc77156
 Built:             Sat May  4 02:34:58 2019
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          18.09.6
  API version:      1.39 (minimum version 1.12)
  Go version:       go1.10.8
  Git commit:       481bc77
  Built:            Sat May  4 02:02:43 2019
  OS/Arch:          linux/amd64
  Experimental:     false


 docker info            
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 18.09.6
Storage Driver: overlay2
 Backing Filesystem: xfs
 Supports d_type: true
 Native Overlay Diff: true
Logging Driver: json-file
Cgroup Driver: cgroupfs
Plugins:
 Volume: local
 Network: bridge host macvlan null overlay
 Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
Swarm: inactive
Runtimes: runc
Default Runtime: runc
Init Binary: docker-init
containerd version: bb71b10fd8f58240ca47fbb579b9d1028eea7c84
runc version: 2b18fe1d885ee5083ef9f0838fee39b62d653e30
init version: fec3683
Security Options:
 seccomp
  Profile: default
Kernel Version: 3.10.0-957.1.3.el7.x86_64
Operating System: CentOS Linux 7 (Core)
OSType: linux
Architecture: x86_64
CPUs: 1
Total Memory: 982.1MiB
Name: d40
ID: FUMU:UPPO:NZ5P:E5IC:UNZ5:QTLJ:QTX6:ZH6G:4TDW:BOO3:ZTJR:Q6RP
Docker Root Dir: /var/lib/docker
Debug Mode (client): false
Debug Mode (server): false
Registry: https://index.docker.io/v1/
Labels:
Experimental: false
Insecure Registries:
 127.0.0.0/8
Registry Mirrors:
 https://registry.docker-cn.com/
Live Restore Enabled: false
Product License: Community Engine
7 修改docker监听端口
Swarm是通过监听2375端口进行通信的，所以在使用Swarm进行集群管理之前，需要设置一下2375端口的监听。所有主机节点docker开启2375,2377（swarm集群）监听，docker版本不同，配置方式不一样
vim /lib/systemd/system/docker.service
在ExecStart加入:
-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock

8、重启docker服务
systemctl daemon-reload ##使配置文件生效
systemctl restart docker