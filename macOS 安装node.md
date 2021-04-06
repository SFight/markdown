# macOS 安装node

```shell
brew link node
brew uninstall node
brew install node
```

**此时会出现link异常**

```shell
Linking /usr/local/Cellar/node/15.11.0... 
Error: Could not symlink include/node/common.gypi
/usr/local/include/node is not writable.
```

**解决方式**

```shell
cd /usr/local
$ sudo chown -R <your-username>:<your-group-name> *

#如果不知道group
id -g
```

**重新执行link**

```shell
brew link --overwrite node
```
