## Linux下安装Java环境三种方式（tar.gz、rpm、yum）



### 一、jdk下载

官网下载：http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.rpm。

### 二、卸载环境
#### RPM方式卸载Java环境：
1、rpm查询java安装包名称（注：rpm -qa 列举出所有RPM安装的包） 
```shell
rpm -qa | grep 'java\|jdk\|gcj\|jre'    
```
2、查询安装包安装文件位置（rpm -ql 安装包名称）
```shell
rpm -ql jdk-1.8-1.8.0_381-9.x86_64
```
3、卸载查询出来的所有安装包名称（注：rpm -e --nodeps 是RPM卸载命令）
```shell
rpm -e --nodeps jdk-1.8-1.8.0_381-9.x86_64  
``` 

#### yum方式卸载Java环境：
1、yum查询Java安装的环境信息（注：yum list installed 列举所有安装的服务）
```shell
yum list installed | grep 'java\|jdk\|gcj\|jre'
``` 
2、 卸载查询出来的所有安装信息（注：yum -y remove 是yum卸载命令）
```shell
yum -y remove jdk-1.8.x86_64
```

#### tar.gz方式安装后的卸载：
1、删除之前的解压文件位置
```shell
rm -rf /usr/local/jdk1.8.0_381/
```
2、剔除之前配置的 /etc/profile 下的配置信息，如下示例：
```shell
export JAVA_HOME=/usr/local/jdk1.8.0_381
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib
```
3、删除配置生效
```shell
source /etc/profile
```

### 三、安装

1、yum 安装

2、rpm 安装

### 四、配置环境变量

liunx路径/etc/profile中增加：

export JAVA_HOME=/usr/java/jdk1.8.0_131
export CLASSPATH=.:$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar
export PATH=$JAVA_HOME/bin:$PATH

### 五、查看安装

1、查看版本
java -version
2、查看JDK的安装路径（安装后才有）
which java

### 六、其他：

一般rpm、yum方式安装的不需要配置环境变量，但是若识别不到还是老老实实配置环境变量
若yum安装则默认Java被安装在 “ /usr/lib/jvm ”（一般不用手动配置）
若RPM安装则默认Java被安装在 “ /usr/java/jdk1.8.0-x64 ”（一般不用手动配置）

### 参考文献
文献：https://www.cnblogs.com/antLaddie/p/17599359.html


