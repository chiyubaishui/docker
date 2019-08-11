1.Docker 镜像命令
①：镜像检索
docker search  镜像名——》docker search nginx
  1） 搜索官方docker hub 仓库的镜像
  2） -f (--filter) 参数 过滤输出内容
 docker search  --filter=STARS=5 nginx
②：镜像下载
docker pull 镜像名 [ : tag ]———》docker pull nginx:1.14-alpine
1） 如果不显式指定TAG，则默认会选择lastest标签
2） 从下载的过程中看出，镜像文件一般由若干层（layer）组成，使用docker pull命令下载中会获取并输出镜像的各层信息。当不同的镜像包括相同的层时，本地仅存储了层的一份内容，减小了存储的空间。
3）默认情况下docker pull会从docker hub拉取镜像文件，也可以手动指定一个仓库地址拉取镜像。假如你设置了一个本地仓库地址，那么你只要指定这个地址拉取镜像即可。仓库地址类似一个URL，但是没有协议头http://。
例如从一个镜像地址：myregistry.local:5000，拉取镜像文件：testing/test-image：
$ docker pull myregistry.local:5000/testing/test-image
③：查看本地镜像    （ docker images  --no-trunc）
docker images
docker tag命令来为本地镜像任意添加新的标签
# docker tag nginx:latest mynginx:latest
# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              e445ab08b2be        11 days ago         126MB
mynginx             latest              e445ab08b2be        11 days ago         126MB
docker inspect : 获取容器/镜像的元数据。
docker  inspect  -f='{{ .State.Pid }} {{ .Id }}' $(docker ps -aq)
④：镜像 指定删除
docker rmi image-id  （rm          Remove one or more containers）
清理临时的镜像文件
 # docker image prune -f
Total reclaimed space: 100B
 