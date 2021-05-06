# Elasticsearch环境搭建

## 一、环境准备

* Elasticsearch

* * 下载Linux版本
  
  ```shell
  wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.12.0-linux-x86_64.tar.gz
  wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.12.0-linux-x86_64.tar.gz.sha512
  shasum -a 512 -c elasticsearch-7.12.0-linux-x86_64.tar.gz.sha512 
  tar -xzf elasticsearch-7.12.0-linux-x86_64.tar.gz
  cd elasticsearch-7.12.0/ 
  ```

* * 下载Mac版本
  
  ```shell
  wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.12.0-darwin-x86_64.tar.gz
  wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.12.0-darwin-x86_64.tar.gz.sha512
  shasum -a 512 -c elasticsearch-7.12.0-darwin-x86_64.tar.gz.sha512 
  tar -xzf elasticsearch-7.12.0-darwin-x86_64.tar.gz
  cd elasticsearch-7.12.0/
  ```

* Kibana

* * 下载Linux版本
  
  ```shell
  curl -O https://artifacts.elastic.co/downloads/kibana/kibana-7.12.0-linux-x86_64.tar.gz
  curl https://artifacts.elastic.co/downloads/kibana/kibana-7.12.0-linux-x86_64.tar.gz.sha512 | shasum -a 512 -c - 
  tar -xzf kibana-7.12.0-linux-x86_64.tar.gz
  cd kibana-7.12.0-linux-x86_64/
  ```

* * 下载Mac版本
  
  ```shell
  curl -O https://artifacts.elastic.co/downloads/kibana/kibana-7.12.0-darwin-x86_64.tar.gz
  curl https://artifacts.elastic.co/downloads/kibana/kibana-7.12.0-darwin-x86_64.tar.gz.sha512 | shasum -a 512 -c - 
  tar -xzf kibana-7.12.0-darwin-x86_64.tar.gz
  cd kibana-7.12.0-darwin-x86_64/
  ```

**macOS 无法启动Kinaba的Gatekeepers warnings解决方案**

```shell
xattr -d -r com.apple.quarantine <archive-or-directory>
```

* Canal数据同步ES

* * 下载Linux版本
  
  ```shell
  # Admin管理后台
  wget https://github.com/alibaba/canal/releases/download/canal-1.1.5-alpha-2/canal.admin-1.1.5-SNAPSHOT.tar.gz
  # 发布包
  wget https://github.com/alibaba/canal/releases/download/canal-1.1.5-alpha-2/canal.deployer-1.1.5-SNAPSHOT.tar.gz
  # SyncES包
  wget https://github.com/alibaba/canal/releases/download/canal-1.1.5-alpha-2/canal.adapter-1.1.5-SNAPSHOT.tar.gz
  ```

* Elasticsearch可视化工具（可以不下载）

