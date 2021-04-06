# Nginx API for Lua

[API文档](https://github.com/openresty/lua-nginx-module#nginx-api-for-lua)

## Nginx lua加载指令模块儿

![](https://pic2.zhimg.com/80/v2-77b10c21fcbf91d828a0f85c4fba60b5_1440w.jpg)

## Nginx lua连接redis

```shell
local redis = require "resty.redis"
local red = redis:new()
red:set_timeout(1000)
local ok, err = red:connect("127.0.0.1", 6379)
if not ok then
    ngx.say("failed to connect: ", err)
    return
else
    ngx.say("redis connetc success")
end

-- 写入数据
ok, err = red:set("test", "123")
if not ok then
    ngx.say("set data error", err)
    return
end
ngx.say("set data success")

-- 读取数据
local res, err = red:get("test")
if not res then
    ngx.say("get data error ", err)
    return
end
if res == ngx.null then
    ngx.say("data is nil")
    return
end
```

# 
