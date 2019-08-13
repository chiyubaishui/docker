一、什么是仓库
　　仓库就是集中存放镜像的地方，分为公共仓库和私有仓库，一个容易混淆的概念是注册服务器，实际上注册服务器是存放仓库的具体服务器，一个注册服务器可以有多个仓库，而每个仓库下面有多个镜像

回到顶部
二、使用Docker官方公共仓库
　　官方仓库地址:DockerHub

2.1. 登录仓库
　　使用docker login 命令来登录官方的公共仓库
[root@rsync131 ~]# docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: brianzjz
Password:
Login Succeeded
 
没有用户名密码的可以去官方注册一个
2.2. 下载镜像
　　使用docker pull 命令下载镜像
[root@rsync131 ~]# docker pull nginx
Using default tag: latest
Trying to pull repository docker.io/library/nginx ...
latest: Pulling from docker.io/library/nginx
802b00ed6f79: Pull complete
e9d0e0ea682b: Pull complete
d8b7092b9221: Pull complete
Digest: sha256:24a0c4b4a4c0eb97a1aabb8e29f18e917d05abfe1b7a7c07857230879ce7d3d3
Status: Downloaded newer image for docker.io/nginx:latest
2.3. 推送镜像
　　使用docker push 命令推送镜像

[root@rsync131 ~]# docker push brianzjz/nginx:v1.0
The push refers to a repository [docker.io/brianzjz/nginx]
579c75bb43c0: Mounted from library/nginx
67d3ae5dfa34: Mounted from library/nginx
8b15606a9e3e: Mounted from library/nginx
v1.0: digest: sha256:c0b69559d28fb325a64c6c8f47d14c26b95aa047312b29c699da10380e90b4d7 size: 948

三、国内第三方公共仓库
　　下载镜像：docker pull ccr.ccs.tencentyun.com/[namespace]/[ImageName]:[镜像版本号]

回到顶部
四、搭建自己的本地仓库
　　这个私有仓库才是重点，在我们的企业中很多都会是自己去定制自己的镜像文件，有一些还必须不能对外开放的，也是能够为了提高在内网的下载速度，这样我们就要搭建自己的私有仓库了

4.1. 使用registry镜像创建私有仓库
　　安装docker后，可以通过官方提供的registry镜像来简单的搭建一套本地的私有仓库
[root@rsync131 ~]# mkdir -p /opt/data/docker/                                                  # 创建存放镜像的目录
[root@rsync131 ~]# docker run -it -d -p 5000:5000 -v /opt/data/docker:/tmp/registry registry   # 创建容器 -p指定端口 -v数据卷挂载
Unable to find image 'registry:latest' locally
Trying to pull repository docker.io/library/registry ...
latest: Pulling from docker.io/library/registry
d6a5679aa3cf: Pull complete
ad0eac849f8f: Pull complete
2261ba058a15: Pull complete
f296fda86f10: Pull complete
bcd4a541795b: Pull complete
Digest: sha256:5a156ff125e5a12ac7fdec2b90b7e2ae5120fa249cf62248337b6d04abc574c8
Status: Downloaded newer image for docker.io/registry:latest
6df20cdacf8d4a40ad3cbd3d310299650d41d0757b535130a0daa19f21ee1901
[root@rsync131 ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
6df20cdacf8d        registry            "/entrypoint.sh /e..."   8 seconds ago       Up 7 seconds        0.0.0.0:5000->5000/tcp   dazzling_golick
 

4.2. 管理私有仓库
　　用另一台机器来测试上传下载私有仓库
[root@nfs133 ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
[root@nfs133 ~]# docker pull nginx
Using default tag: latest
Trying to pull repository docker.io/library/nginx ...
latest: Pulling from docker.io/library/nginx
802b00ed6f79: Pull complete
e9d0e0ea682b: Pull complete
d8b7092b9221: Pull complete
Digest: sha256:24a0c4b4a4c0eb97a1aabb8e29f18e917d05abfe1b7a7c07857230879ce7d3d3
Status: Downloaded newer image for docker.io/nginx:latest
[root@nfs133 ~]# docker tag nginx 192.168.73.131:5000/nginx_test
[root@nfs133 ~]# docker images
REPOSITORY                       TAG                 IMAGE ID            CREATED             SIZE
192.168.73.131:5000/nginx_test   latest              06144b287844        2 weeks ago         109 MB
docker.io/nginx                  latest              06144b287844        2 weeks ago         109 MB
=================================================
[root@nfs133 ~]# docker push 192.168.73.131:5000/nginx_test
The push refers to a repository [192.168.73.131:5000/nginx_test]
Get https://192.168.73.131:5000/v1/_ping: http: server gave HTTP response to HTTPS client
 
这里执行的时候报错了，原因就是新版本的docker对安全性要求高了，会要求仓库支持SSL/TLS证书
 
解决方法：
修改daemon配置文件
[root@nfs133 ~]# vim /etc/sysconfig/docker
OPTIONS='--selinux-enabled --log-driver=journald --signature-verification=false  # 原来的配置
修改为：
OPTIONS='--selinux-enabled --log-driver=journald --signature-verification=false --insecure-registry 192.168.73.131:5000'
保存后，重启docker
[root@nfs133 ~]# systemctl restart docker
 
=================================================
# 重新执行push
[root@nfs133 ~]# docker push 192.168.73.131:5000/nginx_test
The push refers to a repository [192.168.73.131:5000/nginx_test]
579c75bb43c0: Pushed
67d3ae5dfa34: Pushed
8b15606a9e3e: Pushed
latest: digest: sha256:c0b69559d28fb325a64c6c8f47d14c26b95aa047312b29c699da10380e90b4d7 size: 948
 
 
# curl 测试
# 温馨提示：下面的执行是错的 因为registry现在已经是2.0版本了
[root@nfs133 ~]# curl http://192.168.73.131:5000/v1/search
404 page not found
 
# 正确的方式
[root@nfs133 ~]# curl -X GET http://192.168.73.131:5000/v2/_catalog
{"repositories":["nginx_test"]}
 
# 下载私有仓库镜像
[root@nfs133 ~]# docker pull 192.168.73.131:5000/nginx_test
Using default tag: latest
Trying to pull repository 192.168.73.131:5000/nginx_test ...
latest: Pulling from 192.168.73.131:5000/nginx_test
Digest: sha256:c0b69559d28fb325a64c6c8f47d14c26b95aa047312b29c699da10380e90b4d7
Status: Downloaded newer image for 192.168.73.131:5000/nginx_test:latest
[root@nfs133 ~]# docker images
REPOSITORY                       TAG                 IMAGE ID            CREATED             SIZE
192.168.73.131:5000/nginx_test   latest              06144b287844        2 weeks ago         109 MB
docker.io/nginx                  latest              06144b287844        2 weeks ago         109 MB
