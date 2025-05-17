## git 相关处理


### 1、设置github 代理
单独给github 设置代理
```shell
git config --global http.https://github.com.proxy 127.0.0.1:7890
git config --global https.https://github.com.proxy 127.0.0.1:7890
```

查看代理设置：
```shell
git config --list | grep -i proxy
```
