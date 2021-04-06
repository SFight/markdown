# API网关流控

## 一、简介

API网关流控是为了应对不断增大的网络流量安全风险。流量通过客户端进入到分布式系统的每一环中：Nginx、API网关、RPC服务、MQ消息应用中心、数据库等。瞬间的大流量对于整个系统的冲击，将会影响到整个系统服务的能否正常运行。

API网关流控常见的有熔断、限流、整流三种方式。也有说单机流控、全局流控、动态流控的。

阿里的API网关流量控制从API分组、API、用户、APP四个维度进行了流量控制，当流量超过设置阀值的时候，API网关直接返回错误信息（错误码403，明确给用户返回因流控被拒的错误信息）给请求者，从而保护后端服务不被攻击，同时支持精准流控和实时生效的功能。

## 二、流控策略

#### 1、熔断

字面意思很好理解，但是熔断分为两层，服务端熔断和客户端熔断。

服务端熔断：后端服务的upstream的一个成员或者为服务的一个服务实例相应超时，这时候API网关根据http相应自动对这一个成员或者实例进行限流，时候来的流量发送到其他的成员或者实例。

客户端熔断：根据某一刻混搭或者某一组客户端行为特征，当流量激增并且与业务行为的典型特征不同，这时候API网关根据客户端IP、用户分布等特征进行熔断。有这样一个案例，某APP做新用户注册的运营活动，只要注册就送10元理财金，并且可以提现。这样的信息被羊毛党盯上后，就会快速跟进，第一次试点薅了几次羊毛后，第二次就成批的注册用户，然后绑卡进行提现。这时候就需要用最快的速度进行客户端熔断。客户端熔断其实还有一个更明显的事件，就是前几年双十一实际上支付宝都是对淘系以外的商家进行直接熔断来的。实际就是保障自己的支付服务的正常进行，同时打压一下小电商的双11分流。

网关熔断：

