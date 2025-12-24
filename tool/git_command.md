## git 相关处理


### 1、设置github 代理
单独给github 设置代理
```shell
git config --global http.https://github.com.proxy 127.0.0.1:7890
git config --global https.https://github.com.proxy 127.0.0.1:7890
```
git config --global http.https://github.com.proxy http://127.0.0.1:7897
查看代理设置：

git config --global --get http.proxy
git config --global --get https.proxy

```shell
git config --list | grep -i proxy
```

清除代理
git config --global --unset http.proxy
git config --global --unset https.proxy

### 2、github 设置token密钥

