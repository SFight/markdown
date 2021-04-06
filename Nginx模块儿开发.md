# Nginx扩展Lua模块儿

## 简介

开发Nginx扩展当然首要前提是对Nginx有一定的了解，然而本文并不打算详细阐述Nginx的方方面面，诸如Nginx的安装和各种详细配置等内容都可以在Nginx官网的Document中找到，本文在这里只会概括性描述一些后面可能会用到的原理和概念。

## 准备工作

* 下载nginx：[Nginx1.19.7](http://nginx.org/download/nginx-1.19.7.tar.gz)

* 编译安装LuaJIT：[LuaJIT](http://luajit.org/download.html)

```shell
tar -xzvf LuaJIT-2.0.5.tar.gz
cd LuaJIT-2.0.5
make
make install
```

编译安装过程可能会出现如下错误：

![](https://upload-images.jianshu.io/upload_images/6275607-34d79cda7b23d9fd.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

> Note for OSX: if the MACOSX_DEPLOYMENT_TARGET environment variable is not set, then it's forced to 10.4.

解决办法（设置环境变量）：

```shell
export MACOSX_DEPLOYMENT_TARGET=10.14
```

* 下载ngx_devel_kit模块儿：[ngx_devel_kit](https://github.com/simplresty/ngx_devel_kit/releases)

* 下载lua-nginx-module模块儿：([Releases · openresty/lua-nginx-module · GitHub](https://github.com/openresty/lua-nginx-module/releases))

* **编译安装Nginx**
  
  ```shell
  export LUAJIT_LIB=/usr/local/lib
  export LUAJIT_INC=/usr/local/include/luajit-2.0
  cd nginx-1.19.7
  ./configure \
  --prefix=/Users/bookask/Workspace/flow-control-test/nginx-1.19.7 \
  --add-module=/Users/bookask/Workspace/flow-control-test/nginx/modules/lua-nginx-module-0.10.19 \
  --add-module=/Users/bookask/Workspace/flow-control-test/nginx/modules/ngx_devel_kit-0.3.1
  make
  make install
  ```

* 启动Nginx

```shell
cd /Users/bookask/Workspace/flow-control-test/nginx-1.19.7
./sbin/nginx
```

不出意外的话我们已经启动成功了，我这里出现了其他的问题，如下：

```shell
nginx: [alert] detected a LuaJIT version which is not OpenResty's; many optimizations will be disabled and performance will be compromised (see https://github.com/openresty/luajit2 for OpenResty's LuaJIT or, even better, consider using the OpenResty releases from https://openresty.org/en/download.html)
nginx: [alert] failed to load the 'resty.core' module (https://github.com/openresty/lua-resty-core); ensure you are using an OpenResty release from https://openresty.org/en/download.html (reason: module 'resty.core' not found:
        no field package.preload['resty.core']
        no file './resty/core.lua'
        no file '/usr/local/share/luajit-2.0.5/resty/core.lua'
        no file '/usr/local/share/lua/5.1/resty/core.lua'
        no file '/usr/local/share/lua/5.1/resty/core/init.lua'
        no file './resty/core.so'
        no file '/usr/local/lib/lua/5.1/resty/core.so'
        no file '/usr/local/lib/lua/5.1/loadall.so'
        no file './resty.so'
        no file '/usr/local/lib/lua/5.1/resty.so'
        no file '/usr/local/lib/lua/5.1/loadall.so') in /Users/bookask/Workspace/flow-control-test/nginx-1.19.7/conf/nginx.conf:127
```

此处告诉我们需要resty，那么按照要求，我们继续安装OpenResty：

[OpenResty - 下载](https://openresty.org/cn/download.html)

MacOS通过brew安装：

homebrew安装方式详见官网：[Homebrew官网](https://brew.sh/)

```shell
brew tap openresty/brew
brew install openresty
```

启动openresty：

```shell
To have launchd start openresty/brew/openresty now and restart at login:
  brew services start openresty/brew/openresty
Or, if you don't want/need a background service you can just run:
  openresty
```

安装成功后继续启动nginx：
