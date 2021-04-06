# 编译epc扩展

## 一、PHP扩展安装

```shell
make clean
phpize --clean
phpize
./configure
make && make install
```

* 编译执行`phpize`过程中出现错误:

```shell
grep: /usr/include/php/main/php.h: No such file or directory
grep: /usr/include/php/Zend/zend_modules.h: No such file or directory
grep: /usr/include/php/Zend/zend_extensions.h: No such file or directory
Configuring for:
PHP Api Version:        
Zend Module Api No:     
Zend Extension Api No:  
Cannot find autoconf. Please check your autoconf installation and the
$PHP_AUTOCONF environment variable. Then, rerun this script.
```

执行`brew install autoconf automake libtool`解决

## 二、allinone容器中编译扩展

* 需要安装PHP的开发工具

```shell
apt-get install autoconf automake libtool m4
apt-get install php7.2-dev
```

* 安装node-gyp

```shell
npm install -g node-gyp@6.1.0
```

## 三、PHP编译单独文件

```shell
g++ -o libCommon.o -shared -fPIC  EpcCommon.cpp
```
