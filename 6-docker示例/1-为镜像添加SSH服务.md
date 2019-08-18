1.0���ؾ���,��������
root@ubuntu:~# docker pull ubuntu:18.04

����������
root@ubuntu:/# run -it -d --name myubuntu ubuntu:18.04 /bin/bash
root@2d7c5e1c06d8:/#
�鿴ubuntu����ķ��а汾�ţ�
root@2d7c5e1c06d8:/# cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=18.04
DISTRIB_CODENAME=bionic
DISTRIB_DESCRIPTION="Ubuntu 18.04.1 LTS"

ִ��apt-get update������²ֿ���Ϣ��֮������ͨ��apt-get���װ�����

1.1��װ������SSH����
ѡ��openshh-server��Ϊ����ˣ���װopenssh-server���������£�

root@2d7c5e1c06d8:/# apt-get install openssh-server
Reading package lists... Done
Building dependency tree
Reading state information... Done
......
���Ҫ��������SSH������ôĿ¼/var/run/sshd������ڣ������ֶ�������

root@2d7c5e1c06d8:/# mkdir -p /var/run/sshd
����������SSH����

root@2d7c5e1c06d8:/# /usr/sbin/sshd -D &
[1] 657
��ʱ�鿴������22�˿ڣ��ɼ��˶˿��Ѿ����ڼ���״̬��

root@2d7c5e1c06d8:/# netstat -lntp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      657/sshd
tcp6       0      0 :::22                   :::*                    LISTEN      657/sshd
�޸�SSH����İ�ȫ���ã�ȡ��pam��¼���ƣ�

root@2d7c5e1c06d8:/# sed -ri 's/session required pam_loginuid.so/#session required pam_loginuid.so/g' /etc/pam.d/sshd
root@2d7c5e1c06d8:/#
��root�û�Ŀ¼�´���.sshĿ¼����������Ҫ��¼�Ĺ�Կ��Ϣ��authorized_keys�ļ��У�

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
�����Զ�����SSH����Ŀ�ִ���ļ�run.sh������ӿ�ִ��Ȩ�ޣ�

root@2d7c5e1c06d8:/# vi run.sh
root@2d7c5e1c06d8:/# chmod +x run.sh
run.sh�ű��������£�

#!/bin/bash
/usr/sbin/sshd -D
����˳�������

root@2d7c5e1c06d8:/# exit
exit
1.2 ���澵��
��������docker commit�����Ϊһ���µ�sshd:ubuntu����

root@ubuntu:~# docker commit 2d7 sshd:ubuntu
sha256:dd6399386b1600d276389d42922c6e7b294f4506acaf131c1dcf1a7f52d24818
ʹ��docker images�鿴����Ŀǰ�ľ����б�

root@ubuntu:~# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
sshd                ubuntu              dd6399386b16        13 seconds ago      225MB
ubuntu              latest              47b19964fb50        4 weeks ago         88.1MB
1.3 ʹ�þ���
��������������Ӷ˿�ӳ��10022~22������10022�����������Ķ˿ڣ�22������SSH��������Ķ˿ڣ�

root@ubuntu:~# run -it --name myssh -d -p 10022:22 sshd:ubuntu sh /run.sh
0fadd9f9f458d7164d4e81beb24fea94b84ae68d483f370216a1710fa2ee274d
�����ɹ��󣬿������������Ͽ����������е���ϸ��Ϣ��

root@ubuntu:~# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                   NAMES
0fadd9f9f458        sshd:ubuntu         "/run.sh"           52 seconds ago      Up 50 seconds       0.0.0.0:10022->22/tcp   clever_brattain
���������������������ϣ�����ͨ��SSH����10022�˿�����¼������

root@ubuntu:~# ssh 192.168.10.128 -p 10022
The authenticity of host '[192.168.10.128]:10022 ([192.168.10.128]:10022)' can't be established.
ECDSA key fingerprint is SHA256:poSthw3GeJ/cYNmBhBiLk2jeoBSEKNtLlWmDZg9sG2A.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[192.168.10.128]:10022' (ECDSA) to the list of known hosts.
root@2d7c5e1c06d8:/#
