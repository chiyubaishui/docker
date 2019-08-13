在介绍VOLUME指令之前，我们来看下如下场景需求：
1）容器是基于镜像创建的，最后的容器文件系统包括镜像的只读层+可写层，容器中的进程操作的数据持久化都是保存在容器的可写层上。一旦容器删除后，这些数据就没了，除非我们人工备份下来（或者基于容器创建新的镜像）。能否可以让容器进程持久化的数据保存在主机上呢？这样即使容器删除了，数据还在。
2）当我们在开发一个web应用时，开发环境是在主机本地，但运行测试环境是放在docker容器上。
这样的话，我在主机上修改文件（如html，js等）后，需要再同步到容器中。这显然比较麻烦。
3）多个容器运行一组相关联的服务，如果他们要共享一些数据怎么办？
对于这些问题，我们当然能想到各种解决方案。而docker本身提供了一种机制，可以将主机上的某个目录与容器的某个目录（称为挂载点、或者叫卷）关联起来，容器上的挂载点下的内容就是主机的这个目录下的内容，这类似linux系统下mount的机制。 这样的话，我们修改主机上该目录的内容时，不需要同步容器，对容器来说是立即生效的。 挂载点可以让多个容器共享。
下面我们来介绍具体的机制。

一、通过docker run命令
1、运行命令：docker run --name test -it -v /home/xqh/myimage:/data ubuntu /bin/bash
其中的 -v 标记 在容器中设置了一个挂载点 /data（就是容器中的一个目录），并将主机上的 /home/xqh/myimage 目录中的内容关联到 /data下。
这样在容器中对/data目录下的操作，还是在主机上对/home/xqh/myimage的操作，都是完全实时同步的，因为这两个目录实际都是指向主机目录。
2、运行命令：docker run --name test1 -it -v /data ubuntu /bin/bash
上面-v的标记只设置了容器的挂载点，并没有指定关联的主机目录。这时docker会自动绑定主机上的一个目录。通过docker inspect 命令可以查看到。
"Mounts": [
            {
                "Name": "2aebfaef1bb42f2486ed236abe897441f64a3e880c64c012fca113022d3aec50",
                "Source": "/local/docker/var/lib/docker/volumes/2aebfaef1bb42f2486ed236abe897441f64a3e880c64c012fca113022d3aec50/_data",
                "Destination": "/local/daasdocker/sofeware",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            },
上面 Mounts下的每条信息记录了容器上一个挂载点的信息，"Destination" 值是容器的挂载点，"Source"值是对应的主机目录。
可以看出这种方式对应的主机目录是自动创建的，其目的不是让在主机上修改，而是让多个容器共享。
二、通过dockerfile创建挂载点
上面介绍的通过docker run命令的-v标识创建的挂载点只能对创建的容器有效。
通过dockerfile的 VOLUME 指令可以在镜像中创建挂载点，这样只要通过该镜像创建的容器都有了挂载点。
还有一个区别是，通过 VOLUME 指令创建的挂载点，无法指定主机上对应的目录，是自动生成的。
语法：
VOLUME ["/var/log/"]
VOLUME /var/log
VOLUME /var/log /var/db
VOLUME ["/data1","/data2"]
上面的dockfile文件通过VOLUME指令指定了两个挂载点 /data1 和 /data2.
我们通过docker inspect 查看通过该dockerfile创建的镜像生成的容器
可以看到两个挂载点的信息。
三、容器共享卷（挂载点）
docker run --name test1 -it myimage /bin/bash
上面命令中的 myimage是用前面的dockerfile文件构建的镜像。 这样容器test1就有了 /data1 和 /data2两个挂载点。
下面我们创建另一个容器可以和test1共享 /data1 和 /data2卷 ，这是在 docker run中使用 --volumes-from标记，如：
可以是来源不同镜像，如：
docker run --name test2 -it --volumes-from test1  ubuntu  /bin/bash
也可以是同一镜像，如：
docker run --name test3 -it --volumes-from test1  myimage  /bin/bash
上面的三个容器 test1 , test2 , test3 均有 /data1 和 /data2 两个目录，且目录中内容是共享的，任何一个容器修改了内容，别的容器都能获取到。



在docker container和物理机中双向拷贝文件
容器内部文件拷贝到宿主机：
sh-4.2# docker cp jupyter-70002111:/home/70002111/教程-研究功能介绍.ipynb .
sh-4.2# ls
Dockerfile  教程-研究功能介绍.ipynb
宿主机文件拷贝到容器：
sh-4.2# docker cp Dockerfile jupyter-70002111:/home/70002111/
sh-4.2# docker exec -it jupyter-70002188 ls 
Dockerfile
