## linux下  yum配置阿里云镜像

### 一、备份之前的仓库文件
```
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.back
```
### 二、下载阿里云的仓库文件
```
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```
### 二、清除缓存
```
yum clean all
yum makecache
```





