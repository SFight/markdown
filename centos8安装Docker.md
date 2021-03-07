# centos8安装Docker

### 参考地址：[https://yq.aliyun.com/articles/748412?spm=5176.10695662.1996646101.searchclickresult.1fcf7180bsGelQ](https://yq.aliyun.com/articles/748412?spm=5176.10695662.1996646101.searchclickresult.1fcf7180bsGelQ)

1、卸载老版本

```shell
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

2、安装docker基础包

```shell
yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

3、设置稳定仓库

```shell
yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

4、安装最新版本docker

```shell
yum install docker-ce docker-ce-cli containerd.io
```

5、centos8可能会有container.io版本过低问题

```shell
wget https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm  
yum install containerd.io-1.2.6-3.3.el7.x86_64.rpm #安装containerd.io
#再次重新安装docker
yum install docker-ce docker-ce-cli containerd.io
```

6、启动和测试

```shell
sudo systemctl start docker  # 启动docker
docker run hello-world  #测试
service docker status
service docker start
```