![](https://pics2.baidu.com/feed/32fa828ba61ea8d3671635d24c72f948241f58ab.png?token=0af2c7572b904a0163d0f2b8ebd7c947)

#### 2、限流

限流字面意思很好理解，不存在服务端限流，主要是针对客户端进行限流。限流的算法很多，比如漏桶（Leaky Bucket）算法、令牌桶算法（Token Bucket）。

限流主要是提供降级服务，限流的算法一般较之熔断来说要更为复杂。熔断一般都是大事件，自动熔断一般较少，基本上都是人工触发熔断，也就是由后台监控之后综合业务表现来发出熔断指令，而限流一般不需要业务参与，回根据各种指标进行技术限流，比如：API相应报文大小、单位时间内的最大/平均请求次数，后台服务根据监控主动限流，针对特定时间段限流、针对特定IP限流，根据用户分组（VIP/非VIP）限流等。

大多的场景都需要上述限流条件的组合，所以对于API网关而言，除了要提供基本的插件方式允许用户自定义限流条件，还要能对限流条件进行组合，排列优先级，判断互斥等。除了技术储备，还要对业务熟练，比如用户分组就是一个与业务场景结合比较紧密的条件，很难从技术上直接进行定义。

网关限流：

![](https://pics0.baidu.com/feed/d31b0ef41bd5ad6ec0f2446c5ab3f0ddb7fd3caf.png?token=f25dc1939f467606c35e5ff94798d618)

#### 3、整流

相对于熔断、限流两种服务降级的手段，熔断比较硬一些，限流又稍微软一些，但是整流相对限流而言，要更软。

整流就是流量整形，或者说叫流量缓冲。流量整形的源出处应该是网络概念里面的术语，有流量监管和流量整形。这个概念在我们生活中有更加形象的例子（高速公路收费口）：

![](https://pics6.baidu.com/feed/b03533fa828ba61e504beb139b4c5e0c314e59e0.jpeg?token=60e57ee1abea63abe1b00f9c0ac540e1)

实际上API网关的整流思路与高速公路的收费口一致，就是无论你进来的流量多少，我都可以把入口流量按照一定的热度调整成后端服务可以承受的速度，这样就不会像熔断和限流那样给到用户非常硬的相应回复，而是一种排队等待。流量整形经常发生在一些特定的业务场景中，比如秒杀。

网关整流：

![](https://pics3.baidu.com/feed/d009b3de9c82d158c56390515a72d0debe3e42e8.png?token=00f744381800c8287b6aa4200681aa2a)

#### 4、API分组流控

即每个API分组在建立的时候，默认设置分组的QPS微500，指该分组下的所有API的QPS之和微500。

#### 5、API流量控制

指单位时间内的调用次数控制。

单位时间：可以是秒、分钟、小时、天四个粒度的流量控制。

秒、分钟：适合配置在调用频繁，请求量大的API上，防攻击，因为不能长时间中断调用，所以单位时间不能设置太长。

小时、天：适合配置在敏感型API上，比如发短信API、获取验证码API等，闲置调用频率。

#### 6、特殊流控策略（APP、用户）

支持给用户和APP设置特殊流控。

主要用于对用户和APP分层，API提供者可以设置API上某个用户（或某个APP）的单独流控。

## 三、流控测试

#### 1、Nginx限流测试

* **基本限流配置**

```shell
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=1r/s;
server {
    listen       9999;
    server_name  localhost;

    location / {
        limit_req zone=mylimit;
        proxy_pass http://docker.for.mac.host.internal:7001/;
    }
}
```

`limit_req_zone`指令定义了流量闲置的相关参数，而limit_req指令在出现的上下文中启用流量限制。

`limit_req_zone`指令通常在HTTP模块儿中定义，使其可在多个上下文中使用，它需要三个参数：

+ Key - 定义应用限制的请求特性。实例中的Nginx变量 `binary_remote_addr` ，保存客户端IP地址的二进制形式。这意味着，我们可以将每个不同的IP地址限制到，通过第三个参数设置的请求速率。（使用该变量是因为比字符串形式的客户端IP地址 `$remote_addr` ，占用更少的空间）

+ Zone - 定义用于存储每个IP地址壮体啊以及被限制请求URL访问频率的共享内存区域。保存在内存共享区域的信息，意味着可以在Nginx的worker进程之间共享。定义分为两个部分：通过 `zone=keyword` 标识区域的名字，以及冒号后面跟区域大小。16000个IP地址的状态信息，大约需要1MB，所以示例中区域可以存储160000个IP地址。

+ Rate - 定义最大请求速率。在示例中，速率不能超过每秒1个请求。Nginx实际上以毫秒的粒度来跟踪请求，所以速率限制相当于美1000毫秒1个请求（此处为了方便测试，设置成1秒一个请求）。因为不允许“突发情况”，这意味着在前一个请求1000毫秒内到达的请求将被拒绝。

> 当Nginx需要添加新条目时存储空间不足，将会删除旧条目。如果释放的空间仍不够容纳新记录，Nginx将会返回 **503状态码**(Service Temporarily Unavailable)。另外，为了防止内存被耗尽，Nginx每次创建新条目时，最多删除两条60秒内未使用的条目。

`limit_req_zone` 指令设置流量限制和共享内存区域的参数，但实际上并不限制请求速率。所以需要通过添加 `limit_req` 指令，将流量限制应用在特定的 `location` 或者 `server` 块。在上面的示例中，我们对 `/` 请求进行流量限制。

现在每个IP地址被限制为每秒只能请求1次 `/` ，更准确地说，在前一个请求的1000毫秒内不能请求该URL。

* **处理突发**

如果我们在1000毫秒内接收到2个请求怎么办？对于第二个请求，Nginx讲给客户端返回状态码503。这可能并不是我们想要的结果，因为应用本质上趋向于突发性。相反的，我们希望缓冲任何超额的请求，然后及时地处理它们。我们更新下配置，在 `limit_req` 中使用 `burst` 参数：

```shell
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=1r/s;
server {
    listen       9999;
    server_name  localhost;

    location / {
        limit_req zone=mylimit burst=20;
        proxy_pass http://docker.for.mac.host.internal:7001/;
    }
}
```

`burst` 参数定义了超出zone制定速率的情况下（示例中的 `mylimit` 区域，速率限制在每秒1个请求，或每1000毫秒1个请求），客户端还能发起多少请求。上一个请求1000毫秒内到达的请求将会被放入队列，我们将队列大小设置为20。

这意味着，如果从一个给定IP地址发送21个请求，Nginx回立即降低一个请求发送到上有服务器群，然后将于下20个请求放入队列中。然后每1000毫秒转发一个排队的请求，只有当传入请求是队列中排队的请求数超过20时，Nginx才回想客户端返回503。

* 无延迟的排队

配置 `burst` 参数将会是通讯更流畅，但是可能会不太实用，因为该配置会使站点看起来很慢。在上面的示例中，队列中的第20个包需要等待2秒才能呗转发，此时返回给客户端的响应可能不再有用，可以在 `burst` 参数后添加 `nodelay` 参数：

```shell
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=1r/s;
server {
    listen       9999;
    server_name  localhost;

    location / {
        limit_req zone=mylimit burst=20 nodelay;
        proxy_pass http://docker.for.mac.host.internal:7001/;
    }
}
```

使用 `nodelay` 参数，Nginx仍将根据 `burst` 参数分配队列中的位置，并应用已配置的速率限制，而不是清理队列中等待转发的请求。相反地，当一个请求到达“太早”时，只要在队列中能分配位置，Nginx将立即转发这个请求。将队列中的该位置标记为”taken”(占据)，并且不会被释放以供另一个请求使用，直到一段时间后才会被释放(在这个示例中是，1000毫秒后)。

假设如前所述，队列中有20个空位，从给定的IP地址发出的21个请求同时到达。Nginx会立即转发这个21个请求，并且标记队列中占据的20个位置，然后每1000毫秒释放一个位置。如果是25个请求同时到达，Nginx将会立即转发其中的21个请求，标记队列中占据的20个位置，并且返回503状态码来拒绝剩下的4个请求。

现在假设，第一组请求被转发后1001毫秒，另20个请求同时到达。队列中只会有一个位置被释放，所以Nginx转发一个请求并返回503状态码来拒绝其他19个请求。如果在20个新请求到达之前已经过去了5001毫秒，5个位置被释放，所以Nginx立即转发5个请求并拒绝另外15个。

效果相当于每秒1个请求的“流量限制”。如果希望不限制两个请求间允许间隔的情况下实施“流量限制”，`nodelay`参数是很实用的。

**注意：** 对于大部分部署，我们建议使用`burst`和`nodelay`参数来配置`limit_req`指令。

* 高级配置

通过将基本的“流量限制”与其他Nginx功能配合使用，我们可以实现更细粒度的流量限制。

### 白名单

下面这个例子将展示，如何对任何不在白名单内的请求强制执行“流量限制”：

```shell
geo $limit {
    default         1;
    10.0.0.0/8         0;
    192.168.0.0/64     0;
}

map $limit $limit_key {
    0 "";
    1 $binary_remote_addr;
}

limit_req_zone $limit_key zone=req_zone:10m rate=5r/s;

server {
    location / {
        limit_req zone=req_zone burst=10 nodelay;
    }
}
```

这个例子同时使用了`geo`和`map`指令。`geo`块将给在白名单中的IP地址对应的`$limit`变量分配一个值0，给其它不在白名单中的分配一个值1。然后我们使用一个映射将这些值转为key，如下：

- 如果`$limit`变量的值是0，`$limit_key`变量将被赋值为空字符串
- 如果`$limit`变量的值是1，`$limit_key`变量将被赋值为客户端二进制形式的IP地址

两个指令配合使用，白名单内IP地址的`$limit_key`变量被赋值为空字符串，不在白名单内的被赋值为客户端的IP地址。当`limit_req_zone`后的第一个参数是空字符串时，不会应用“流量限制”，所以白名单内的IP地址(10.0.0.0/8和192.168.0.0/24 网段内)不会被限制。其它所有IP地址都会被限制到每秒5个请求。

`limit_req`指令将限制应用到 **/** 的location块，允许在配置的限制上最多超过10个数据包的突发，并且不会延迟转发。

### location包含多`limit_req`指令

我们可以在一个location块中配置多个`limit_req`指令。符合给定请求的所有限制都被应用时，意味着将采用最严格的那个限制。例如，多个指令都制定了延迟，将采用最长的那个延迟。同样，请求受部分指令影响被拒绝，即使其他指令允许通过也无济于事。

扩展前面将“流量限制”应用到白名单内IP地址的例子：

```shell
http {
    # ...

    limit_req_zone $limit_key zone=req_zone:10m rate=5r/s;
    limit_req_zone $binary_remote_addr zone=req_zone_wl:10m rate=15r/s;

    server {
        # ...
        location / {
            limit_req zone=req_zone burst=10 nodelay;
            limit_req zone=req_zone_wl burst=20 nodelay;
            # ...
        }
    }
}
```

白名单内的IP地址不会匹配到第一个“流量限制”，而是会匹配到第二个`req_zone_wl`，并且被限制到每秒15个请求。不在白名单内的IP地址两个限制能匹配到，所以应用限制更强的那个：每秒5个请求。

* 配置相关功能

**日志记录**

默认情况下，Nginx会在日志中记录由于流量限制而延迟或丢弃的请求，如下所示：

```shell
2015/06/13 04:20:00 [error] 120315#0: *32086 limiting requests, excess: 1.000 by zone "mylimit", client: 192.168.1.2, server: nginx.com, request: "GET / HTTP/1.0", host: "nginx.com"
```

日志条目中包含的字段：

- *limiting requests* - 表明日志条目记录的是被“流量限制”请求
- *excess* - 每毫秒超过对应“流量限制”配置的请求数量
- *zone* - 定义实施“流量限制”的区域
- *client* - 发起请求的客户端IP地址
- *server* - 服务器IP地址或主机名
- *request* - 客户端发起的实际HTTP请求
- *host* - HTTP报头中host的值

默认情况下，Nginx以`error`级别来记录被拒绝的请求，如上面示例中的`[error]`所示(Ngin以较低级别记录延时请求，一般是`info`级别)。如要更改Nginx的日志记录级别，需要使用`limit_req_log_level`指令。这里，我们将被拒绝请求的日志记录级别设置为`warn`：

```shell
location /login/ {
    limit_req zone=mylimit burst=20 nodelay;
    limit_req_log_level warn;

    proxy_pass http://my_upstream;
}
```

### 指定`location`拒绝所有请求

如果你想拒绝某个指定URL地址的所有请求，而不是仅仅对其限速，只需要在`location`块中配置`deny all `指令：

```shell
location /foo.php {
    deny all;
}
```

#### 2、Sentinel测试

```shell
    <dependencies>
        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-core</artifactId>
            <version>1.8.0</version>
        </dependency>
    </dependencies>
```

```shell
public class FlowControl {
    public static void main(String[] args) {
        initFlowRules();
        while (true) {
            // 1.5.0 版本开始可以直接利用 try-with-resources 特性
            try (Entry entry = SphU.entry("HelloWorld")) {
                // 被保护的逻辑
                System.out.println("hello world");
            } catch (BlockException ex) {
                // 处理被流控的逻辑
                System.out.println("blocked!");
                break;
            }
        }
    }

    private static void initFlowRules(){
        List<FlowRule> rules = new ArrayList<FlowRule>();
        FlowRule rule = new FlowRule();
        rule.setResource("HelloWorld");
        rule.setGrade(RuleConstant.FLOW_GRADE_QPS);
        // Set limit QPS to 20.
        rule.setCount(20);
        rules.add(rule);
        FlowRuleManager.loadRules(rules);
    }

    @SentinelResource(value = "helloWorld")
    public void  helloWorld() {
        // 资源中的逻辑
        System.out.println("Hello World");
    }
}
```

Sentinel是一个很好的代码注入形式的架构，不对业务进行侵入，只需要使用注解就可以实现。此处我们通过设置QPS为20，当QPS高于20的时候，测试结果将打印blocked并且退出程序。

参考：

[Nginx如何做流量控制 - 云+社区 - 腾讯云](https://cloud.tencent.com/developer/article/1182207)

[API网关之流控篇](https://baijiahao.baidu.com/s?id=1668904920302399525&wfr=spider&for=pc)

[Nginx 的两种限流方式 - 开发者头条](https://toutiao.io/posts/r9wf3f/preview)

[Sentinel](https://sentinelguard.io/zh-cn/docs/quick-start.html)

[统一流控服务开源-1：场景&amp;业界做法&amp;算法篇 - Eric zhou - 博客园](https://www.cnblogs.com/tianqing/p/9044155.html)
