# 安装OpenResty映射目录升级

**学堂网站docker启动命令和nginx配置文件：**

1. docker启动命令： /data/www2/etc/80reRunContainers-web.sh  
2. nginx配置文件：映射到镜像（/data/www2/etc/nginx:/etc/nginx)  
3. nginx通过sock文件连接PHP-FPM：映射到镜像（runXt:/var/run）

## 相关项目地址：

* 文泉书局PHP

[项目地址](git@git.bookask.cn:wqxuetang/wqxuetang-shuju-php.git)

* 学堂后端PHP程序

[项目地址](git@git.bookask.cn:wqxuetang/www2.git)

## 替换的nginx

* 书局的nginx名称：shujuNginx

* 学堂的nginx名称：xuetangNginx

## docker编译安装：

```shell
# BUILD COMMAND:
#   docker build . -f zz-allinone-buster.Dockerfile -t allinone
# START COMMAND:
# docker run -d -p 7001:7001 -v $PWD:/data dodosong/node-cn:10   npm i --production --registry=https://registry.npm.taobao.org && npm run start 

FROM debian:buster 
# python:3.8-buster
 

### Change repositories, use dnspod's dns for china, apk update, timezone=Asia/Shanghai
RUN cp /etc/apt/sources.list /etc/apt/sources.list.bak \
    && sed -i 's/deb.debian.org/mirrors.ustc.edu.cn/g' /etc/apt/sources.list \
    && sed -i 's|security.debian.org/debian-security|mirrors.ustc.edu.cn/debian-security|g' /etc/apt/sources.list \
    && apt-get update \
    && apt-get install -y --no-install-recommends \
        apt-transport-https \
        ca-certificates \
        netbase \
        gnupg \
        dirmngr \
        openssh-client \
        curl \
        wget \
        bzip2 \
        unzip \
        perl \
    && mv /etc/localtime /etc/localtime.bak \
    && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
	  && rm -rf /var/lib/apt/lists/*

## Make dir for jre https://github.com/geerlingguy/ansible-role-java/issues/64
RUN mkdir -p /usr/share/man/man1
  
## ----------- nodejs nginx openjdk ----------- ##
RUN curl -sL https://deb.nodesource.com/setup_10.x | bash - \
    && apt-get  update \
    && apt-get  install -y --no-install-recommends \
        nodejs \
        redis \
        imagemagick \
        openjdk-11-jre \
        \
    ; rm -rf /var/lib/apt/lists/* \
    && rm -rf /usr/share/doc/* /usr/share/man/* /usr/share/info/*

     # python3-opencv \

## Make dir for nginx  
RUN mkdir -p /var/run \
  && touch /var/run/nginx.pid \
  && mkdir -p /var/log/nginx \
  && chmod 777 /var/run /var/log /tmp

## ----------- php ----------- ##
ENV PHP_VERSION 7.2

RUN wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg \
    && echo "deb https://packages.sury.org/php/ buster main" > /etc/apt/sources.list.d/php.list \
    && apt-get  update \
    && apt-get  install -y --no-install-recommends \
        php${PHP_VERSION}-dev \
        php${PHP_VERSION} \
        php${PHP_VERSION}-cli \
        php${PHP_VERSION}-fpm \
        php${PHP_VERSION}-common \
        php${PHP_VERSION}-curl  \
        php${PHP_VERSION}-memcache \
        php${PHP_VERSION}-redis \
        php${PHP_VERSION}-igbinary \
        php${PHP_VERSION}-gd \
        php${PHP_VERSION}-json \
        php${PHP_VERSION}-mbstring \
        php${PHP_VERSION}-mysql \
        php${PHP_VERSION}-xml \
        php${PHP_VERSION}-zip \
    ; rm -rf /var/lib/apt/lists/* \
    && rm -rf /usr/share/doc/* /usr/share/man/* /usr/share/info/*


## ----------- build ----------- ##
      # re2c \ 库中版本太低，编译安装2.+版
ENV PHPIZE_DEPS \
      git \
      autoconf \
      dpkg-dev \
      file \
      g++ \
      gcc \
      make \
      automake \
      pkg-config \
      libbz2-dev \
      libc-dev \
      libc6-dev \
      libcurl4-openssl-dev \
      libdb-dev \
      libevent-dev \
      libffi-dev \
      libgdbm-dev \
      libglib2.0-dev \
      libgmp-dev \
      libjpeg-dev \
      libkrb5-dev \
      liblzma-dev \
      libmagickcore-dev \
      libmagickwand-dev \
      libmaxminddb-dev \
      libncurses5-dev \
      libncursesw5-dev \
      libpng-dev \
      libpq-dev \
      libreadline-dev \
      libsqlite3-dev \
      libssl-dev \
      libtool \
      libwebp-dev \
      libxml2-dev \
      libxslt-dev \
      libyaml-dev \
      patch \
      xz-utils \
      zlib1g-dev \
      libpcre3-dev \
      build-essential
    
  ## ----- xunsearch ----- ##
  ## Install xunsearch with scws and scws-dicts ,then modify xs-ctl.sh 
  ### Install  php-pecl-scws and other php extensions \
RUN apt-get update \
	&& apt-get install -y --no-install-recommends $PHPIZE_DEPS \
	&& rm -rf /var/lib/apt/lists/* \
  && mkdir -p /tmp/pkgs \
    && cd /tmp/pkgs \
    && rm -rf xunsearch-* \
    && wget -O - http://www.xunsearch.com/download/xunsearch-full-latest.tar.bz2 | tar -xjf - \
    && cd  xunsearch-full-* \
    && wget https://github.com/libevent/libevent/releases/download/release-2.1.12-stable/libevent-2.1.12-stable.tar.gz \
    && tar -xzf libevent-2.1.12-stable.tar.gz \
    && rm -f packages/libevent-* \
    && tar -cjf packages/libevent-2.1.12-stable.tar.bz2 libevent-2.1.12-stable \
  && sed -i 's/EVENT_VERSION/EVENT__VERSION/g' setup.sh \
    && sh setup.sh --prefix=/usr/local/xunsearch --no-clean \
    && rm -rf /usr/share/doc/* /usr/share/man/* /usr/share/info/* \
  && cd /tmp/pkgs \
    && rm -rf re2c-* \
    && wget https://github.com/skvadrik/re2c/releases/download/2.0.3/re2c-2.0.3.tar.xz \
    && tar -xf re2c*.tar.xz \
    && rm re2c*.tar.xz \
    && cd re2c* \
    && ./configure \
    && make && make install \
    && cd - && rm -rf re2c* \
    && cd  xunsearch-full-*/scws-*/phpext \
    &&  phpize \
    && ./configure --with-scws=/usr/local/xunsearch/ \
    && make  \
    && make install \
    && { \
            echo "extension=scws.so"; \
            echo "scws.default.charset = utf8"; \
            echo "scws.default.fpath = /usr/local/scws/etc"; \
      } | tee /etc/php/${PHP_VERSION}/mods-available/scws.ini \
    && ln -sf /etc/php/${PHP_VERSION}/mods-available/scws.ini  /etc/php/${PHP_VERSION}/cli/conf.d/40-scws.ini \
    && ln -sf /etc/php/${PHP_VERSION}/mods-available/scws.ini  /etc/php/${PHP_VERSION}/fpm/conf.d/40-scws.ini \
    && php -m |grep scws \
    && cd /tmp/pkgs \
    && rm -rf xunsearch-* \
  && cd /tmp/pkgs \
    && rm -rf openresty-* \
    && wget https://openresty.org/download/openresty-1.19.3.1.tar.gz \
    && tar -xzf openresty*.tar.gz \
    && rm openresty*.tar.gz \
    && cd openresty* \
    && ./configure --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock \
    && make && make install \
    && cd - && rm -rf openresty* \
  && apt-get remove -y $PHPIZE_DEPS \
  && apt-get autoremove -y \
  && rm -rf /var/lib/apt/lists/*

# Add additional binaries into PATH for convenience
ENV PATH=$PATH:/usr/local/openresty/luajit/bin:/usr/local/openresty/nginx/sbin:/usr/local/openresty/bin

# Add LuaRocks paths
# If OpenResty changes, these may need updating:
#    /usr/local/openresty/bin/resty -e 'print(package.path)'
#    /usr/local/openresty/bin/resty -e 'print(package.cpath)'
ENV LUA_PATH="/usr/local/openresty/site/lualib/?.ljbc;/usr/local/openresty/site/lualib/?/init.ljbc;/usr/local/openresty/lualib/?.ljbc;/usr/local/openresty/lualib/?/init.ljbc;/usr/local/openresty/site/lualib/?.lua;/usr/local/openresty/site/lualib/?/init.lua;/usr/local/openresty/lualib/?.lua;/usr/local/openresty/lualib/?/init.lua;./?.lua;/usr/local/openresty/luajit/share/luajit-2.1.0-beta3/?.lua;/usr/local/share/lua/5.1/?.lua;/usr/local/share/lua/5.1/?/init.lua;/usr/local/openresty/luajit/share/lua/5.1/?.lua;/usr/local/openresty/luajit/share/lua/5.1/?/init.lua"

ENV LUA_CPATH="/usr/local/openresty/site/lualib/?.so;/usr/local/openresty/lualib/?.so;./?.so;/usr/local/lib/lua/5.1/?.so;/usr/local/openresty/luajit/lib/lua/5.1/?.so;/usr/local/lib/lua/5.1/loadall.so;/usr/local/openresty/luajit/lib/lua/5.1/?.so"
 
RUN opm get hamishforbes/lua-resty-iputils

## make dir for bookask and wqxuetang
RUN set -ex \
  && mkdir -p /var/code \
  && mkdir /data \
  ### create shell for nginx+php-fpm
  && mkdir -p /root/bin \
  && cd /root/bin \
  && { \
            echo "#!/bin/sh"; \
            echo "crond"; \
            echo "nginx";  \
            echo "php-fpm7 -F";  \
      } | tee init.sh \
  && chmod 755 init.sh \
  ### create shell for wqxuetang-offline
  && mkdir -p /root/bin \
  && cd /root/bin \
  && { \
            echo "#!/bin/sh"; \
            echo "echo \"127.0.0.1 lib-offline.wqxuetang.com\" >> /etc/hosts"; \
            echo "bash ./init.sh"; \
      } | tee offline.sh \
  && chmod 755 offline.sh \
  ### create shell for xunsearch \
  && echo '' >> /usr/local/xunsearch/bin/xs-ctl.sh \
  && echo 'tail -f /dev/null' >> /usr/local/xunsearch/bin/xs-ctl.sh \
  && { \
        echo "#!/bin/sh"; \
        echo "rm -f /usr/local/xunsearch/tmp/pid.*"; \
        echo "echo -n > /usr/local/xunsearch/tmp/docker.log"; \
        echo "/usr/local/xunsearch/bin/xs-indexd -l /usr/local/xunsearch/tmp/docker.log -k start" ; \
        echo "sleep 1" ; \
        echo "/usr/local/xunsearch/bin/xs-searchd -l /usr/local/xunsearch/tmp/docker.log -k start";  \
        echo "sleep 1" ; \
        echo "tail -f /usr/local/xunsearch/tmp/docker.log";  \
    } | tee /usr/local/xunsearch/bin/xs-docker.sh \
  && chmod 755 /usr/local/xunsearch/bin/xs-docker.sh \
  && mkdir -p /root/bin \
  && ln -s /usr/local/xunsearch/bin/xs-docker.sh  /root/bin/xs-docker.sh
  
## create user who use the same uid with code's directories \
RUN set -ex \ 
  && groupmod -g 82 www-data \
  && usermod -u 82 www-data \
  && chmod 777 -R   /var/lock /var/log /var/run /var/tmp  /var/code 
  
  # \
  # && chown -R www-data:www-data /var/www var/tmp var/tmp


# ENV PATH=/usr/lib/jvm/default-jvm/jre/bin:$PATH
WORKDIR "/var/code"

# ENTRYPOINT ["sh"]
# CMD ["/usr/local/xunsearch/bin/xs-docker.sh"]
# CMD ["openresty", "-p", "/etc/nginx", "-c", "nginx.conf", "-g", "daemon off;"]
```

* 编译后，启动增加`-p`参数方可改变工作目录。否则为`/usr/local/openresty/nginx`目录

```shell
docker run --name xxx -d allinone:v1.1 openresty -p xxx -g "daemon off;"
```


