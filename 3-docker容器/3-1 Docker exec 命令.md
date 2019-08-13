docker exec ：在运行的容器中执行命令
语法
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
OPTIONS说明：
-d :分离模式: 在后台运行
-i :即使没有附加也保持STDIN 打开
-t :分配一个伪终端
-e: 设置环境变量
-u，用户名或UID，例如myuser:myusergroup

1、在容器 mynginx 中以交互模式执行容器内 /root/runoob.sh 脚本:
runoob@runoob:~$ docker exec -it mynginx /bin/sh /root/runoob.sh
http://www.runoob.com/
在容器 mynginx 中开启一个交互模式的终端:
runoob@runoob:~$ docker exec -i -t  mynginx /bin/bash
root@b1a0703e41e7:/#
也可以通过 docker ps -a 命令查看已经在运行的容器，然后使用容器 ID 进入容器。
查看已经在运行的容器 ID：
# docker ps -a 
...
9df70f9a0714        openjdk             "/usercode/script.sh…" 
...
第一列的 9df70f9a0714 就是容器 ID。
通过 exec 命令对指定的容器执行 bash:
# docker exec -it 9df70f9a0714 /bin/bash

2、执行命令并设置环境变量
$ docker exec -e VAR=1 ubuntu_bash bash

3、通常COMMAND只能是一条语句，为了支持多个命令的执行，需要将多个命令连接起来交给Shell，docker exec命令的使用示例如下：

sudo docker exec myContainer bash -c "cd /home/myuser/myproject && git fetch ssh://gerrit_server:29418/myparent/myproject ${GERRIT_REFSPEC} && git checkout FETCH_HEAD";
sudo docker exec myContainer bash -c "cd /home/myuser/myproject;git fetch ssh://gerrit_server:29418/myparent/myproject ${GERRIT_REFSPEC};git checkout FETCH_HEAD";

4、创建容器时传入环境变量
在实际应用场景中，不论是从安全还是可配置方面去考虑，很多参数是比较适合用环境变量加载进去的，比如数据库的连接信息，时区，还有字体支持等等，在创建容器的时候其实都可以使用-e 指定key/value进行传递环境变量进去。
sh-4.2# docker run -itd --name test-env -e TZ='Asia/Shanghai' 172.25.46.9:5001/centos6.8-jdjr-test-app 
ee20b44301e27c16eae63dab243d293054178dd5f819c23d44bd9e534208bb42
sh-4.2# docker exec -it test-env date
2017年 01月 17日 星期二 10:35:17 CST
sh-4.2# date
Tue Jan 17 10:35:21 CST 2017
可以看到加了时区环境变量的容器已经和宿主机在同一个时区(CST)，并且时间和宿主机基本同步

sh-4.2# docker run -itd --name test  172.25.46.9:5001/centos6.8-jdjr-test-app
d6a02874b999ff4eea79e3b302148b42043af01c89a5d31e5d858e0806f9077a
sh-4.2# docker exec -it test date
2017年 01月 20日 星期五 01:43:48 Asia
默认没有加时区环境变量的容器还是Asia


