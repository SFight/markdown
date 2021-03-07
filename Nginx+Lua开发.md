# Nginx+Lua开发 OpenResty

## 简介

OpenResty（也称为 ngx_openresty）是一个全功能的 Web 应用服务器。它打包了标准的 Nginx 核心，很多的常用的第三方模块，以及它们的大多数依赖项。

通过揉和众多设计良好的 Nginx 模块，OpenResty 有效地把 Nginx 服务器转变为一个强大的 Web 应用服务器，基于它开发人员可以使用 Lua 编程语言对 Nginx 核心以及现有的各种 Nginx C 模块进行脚本编程，构建出可以处理一万以上并发请求的极端高性能的 Web 应用。

OpenResty 致力于将你的服务器端应用完全运行于 Nginx 服务器中，充分利用 Nginx 的事件模型来进行非阻塞 I/O 通信。不仅仅是和 HTTP 客户端间的网络通信是非阻塞的，与MySQL、PostgreSQL、Memcached 以及 Redis 等众多远方后端之间的网络通信也是非阻塞的。

## 一、Openesty安装

笔者使用的是macOS系统，此处指做macOS系统的相关安装及问题处理，其他系统可参见：

[OpenResty安装](https://openresty.org/cn/installation.html)

```shell
brew install openresty/brew/openresty
# 报错信息如下：
Error: Your Command Line Tools are too outdated.
Update them from Software Update in System Preferences or run:
  softwareupdate --all --install --force

If that doesn't show you any updates, run:
  sudo rm -rf /Library/Developer/CommandLineTools
  sudo xcode-select --install
```

安装完成如下：

```shell
If you need to have openresty-openssl111 first in your PATH, run:
  echo 'export PATH="/usr/local/opt/openresty-openssl111/bin:$PATH"' >> ~/.zshrc

For compilers to find openresty-openssl111 you may need to set:
  export LDFLAGS="-L/usr/local/opt/openresty-openssl111/lib"
  export CPPFLAGS="-I/usr/local/opt/openresty-openssl111/include"

For pkg-config to find openresty-openssl111 you may need to set:
  export PKG_CONFIG_PATH="/usr/local/opt/openresty-openssl111/lib/pkgconfig"

==> openresty
To have launchd start openresty/brew/openresty now and restart at login:
  brew services start openresty/brew/openresty
Or, if you don't want/need a background service you can just run:
  openresty
```

我们设置路径变量，以便后续方便操作：

```shell
# OpenResty配置
export OpenResty="/usr/local/opt/openresty-openssl111/bin"
export LDFLAGS="-L/usr/local/opt/openresty-openssl111/lib"
export CPPFLAGS="-I/usr/local/opt/openresty-openssl111/include"

export PATH="$PATH:$OpenResty"
```

之后我们就可以像使用nginx命令一样使用openresty来操作nginx，加载我们想要的数据，编写我们的lua文件了。

```shell
openresty -h
# 结果如下：
nginx version: openresty/1.19.3.1
Usage: nginx [-?hvVtTq] [-s signal] [-c filename] [-p prefix] [-g directives]

Options:
  -?,-h         : this help
  -v            : show version and exit
  -V            : show version and configure options then exit
  -t            : test configuration and exit
  -T            : test configuration, dump it and exit
  -q            : suppress non-error messages during configuration testing
  -s signal     : send signal to a master process: stop, quit, reopen, reload
  -p prefix     : set prefix path (default: /usr/local/Cellar/openresty/1.19.3.1_1/nginx/)
  -c filename   : set configuration file (default: /usr/local/etc/openresty/nginx.conf)
  -g directives : set global directives out of configuration file
```

## 二、开始使用Openesty

* 创建我们的工作目录：

```shell
mkdir openresty
cd openresty
mkdir logs/ conf/
touch conf/nginx.conf
```

* 创建nginx.conf文件，并初始化数据：

```shell
touch conf/nginx.conf
# 输入如下内容
worker_processes  1;
error_log logs/error.log;
events {
    worker_connections 1024;
}
http {
    server {
        listen 8080;
        location / {
            default_type text/html;
            content_by_lua_block {
                ngx.say("<p>hello, world</p>")
            }
        }
    }
}
```

* 测试配置文件

```shell
cd openresty
openresty -p `pwd` -c conf/nginx.conf -t
# 启动
openresty -p `pwd` -c conf/nginx.conf
```

* 访问[localhost:8080](http://localhost:8080)

出现hello, world证明我们已经成功搭建好openresty环境。开始我们的redis连接吧。

## 三、使用lua连接redis

* 创建lua文件目录

```shell
cd openresty
mkdir lua
```

* 创建redis连接lua操作文件

```shell
touch lua/redis.lua
```
