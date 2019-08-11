容器命令
①：运行镜像

docker run - --name container-name -d image-name

②：查看容器列表

docker ps

③：通过以下命令查看运行和停止的容器

docker ps -a

④：停止容器

docker stop container-name/container-id

⑤：启动容器

docker start container-name/container-id
docker cp :用于容器与主机之间的数据拷贝。
语法
docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH
实例
将主机/www/runoob目录拷贝到容器96f7f14e99ab的/www目录下。
docker cp /www/runoob 96f7f14e99ab:/www/
将主机/www/runoob目录拷贝到容器96f7f14e99ab中，目录重命名为www。
docker cp /www/runoob 96f7f14e99ab:/www
将容器96f7f14e99ab的/www目录拷贝到主机的/tmp目录中。
docker cp  96f7f14e99ab:/www /tmp/