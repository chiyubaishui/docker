一、修改已有镜像
1、先使用下载的镜像启动容器。
# docker run -it --name mynginx -d nginx
1fe79bf8610c4ca8dc8d751b516ffca561f21407264c763a7b2928e805f0fae1
2、在容器中添加文件。
# docker exec -it 1fe79bf8610c /bin/bash
# cd /usr/share/nginx/html/
# echo "hello">index.html 
当结束后，我们使用 exit 来退出，现在我们的容器已经被我们改变了，使用 docker commit 命令来提交更新后的副本。
# docker commit -m 'modify index file' -a 'xuquan599' 1fe79bf8610c mynginx:0.1
其中，-m 来指定提交的说明信息，跟我们使用的版本控制工具一样；-a 可以指定更新的用户信息；之后是用来创建镜像的容器的 ID；最后指定目标镜像的仓库名和 tag 信息。创建成功后会返回这个镜像的 ID 信息。
使用 docker images 来查看新创建的镜像。
# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
mynginx             0.1                 163092c88161        7 seconds ago       126MB
nginx               latest              e445ab08b2be        11 days ago         126MB
之后，可以使用新的镜像来启动容器
二、利用 Dockerfile 来创建镜像
使用 docker commit 来扩展一个镜像比较简单，但是不方便在一个团队中分享。我们可以使用 docker build 来创建一个新的镜像。为此，首先需要创建一个 Dockerfile，包含一些如何创建镜像的指令。
三、从本地文件系统导入
要从本地文件系统导入一个镜像，可以使用 openvz（容器虚拟化的先锋技术）的模板来创建：openvz 的模板下载地址为templates 。
比如，先下载了一个 ubuntu-14.04 的镜像，之后使用以下命令导入：
cat ubuntu-14.04-x86_64-minimal.tar.gz |docker import - ubuntu:14.04
然后查看新导入的镜像。
docker images
REPOSITORY     TAG         IMAGE ID      CREATED       VIRTUAL SIZE
ubuntu       14.04        05ac7c0b9383    17 seconds ago   215.5 MB

 