* * 下载ElasticHD
  
  [GitHub - 360EntSecGroup-Skylar/ElasticHD: Elasticsearch 可视化DashBoard, 支持Es监控、实时搜索，Index template快捷替换修改，索引列表信息查看， SQL converts to DSL等](https://github.com/360EntSecGroup-Skylar/ElasticHD)

> **下载这些包，准备Java8或更高版本的环境，然后依次进行启动，由于jdk版本的不同，start.sh中的启动jvm参数需要做相应的调整。**

## 二、启动相关服务（此处仅介绍Mac版本）

+ 启动Elasticsearch

```shell
# 创建服务目录
mkdir ES
cd ES
# 下载Mac包
curl -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.12.0-darwin-x86_64.tar.gz
curl -O https://artifacts.elastic.co/downloads/kibana/kibana-7.12.0-darwin-x86_64.tar.gz
curl -O https://github.com/alibaba/canal/releases/download/canal-1.1.5-alpha-2/canal.admin-1.1.5-SNAPSHOT.tar.gz
curl -O https://github.com/alibaba/canal/releases/download/canal-1.1.5-alpha-2/canal.deployer-1.1.5-SNAPSHOT.tar.gz
curl -O https://github.com/alibaba/canal/releases/download/canal-1.1.5-alpha-2/canal.adapter-1.1.5-SNAPSHOT.tar.gz
# 解压
tar -xzvf elasticsearch-7.12.0-darwin-x86_64.tar.gz
tar -xzvf kibana-7.12.0-darwin-x86_64.tar.gz
tar -xzvf canal.admin-1.1.5-SNAPSHOT.tar.gz
tar -xzvf canal.deployer-1.1.5-SNAPSHOT.tar.gz
tar -xzvf canal.adapter-1.1.5-SNAPSHOT.tar.gz
# 启动ES
sh elasticsearch-7.12.0/bin/elasticsearch
# 启动Kinaba 成功后访问：http://localhost:5601
xattr -d -r com.apple.quarantine kibana-7.12.0-darwin-x86_64
sh kibana-7.12.0-darwin-x86_64/bin/kibana

# 启动、关闭Canal Admin管理
sh canal.admin-1.1.5-SNAPSHOT/bin/startup.sh
# 在启动命令中使用参数，指定配置：
sh canal.admin-1.1.5-SNAPSHOT/bin/startup.sh local
sh canal.admin-1.1.5-SNAPSHOT/bin/stop.sh
# 启动、关闭Canal Deployer发布
sh canal.deployer-1.1.5-SNAPSHOT/bin/startup.sh
sh canal.deployer-1.1.5-SNAPSHOT/bin/stop.sh
# 启动、关闭Canal Adapter 同步ES
sh canal.adapter-1.1.5-SNAPSHOT/bin/startup.sh
sh canal.adapter-1.1.5-SNAPSHOT/bin/stop.sh
```

> **启动Canal后台的时候需要修改startup.sh文件的jvm参数，由于jdk版本的不同，相关jvm参数已经不再支持，需要修正或删除**

* 启动Canal Admin后访问`localhost:8089`默认密码：admin/123456

## 三、修改配置，同步mysql到ES

* Canal Adapter配置文件修改

```shell
# conf/es7下新增canal_user.yml
dataSourceKey: defaultDS
destination: example
groupId: g1
esMapping:
  _index: canal_user
  _id: _id
#  upsert: true
#  pk: id
  sql: "select a.id as _id, a.name, a.username, a.roles from canal_manager.canal_user as a"
#  objFields:
#    _labels: array:;
  # etlCondition: "where a.c_time>={}"
  commitBatch: 3000
```

```shell
# 修改conf/application.yml 增加srcDataSources和outerAdapters的es7
server:
  port: 8081
spring:
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8
    default-property-inclusion: non_null

canal.conf:
  mode: tcp #tcp kafka rocketMQ rabbitMQ
  flatMessage: true
  zookeeperHosts:
  syncBatchSize: 1000
  retries: 0
  timeout:
  accessKey:
  secretKey:
  consumerProperties:
    # canal tcp consumer
    canal.tcp.server.host: 127.0.0.1:11111
    canal.tcp.zookeeper.hosts:
    canal.tcp.batch.size: 500
    canal.tcp.username:
    canal.tcp.password:
    # kafka consumer
    kafka.bootstrap.servers: 127.0.0.1:9092
    kafka.enable.auto.commit: false
    kafka.auto.commit.interval.ms: 1000
    kafka.auto.offset.reset: latest
    kafka.request.timeout.ms: 40000
    kafka.session.timeout.ms: 30000
    kafka.isolation.level: read_committed
    kafka.max.poll.records: 1000
    # rocketMQ consumer
    rocketmq.namespace:
    rocketmq.namesrv.addr: 127.0.0.1:9876
    rocketmq.batch.size: 1000
    rocketmq.enable.message.trace: false
    rocketmq.customized.trace.topic:
    rocketmq.access.channel:
    rocketmq.subscribe.filter:
    # rabbitMQ consumer
    rabbitmq.host:
    rabbitmq.virtual.host:
    rabbitmq.username:
    rabbitmq.password:
    rabbitmq.resource.ownerId:

  srcDataSources:
    defaultDS:
      url: jdbc:mysql://127.0.0.1:3306/canal_manager?useUnicode=true&characterEncoding=UTF-8&useSSL=false
      username: canal
      password: canal
  canalAdapters:
  - instance: example # canal instance Name or mq topic name
    groups:
    - groupId: g1
      outerAdapters:
      - name: logger
#      - name: rdb
#        key: mysql1
#        properties:
#          jdbc.driverClassName: com.mysql.jdbc.Driver
#          jdbc.url: jdbc:mysql://127.0.0.1:3306/mytest2?useUnicode=true
#          jdbc.username: root
#          jdbc.password: 121212
#      - name: rdb
#        key: oracle1
#        properties:
#          jdbc.driverClassName: oracle.jdbc.OracleDriver
#          jdbc.url: jdbc:oracle:thin:@localhost:49161:XE
#          jdbc.username: mytest
#          jdbc.password: m121212
#      - name: rdb
#        key: postgres1
#        properties:
#          jdbc.driverClassName: org.postgresql.Driver
#          jdbc.url: jdbc:postgresql://localhost:5432/postgres
#          jdbc.username: postgres
#          jdbc.password: 121212
#          threads: 1
#          commitSize: 3000
#      - name: hbase
#        properties:
#          hbase.zookeeper.quorum: 127.0.0.1
#          hbase.zookeeper.property.clientPort: 2181
#          zookeeper.znode.parent: /hbase
      - name: es7
        hosts: 127.0.0.1:9300 # 127.0.0.1:9200 for rest mode
        properties:
          mode: transport # or rest
          # security.auth: test:123456 #  only used for rest mode
          cluster.name: elasticsearch
#        - name: kudu
#          key: kudu
#          properties:
#            kudu.master.address: 127.0.0.1 # ',' split multi address
```

## 四、创建索引

* 创建canal_user的索引

```shell
# 新增索引
PUT /canal_user
{
  "settings": {
    "number_of_shards": 1
  },
  "mappings": {
      "properties": {
        "username": {"type": "text"},
        "name": {"type": "text"},
        "roles": {"type": "text"}
      }
  }
}
```

* 新增原始数据

```shell
# 增加数据
POST /canal_user/_doc/2
{
  "username": "sjj",
  "name": "SJJ Canal Manager",
  "roles": "admin"
}
```

**ES同步的过程中只存在INSERT和UPDATE，所以旧数据需要先新增到索引库中**

## 五、测试

* 开启上述相关服务后，使用`canal.admin-1.1.5-SNAPSHOT/conf/canal_manager.sql`文件初始化数据库。创建`canal` 的用户，密码为`canal`。

* 在数据库`canal_user`表新增一条数据，此时可以看到adapter的日志中出现DML日志，并讲结果新增到了ES中

```shell
# 新增
2021-04-06 14:05:10.886 [pool-1-thread-1] INFO  c.a.o.canal.client.adapter.logger.LoggerAdapterExample - DML: {"data":[{"id":3,"username":"zhangsan","password":"123456","name":"go back","roles":"admin","introduction":null,"avatar":null,"creation_date":1617689110000}],"database":"canal_manager","destination":"example","es":1617689110000,"groupId":"g1","isDdl":false,"old":null,"pkNames":["id"],"sql":"","table":"canal_user","ts":1617689110886,"type":"INSERT"}
2021-04-06 14:05:10.888 [pool-1-thread-1] DEBUG c.a.o.canal.client.adapter.es.core.service.ESSyncService - DML: {"data":[{"id":3,"username":"zhangsan","password":"123456","name":"go back","roles":"admin","introduction":null,"avatar":null,"creation_date":1617689110000}],"database":"canal_manager","destination":"example","es":1617689110000,"groupId":"g1","isDdl":false,"old":null,"pkNames":["id"],"sql":"","table":"canal_user","ts":1617689110886,"type":"INSERT"} 
Affected indexes: canal_user 
# 删除
2021-04-06 14:04:57.224 [pool-1-thread-1] INFO  c.a.o.canal.client.adapter.logger.LoggerAdapterExample - DML: {"data":[{"id":3,"username":"test","password":"6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9","name":"Test","roles":"admin","introduction":null,"avatar":null,"creation_date":1617444981000}],"database":"canal_manager","destination":"example","es":1617689097000,"groupId":"g1","isDdl":false,"old":null,"pkNames":["id"],"sql":"","table":"canal_user","ts":1617689097224,"type":"DELETE"}
2021-04-06 14:04:57.226 [pool-1-thread-1] DEBUG c.a.o.canal.client.adapter.es.core.service.ESSyncService - DML: {"data":[{"id":3,"username":"test","password":"6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9","name":"Test","roles":"admin","introduction":null,"avatar":null,"creation_date":1617444981000}],"database":"canal_manager","destination":"example","es":1617689097000,"groupId":"g1","isDdl":false,"old":null,"pkNames":["id"],"sql":"","table":"canal_user","ts":1617689097224,"type":"DELETE"} 
Affected indexes: canal_user 
# 修改
2021-04-06 14:09:51.272 [pool-1-thread-1] INFO  c.a.o.canal.client.adapter.logger.LoggerAdapterExample - DML: {"data":[{"id":3,"username":"zhangsan","password":"123456","name":"go back alone","roles":"admin","introduction":null,"avatar":null,"creation_date":1617689390000}],"database":"canal_manager","destination":"example","es":1617689390000,"groupId":"g1","isDdl":false,"old":[{"name":"go back","creation_date":1617689110000}],"pkNames":["id"],"sql":"","table":"canal_user","ts":1617689391270,"type":"UPDATE"}
2021-04-06 14:09:51.287 [pool-1-thread-1] DEBUG c.a.o.canal.client.adapter.es.core.service.ESSyncService - DML: {"data":[{"id":3,"username":"zhangsan","password":"123456","name":"go back alone","roles":"admin","introduction":null,"avatar":null,"creation_date":1617689390000}],"database":"canal_manager","destination":"example","es":1617689390000,"groupId":"g1","isDdl":false,"old":[{"name":"go back","creation_date":1617689110000}],"pkNames":["id"],"sql":"","table":"canal_user","ts":1617689391270,"type":"UPDATE"} 
Affected indexes: canal_user 
```

* ES中的搜索结果如下：

```shell
# 搜索canal_user的全部索引
GET /canal_user/_search
{
  "query": {
    "match_all": {}
  }
}

# 结果如下
{
  "took" : 71,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "canal_user",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : 1.0,
        "_source" : {
          "name" : "go back alone",
          "username" : "zhangsan",
          "roles" : "admin"
        }
      }
    ]
  }
}
```

可见我们的数据已经新增到了ES中，并且可以做实时同步。

> **特别注意**：当前的 canal 支持源端 MySQL 版本包括 5.1.x , 5.5.x , 5.6.x , 5.7.x , 8.0.x
> 
> 且需要开启binlog日志，日志格式为ROW
> 
> ```shell
> show variables like 'log_bin';
> show variables like 'binlog_format';
> show variables like 'log_bin_trust_function_creators'
> 
> set global log_bin_trust_function_creators=1;
> 
> # 修改my.conf
> [mysqld]  
> log-bin=mysql-bin #添加这一行就ok  
> binlog-format=ROW #选择row模式  
> server_id=1 #配置mysql replaction需要定义，不能和canal的slaveId重复 
> ```

> [JVM8启动参数参考地址](https://www.cnblogs.com/qlqwjy/p/8037797.html)
> 
> [JVM11启动参数参考地址](https://juejin.cn/post/6844904073481699336)
> 
> [Mysql同步ES](https://segmentfault.com/a/1190000019066098)

## 六、安装IK分词器

```shell
./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.12.0/elasticsearch-analysis-ik-7.12.0.zip
```

## 七、docker中环境问题

* 外网访问需要修改elasticsearch的`network.host: 0.0.0.0` ，同时修改`cluster.initial_master_nodes: ["elasticsearch"]` 此时会报内存不够的异常，使用如下参数修改：

```shell
sysctl -w vm.max_map_count=262144 
sysctl -a|grep vm.max_map_count # 设置完成后可以通过该命令查看是否设置成功
```

此时在容器中会报read-only file错误，在启动的时候增加`--privileged` 参数解决问题。

* kibana外网访问，修改`server.host: "0.0.0.0"` 

参考地址：[来自于同事的docker使用过程中的一些记录 - js.yeyong - 博客园](https://www.cnblogs.com/yeyong/p/12518168.html)

## 八、分词安装

```shell
# IK分词
bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.3.0/elasticsearch-analysis-ik-6.3.0.zip
# PinYin分词
bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-pinyin/releases/download/v6.8.14/elasticsearch-analysis-pinyin-6.8.14.zip
```

* ik分词的ik_max_word存在问题

```shell
startOffset must be non-negative, and endOffset must be >= startOffset, and offsets must not go backwards startOffset=15,endOffset=16,lastStartOffset=16 for field 'title'"
"properties" : {
"title" : {
"type" : "text",
"analyzer" : "ik_max_word",
"fields" : {
"keyword" : {
"type" : "keyword"
}
}
}
}

elasticsearch 和ik版本皆为6.8.1
```

> **解决方案**
> 
> 目前不使用`ik_max_word` ，后期看是否需要更新
> 
> 参考地址：[ik分词器ik_max_word startOffset must be non-negative, and endOffset must be &gt;= startOffset, and offsets must not go backwards startOffset=15,endOffset=16,lastStartOffset=16 for field &#39;title&#39;&quot; · Issue #714 · medcl/elasticsearch-analysis-ik · GitHub](https://github.com/medcl/elasticsearch-analysis-ik/issues/714)
