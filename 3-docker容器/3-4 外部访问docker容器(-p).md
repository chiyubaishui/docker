容器中可以运行一些网络应用，要让外部也可以访问这些应用，可以通过 -P（大写） 或 -p （小写） 参数来指定端口映射。
（1）当使用 -P 标记时，Docker 会随机映射一个 49000~49900 的端口到内部容器开放的网络端口。
使用 docker ps 可以看到，本地主机的 49155 被映射到了容器的 5000 端口。此时访问本机的 49155 端口即可访问容器内 web 应用提供的界面。
$ sudo docker run -d -P training/webapp python app.py
$ sudo docker ps -l
CONTAINER ID  IMAGE                   COMMAND       CREATED        STATUS        PORTS                    NAMES
bc533791f3f5  training/webapp:latest  python app.py 5 seconds ago  Up 2 seconds  0.0.0.0:49155->5000/tcp  nostalgic_morse

同样的，可以通过 docker logs 命令来查看应用的信息。
$ sudo docker logs -f nostalgic_morse
* Running on http://0.0.0.0:5000/
10.0.2.2 - - [23/May/2014 20:16:31] "GET / HTTP/1.1" 200 -
10.0.2.2 - - [23/May/2014 20:16:31] "GET /favicon.ico HTTP/1.1" 404 -

（2）-p（小写）则可以指定要映射的IP和端口，但是在一个指定端口上只可以绑定一个容器。支持的格式有 hostPort:containerPort、ip:hostPort:containerPort、 ip::containerPort。
hostPort:containerPort（映射所有接口地址）
将本地的 5000 端口映射到容器的 5000 端口，可以执行如下命令：
$ sudo docker run -d -p 5000:5000 training/webapp python app.py 此时默认会绑定本地所有接口上的所有地址。
ip:hostPort:containerPort （映射指定地址的指定端口）
指定映射使用一个特定地址，比如 localhost 地址 127.0.0.1
$ sudo docker run -d -p 127.0.0.1:5000:5000 training/webapp python app.py
ip::containerPort （映射指定地址的任意端口）
绑定 localhost 的任意端口到容器的 5000 端口，本地主机会自动分配一个端口。
sudo docker run -d -p 127.0.0.1::5000 training/webapp python app.py
还可以使用 udp 标记来指定 udp 端口
$ sudo docker run -d -p 127.0.0.1:5000:5000/udp training/webapp python app.py
查看映射端口配置
使用 docker port 来查看当前映射的端口配置，也可以查看到绑定的地址
$ docker port nostalgic_morse 5000
127.0.0.1:49155.

注意：
容器有自己的内部网络和 ip 地址（使用 docker inspect 可以获取所有的变量，Docker 还可以有一个可变的网络配置。）
-p 标记可以多次使用来绑定多个端口
例如
$ sudo docker run -d -p 5000:5000 -p 3000:80 training/webapp python app.py


