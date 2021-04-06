# Elasticsearch学习

## 一、下载安装Elasticsearch

[下载地址](https://www.elastic.co/cn/downloads/elasticsearch)

```shell
# 下载
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.12.0-darwin-x86_64.tar.gz
# 解压
tar -xzvf elasticsearch-7.12.0-darwin-x86_64.tar.gz
# 启动elasticsearch
cd elasticsearch-7.12.0-darwin-x86_64
./bin/elasticsearch
```

```shell
bookask@SJJ-MBP elasticsearch-7.11.1 % bin/elasticsearch --help
Starts Elasticsearch

Option                Description                                               
------                -----------                                               
-E <KeyValuePair>     Configure a setting                                       
-V, --version         Prints Elasticsearch version information and exits        
-d, --daemonize       Starts Elasticsearch in the background                    
-h, --help            Show help                                                 
-p, --pidfile <Path>  Creates a pid file in the specified path on start         
-q, --quiet           Turns off standard output/error streams logging in console
-s, --silent          Show minimal output                                       
-v, --verbose         Show verbose output
```

* 访问`http://localhost:9200`

## 二、安装ElasticHD来进行可视化访问

[下载地址](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2F360EntSecGroup-Skylar%2FElasticHD)

**在Releases中找到对应的版本进行下载**

```shell
# 解压
unzip elasticHD_darwin_adm64.zip
# 查看权限
ls -l
# 赋予执行权限
chmod 755 ElasticHD
# 运行，IP可使用127.0.0.1 或者本机ip，9800是默认端口可更换
ElasticHD -p 127.0.0.1:9800
```

* 通过修改`config/elasticsearch.yml`中的`network.host=192.168.3.174`和`http.port=9200`来更改启动的配置地址

## 三、安装配置Kibana

[下载地址](https://artifacts.elastic.co/downloads/kibana/kibana-7.12.0-darwin-x86_64.tar.gz)

```shell
# 解压
tar -xzvf kibana-7.12.0-darwin-x86_64.tar.gz
# 启动
cd kibana-7.12.0-darwin-x86_64
bin/kibnana
```

* 访问`http://localhost:5601`

## 四、数据初始化

* 阿里canal

```shell
# 下载
wget https://github.com/alibaba/canal/releases/download/canal-1.1.4/canal.deployer-1.1.4.tar.gz
# 解压
tar -xzvf canal.deployer-1.1.4.tar.gz
# 启动
cd canal.deployer-1.1.4
bin/startup.sh
```

* 创建用户，以便启动canal

```shell
create user 'canal'@'localhost' IDENTIFIED BY 'canal';
create user 'canal'@'%' IDENTIFIED BY 'canal';
grant all privileges on *.* to 'canal'@'localhost' identified by 'canal';
grant all privileges on *.* to 'canal'@'%' identified by 'canal';
flush privileges;
```

* * 启动报错，查看`jvm.options`，JDK8和JDK11不同
  
  * 下载[canal.adapter-1.1.4.tar.gz](https://github.com/alibaba/canal/releases/download/canal-1.1.4/canal.adapter-1.1.4.tar.gz)
  
  * 解压`tar -xzvf canal.adapter-1.1.4.tar.gz`
  
  * 
