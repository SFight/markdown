# Linux删除重复行

```shell
# 文本处理时，经常要删除重复行，下面是三种方法

# 第一，用sort+uniq，注意，单纯uniq是不行的。
sort -n test.txt | uniq

# 第二，用sort+awk命令，注意，单纯awk同样不行，原因同上。

sort -n $file | awk '{if($0!=line)print; line=$0}'
　
# 第三，用sort+sed命令，同样需要sort命令先排序。

sort -n $file | sed '$!N; /^.∗.∗\n\1$/!P; D'

```

# 查询文本文件指定字符串

```shell

```
