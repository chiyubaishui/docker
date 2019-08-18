1.0下载镜像,启动容器
root@ubuntu:~# docker pull ubuntu:18.04

启动容器：
root@ubuntu:/# run -it -d --name myubuntu ubuntu:18.04 /bin/bash
root@2d7c5e1c06d8:/#
查看ubuntu镜像的发行版本号：
root@2d7c5e1c06d8:/# cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=18.04
DISTRIB_CODENAME=bionic
DISTRIB_DESCRIPTION="Ubuntu 18.04.1 LTS"

执行apt-get update命令更新仓库信息，之后便可以通过apt-get命令安装软件：

1.1安装和配置SSH服务
选择openshh-server作为服务端，安装openssh-server的命令如下：

root@2d7c5e1c06d8:/# apt-get install openssh-server
Reading package lists... Done
Building dependency tree
Reading state information... Done
......
如果要正常启动SSH服务，那么目录/var/run/sshd必须存在，下面手动创建：

root@2d7c5e1c06d8:/# mkdir -p /var/run/sshd
接下来启动SSH服务：

root@2d7c5e1c06d8:/# /usr/sbin/sshd -D &
[1] 657
此时查看容器的22端口，可见此端口已经处于监听状态：

root@2d7c5e1c06d8:/# netstat -lntp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      657/sshd
tcp6       0      0 :::22                   :::*                    LISTEN      657/sshd
修改SSH服务的安全配置，取消pam登录限制：

root@2d7c5e1c06d8:/# sed -ri 's/session required pam_loginuid.so/#session required pam_loginuid.so/g' /etc/pam.d/sshd
root@2d7c5e1c06d8:/#
在root用户目录下创建.ssh目录，并复制需要登录的公钥信息到authorized_keys文件中：

root@2d7c5e1c06d8:/# mkdir root/.ssh
root@2d7c5e1c06d8:/#
root@2d7c5e1c06d8:/#
root@2d7c5e1c06d8:/# ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:wNq35+CSXxsXsWL2cNx+ZGQe4sucSPnOBdyxMeVlsfI root@a5503782666d
The key's randomart image is:
+---[RSA 2048]----+
|               .=|
|     .         o+|
|      o     .o *+|
|     o .   .++=+*|
|    . . S =o=+.E+|
|       . +.==o+o |
|       .o +.o*...|
|      o. = +o .. |
|       oo o  o   |
+----[SHA256]-----+
root@2d7c5e1c06d8:/#
root@2d7c5e1c06d8:/# ll root/.ssh/
total 16
drwxr-xr-x 2 root root 4096 Mar 10 12:59 ./
drwx------ 1 root root 4096 Mar 10 12:58 ../
-rw------- 1 root root 1675 Mar 10 12:59 id_rsa
-rw-r--r-- 1 root root  399 Mar 10 12:59 id_rsa.pub
root@2d7c5e1c06d8:/#
root@2d7c5e1c06d8:/# cp root/.ssh/id_rsa.pub root/.ssh/authorized_keys
创建自动启动SSH服务的可执行文件run.sh，并添加可执行权限：

root@2d7c5e1c06d8:/# vi run.sh
root@2d7c5e1c06d8:/# chmod +x run.sh
run.sh脚本内容如下：

#!/bin/bash
/usr/sbin/sshd -D
最后，退出容器：

root@2d7c5e1c06d8:/# exit
exit
1.2 保存镜像
将容器用docker commit命令保存为一个新的sshd:ubuntu镜像：

root@ubuntu:~# docker commit 2d7 sshd:ubuntu
sha256:dd6399386b1600d276389d42922c6e7b294f4506acaf131c1dcf1a7f52d24818
使用docker images查看本地目前的镜像列表：

root@ubuntu:~# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
sshd                ubuntu              dd6399386b16        13 seconds ago      225MB
ubuntu              latest              47b19964fb50        4 weeks ago         88.1MB
1.3 使用镜像
启动容器，并添加端口映射10022~22，其中10022是宿主主机的端口，22是容器SSH服务监听的端口：

root@ubuntu:~# run -it --name myssh -d -p 10022:22 sshd:ubuntu sh /run.sh
0fadd9f9f458d7164d4e81beb24fea94b84ae68d483f370216a1710fa2ee274d
启动成功后，可以在宿主机上看到容器运行的详细信息：

root@ubuntu:~# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                   NAMES
0fadd9f9f458        sshd:ubuntu         "/run.sh"           52 seconds ago      Up 50 seconds       0.0.0.0:10022->22/tcp   clever_brattain
在宿主主机或其他主机上，可以通过SSH访问10022端口来登录容器：

root@ubuntu:~# ssh 192.168.10.128 -p 10022
The authenticity of host '[192.168.10.128]:10022 ([192.168.10.128]:10022)' can't be established.
ECDSA key fingerprint is SHA256:poSthw3GeJ/cYNmBhBiLk2jeoBSEKNtLlWmDZg9sG2A.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[192.168.10.128]:10022' (ECDSA) to the list of known hosts.
root@2d7c5e1c06d8:/#
