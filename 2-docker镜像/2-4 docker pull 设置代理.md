你在终端设置代理的时候docker pull的时候是不会走代理的，下面是docker pull设置代理的正确方式
操作
环境是在centos下，如果没有新建下面这个文件夹
sudo mkdir -p /etc/systemd/system/docker.service.d
之后新建下面这个文件走http代理
vim /etc/systemd/system/docker.service.d/http-proxy.conf
填入
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:80/"

编辑下面这个文件走https代理
vim /etc/systemd/system/docker.service.d/https-proxy.conf
[Service]
Environment="HTTPS_PROXY=https://proxy.example.com:443/"

之后你使用docker pull的时候就可以pull gcr.io上的镜像了

