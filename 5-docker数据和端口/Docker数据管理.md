一、什么是数据卷
　　生成环境中使用docker的过程中，往往需要对数据进行持久化，或者需要多个容器之间进行数据共享，这个就涉及到了容器数据管理

　　容器中管理数据主要有两种方式：

数据卷：容器内数据之间映射到本地主机环境
数据卷容器：使用特定的容器来维护数据卷
　　数据卷是一个可供容器使用的特殊目录，它将主机操作系统目录直接映射进容器，类似Linux 的mount挂载

　　数据卷的特性

数据卷可以在容器之间共享和重用，容器间传递数据将变得高效方便
对数据卷内数据的修改立马生效，无论是容器内还是本地操作
对数据卷的更新不会影响镜像，解耦了应用和数据
卷会一直存在，直到没有容器使用，可以安全的卸载它
回到顶部
二、创建数据卷
　　在用docker run 命令的时候，使用 -v 标记可以在容器内创建一个数据卷。多次重复使用 -v 标记可以创建多个数据卷

[root@rsync131 ~]# docker run -it -P -d --name web -v /opt/web/ centos
b0772ffc49ea226536a23ca5d73f1a69c4212407ad54ab406b1b098138e64a65
　　PS：-P参数是将容器服务暴露的端口，是自动映射到本地主机的临时端口

　　也可以挂载主机目录做我数据卷(经常使用的方式)

[root@rsync131 ~]# docker run -it -P -d --name web1 -v /opt/web:/opt/ centos
acb1b0f0c804ca2b9bbe8c1fe0373c6025c51342e138204e40e74ac9123c8854
　　上面的命令是将主机的/opt/web 目录挂载到容器的/opt/下，这样是很方便的在你进行数据修改的是，直接修改本地文件就行了，容器直接就会同步了

　　docker 挂载数据卷的默认权限是读写(rw) ,我们也可以改权限

[root@rsync131 ~]# docker run -it -P -d --name web2 -v /opt/web:/opt/:ro centos
8400a6999dbafe14dd6cb28796dc84595282361e619a8d183db405106e07d85d
　　加了:ro 容器内对所挂载的数据卷内的数据就不能修改了

　　也可以只挂载本地主机的单个文件到容器中作为数据卷(强烈不推荐，生产也很少用，可忽略)

[root@rsync131 ~]# docker run -it -P -d --name web4 -v /root/.bash_history:/.bash_history  centos
b5a45865eca2ad16cde207c9a35702667f3bd9b947383a33451b67098548331e
　　

回到顶部
三、数据卷容器
　　如果用户需要在多个容器之间共享一些持续更新的数据，最简单的方式是使用数据卷容器，数据卷容器也是个容器，但是它的目的是专门用来提供数据卷供其他容器挂载的

　　首先创建一个数据卷容器

[root@rsync131 /]# docker run -it -d -v /dbdata --name dbdata centos
ea8c0b5014166f57afc5d7cf52b1ca532e0e0a259f414d2891d0ebf9dc397624
[root@rsync131 /]# docker exec -it ea8c0b5014166f57afc5d7cf52b1ca532e0e0a259f414d2891d0ebf9dc397624 bash
[root@ea8c0b501416 /]# ls
anaconda-post.log  bin  dbdata  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
　　挂载

[root@rsync131 /]# docker run -it -d --volumes-from dbdata --name db1 centos
ad8b9458a39c85516f2dc88645af8f3725418d5f4bf058566b0cbc977352490c
[root@rsync131 /]# docker run -it -d --volumes-from dbdata --name db2 centos
b07899fb1a46aa5538fc82dceebc4e53d7abf74ab870e14f916a59a03d1fab45
　　此时，容器db1和db2都挂载同一个数据卷到相同的/dbdata 目录，三个容器任何一方在该目录下进行操作，其他的容器都能看见

　　测试
# db1 容器
[root@rsync131 /]# docker exec -it db1 bash
[root@ad8b9458a39c /]# cd /
[root@ad8b9458a39c /]# ls
anaconda-post.log  bin  dbdata  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@ad8b9458a39c /]# cd dbdata/
[root@ad8b9458a39c dbdata]# ls
[root@ad8b9458a39c dbdata]# touch  a.txt
[root@ad8b9458a39c dbdata]# ls
a.txt
[root@ad8b9458a39c dbdata]# exit
exit
 
# db2 容器
[root@rsync131 /]# docker exec -it db2 bash
[root@b07899fb1a46 /]# cd /d
dbdata/ dev/
[root@b07899fb1a46 /]# cd /dbdata/
[root@b07899fb1a46 dbdata]# ls
a.txt
　　其实在有容器也可以对db1或者db2进行挂载

[root@rsync131 /]# docker run -it -d --volumes-from db1 --name db3 centos
95547b8422226a2f18dd71db4e8c551fd8df72771cf6001cfdfb963899b71477
[root@rsync131 /]# docker exec -it db3 bash
[root@95547b842222 /]# ls /dbdata/
a.txt
　　--volumes-from 参数所挂载数据卷的容器自身并不需要保持正在运行

　　如果删除了挂载的容器(包括dbdata、db1和db2)，数据卷并不会被自动的删除，如果要删除一个数据卷，必须在删除最后一个还挂载着它的容器时使用docker rm -v命令来指定同时删除关联的容器

回到顶部
四、利用数据卷容器来迁移数据
4.1. 备份
　　使用下面的命令来备份dbdata数据卷容器内的数据卷
[root@rsync131 ~]# docker run --volumes-from dbdata -v $(pwd):/backup --name backup1 centos tar zcvf /backup/backup.tar.gz /dbdata
/dbdata/
/dbdata/a.txt
[root@rsync131 ~]# ll
总用量 273540
-rw-------. 1 root root      1513 8月  20 20:25 anaconda-ks.cfg
-rw-r--r--  1 root root       142 9月  23 14:13 backup.tar.gz
 
# 具体的意思是：利用centos镜像创建一个容器，使用--volumes-from dbdata参数来让容器挂载dbdata容器的数据卷，使用-v 参数来挂载本地当前目录到容器的/backup目录,容器启动后使用tar打包备份到本地目录
4.2. 恢复
　　为了恢复我们创建一个容器

[root@rsync131 ~]# docker run -it -d -v /dbdata --name db8 centos
042d3c9567154d047977e8097939215bef441d3206c81c6bca99b6d227501169
[root@rsync131 ~]# docker run --volumes-from db8 -v $(pwd):/backup centos tar zxvf /backup/backup.tar.gz
dbdata/
dbdata/a.txt
[root@rsync131 ~]# docker exec -it db8 bash
[root@042d3c956715 /]# ls
anaconda-post.log  bin  dbdata  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@042d3c956715 /]# cd dbdata/
[root@042d3c956715 dbdata]# ls
a.txt