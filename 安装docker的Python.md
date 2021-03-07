# 安装docker的Python

### 下载镜像

```shell
docker pull python
```

### Centos7 安装mailx发送邮件

```shell
sudo yum install mailx -y
```

#### mailx使用

```shell
mail -h
```

1. 发送邮件

   ```shell
   echo "hello world" | mail -s "title" xxxx@qq.com xxxx@163.com
   ```

2. 发送邮件设置外部邮箱

   ```shell
   set from=xxxx@163.com
   set smtp=smtp.163.com  
   set smtp-auth-user=xxxx@163.com
   set smtp-auth-password=授权码
   set smtp-auth=login
   ```
