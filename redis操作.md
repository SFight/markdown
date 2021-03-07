# 查询redis中的大key

```shell
redis-cli -h wqxuetang.redis.rds.aliyuncs.com --bigkeys
```

# 批量删除redis中的key

```shell
redis-cli -h wqxuetang.redis.rds.aliyuncs.com -n 0 keys article_* |xargs redis-cli -h wqxuetang.redis.rds.aliyuncs.com  -n 0 DEL
redis-cli -h wqxuetang.redis.rds.aliyuncs.com -n 0 keys bklong_* |xargs redis-cli -h wqxuetang.redis.rds.aliyuncs.com -n 0 DEL
redis-cli -h wqxuetang.redis.rds.aliyuncs.com -n 0 keys wq_cata_* |xargs redis-cli -h wqxuetang.redis.rds.aliyuncs.com -n 0 DEL
redis-cli -h wqxuetang.redis.rds.aliyuncs.com -n 0 keys wq_srch_init* |xargs redis-cli -h wqxuetang.redis.rds.aliyuncs.com -n 0 DEL
redis-cli -h wqxuetang.redis.rds.aliyuncs.com -n 0 keys wq_srch_cnclass* |xargs redis-cli -h wqxuetang.redis.rds.aliyuncs.com -n 0 DEL
redis-cli -h wqxuetang.redis.rds.aliyuncs.com -n 0 keys limit_nvc* |xargs redis-cli -h wqxuetang.redis.rds.aliyuncs.com -n 0 DEL
redis-cli -h wqxuetang.redis.rds.aliyuncs.com -n 0 keys type_wqxuetang_* |xargs redis-cli -h wqxuetang.redis.rds.aliyuncs.com -n 0 DEL
```
