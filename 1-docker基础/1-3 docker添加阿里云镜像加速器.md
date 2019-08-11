1、docker添加阿里云镜像加速器
https://blog.csdn.net/chenjin_chenjin/article/details/86674521

2、配置阿里云加速器
阿里云会根据账号生成一个账号加速器地址，例如：
https://k9e55i4n.mirror.aliyuncs.com

将加速器地址配置到docker的daemon.json文件中：
# 编辑daemon.json
vim /etc/docker/daemon.json
# 设置加速器地址
{
  "registry-mirrors": ["https://k9e55i4n.mirror.aliyuncs.com"]
}

最后重新加载和重启docker：
systemctl daemon-reload
systemctl restart docker