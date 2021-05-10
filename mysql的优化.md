# mysql的优化

## 一、索引优化-方案

+ 表记录很少不需要创建索引

+ 一个表的索引个数不能过多

+ 频繁更新的字段不建议作为索引

+ 区分度低的字段，不建议建索引

+ 主键索引建议使用整型或者长整型类型

+ 进量创建组合索引，而不是单列索引

+ 开启索引条件下推 `show variables like 'optimizer_switch';`

## 二、SQL语句优化

+ Limit优化（深翻页过程中可以增加Where中id大于多少来进行limit控制）

+ 小表驱动大表

+ 避免全表扫描

+ 关联字段都建立索引

+ WHERE条件中进量不要使用not in、<>（会导致索引失效）

## 三、MySQL数据库配置优化

+ 缓冲池大小`innodb_buffer_pool_size`推荐值为物理内存的50%～80%

+ Redo log刷盘策略`innodb_flush_log_at_trx_commit=1`默认是1 ，进量不要更改，避免丢失数据

+ Bin log刷盘策略`sync_binlog=1`妹提交1次十五同步写到磁盘中，可以设置为n

+ 脏页过多触发刷盘百分比`innodb_max_dirty_pages_pct=30`，脏页占`innodb_buffer_poll_size`的比例时，触发刷脏页到磁盘，推荐值为25%～50%

+ 后台进程最大IO性能指标`innodb_io_capacity=200`默认200，如果SSD，调整为5000～20000

## 四、操作系统优化

+ 内核参数优化

+ + CentOS系统针对mysql的参数优化
  
  + + 内核相关参数（/etc/sysctl.conf）
      
      以下参数可以直接放到sysctl.conf文件的末尾:
      
      1. 增加监听队列上限：
      
      ```shell
      net.core.somaxconn=65535
      net.core.netdev_max_backlog=65535
      net.ipv4.tcp_max_syn_backlog=65535
      ```
      
      2. 加快TCP连接的回收：
      
      ```shell
      net.ipv4.tcp_fin_timeout=10
      net.ipv4.tcp_tw_reuse=1
      net.ipv4.tcp_tw_recycle=1
      
      ```
      
      
      
      3. TCP连接接受和发送缓冲区大小的默认值和最大值：
      
      ```shell
      net.core.wmem_default=87380
      net.core.wmem_max=16777216
      net.core.rmem_default=87380
      net.core.rmem_max=16777216
      ```
      
      
      
      4. 减少失效连接所占用的TCP资源的数量，加快资源回收的效率：
      
      ```shell
      net.ipv4.tcp_keepalive_time=120
      net.ipv4.tcp_keepalive_intvl=30
      net.ipv4.tcp_keepalive_probes=3
      ```
      
      
      
      5. 单个共享内存段的最大值：
      
      ```shell
      kernel.shmmax=4294967295
      ```

                        这个参数应该设置的足够大，以便能在一个共享内存段下容纳整个的Innodb缓冲池的大小。

                        这个值的大小对于64为linux系统，可取的最大值为（物理内存值-1）byte，建议值为大于物理内存的一半，一般取值大于Innodb缓冲池的大小即可。

                        6. 控制换出运行时内存的相对权重：

                          `vm.swappiness=0`

                        这个参数当内存不足时会对性能产生比较明显的影响。（设置为0，表示Linux内核虚拟内存完全被占用，才会要使用交换区。）



+ 磁盘调度策略
1. cfq（完全公平队列策略，Linux2.6.18之后内核的系统默认策略）
   
   该模式按进程创建多个队列，各个进程发来的IO请求会被cfq以轮循方式处理，对每个IO请求都是公平的。该策略适合离散读的应用。

2. deadline（截止时间调度策略）

deadline，包含读和写两个队列，确保在一个截止时间内服务请求（截止时间时可调整的），而默认读期限短于写期限。这样就防止了写操作因为不能被读取而饿死的现象，deadline对数据库类应用时最好的选择。

3. noop（电梯式调度策略）
   
   noop只实现一个简单的FIFO队列，倾向饿死读而利于写，因此noop对于闪存设备、RAM及嵌入式系统是最好的选择。

4. anticipatory（预料I/O调度策略）
   
   本质上与deadline策略一样，但在最后一次读操作后，要等待6ms，才能继续进行对其它I/O请求进行调度。它会在每个6ms中插入新的I/O操作，合并写入流，用写入延时换取最大的写入吞吐量。anticipatory适合与写入较多的环境，比如文件服务器。该策略对数据库环境表现很差。
   
   查看调度策略的方法：
   
   `cat /sys/block/devname/queue/scheduler`
   
   修改调度策略的方法：
   
   `echo <schedulername> /sys/block/devname/queue/scheduler`
+ 增加资源限制

打开文件数的限制以下参数可以直接放到（/etc/security/limit.conf）文件的末尾：

`* soft nofile 65535`

`* hard nofile 65535`

*：表示对所有用户有效

soft：表示当前系统生效的设置（soft不能大于hard）

hard：表明系统中所能设定的最大值

nofile：表示所限制的资源时打开文件的最大数目

65535：限制的数量

以上两行配置将可打开的文件数量增加到65535个，以保证打开足够多的文件句柄。

> 注意：这个文件的修改需要重启系统才能生效。



## 五、服务器硬件优化

+ 高频率的内存

+ + 频率不能高于主板的支持

+ 提升网络带宽

+ + 提升网络带宽

+ 使用SSD高速磁盘

+ + 内使用SSD高速磁盘的情况下进量选择SSD

+ 提升CPU性能等。

+ + 对于数据库兵法比较高的场景，CPU的数量比频率重要。
  
  + 对于CPU密集型长江和频繁执行复杂SQL的场景，CPU的频率越高越好

## 六、b树

1. 每个节点的容量是16K固定的

2. 度=16K/每行数据的大小（取决于表结构的定义）

3. 每个数据左右两边都有一个指针。

4. 每行数据是
   
   数据8字节+指针4字节*2=16字节
   
   16K/16字节=1024
   
   两层节点（b树的高度为2的时候）
   
   1024*1024 = 大约100w的数据
   
   > 问题：
   > 
   > 1. 范围查询时性能不好。
   > 
   > 2. 每个数据其实是一个数据行的综合。如果数据行变大，节点的度变小，树的高度变高，产生io的次数变多，性能下降。

## 七、b+树（大部分索引的选择）

1. 包含b树的所有特点。

2. 所有的数据都保存在叶子结点中，中间节点只保存主键。

3. 叶子结点中的数据是有序的，并且节点和节点之间有双向指针，方便做范围查询。


