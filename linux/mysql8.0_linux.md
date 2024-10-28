## linux系统  yum安装mysql 8.0 过程详解

### 一、卸载

1、检查mysql
rpm -qa | grep mysql
2、删除包
rpm -e --nodeps +包名
3、检查mysql 文件夹
find / -name mysql
4、删除包
rm -rf +包名

### 二、下载解压

官方网址：https://downloads.mysql.com/archives/community/

解压,文件夹重命名为mysql，移动位置并重新命名

```
tar -xf mysql-8.0.20-linux-glibc2.12-x86_64.tar.xz 
mv mysql-8.0.20-linux-glibc2.12-x86_64.tar.xz /usr/local/mysql
```

### 三、创建mysql组


1、创建一个新的用户组，名为“mysql”
2、创建一个新的用户，也名为“mysql”

```
groupadd mysql 
useradd -r -g mysql mysql 	

```

### 四、创建目录并赋予权限

1、创建目录
2、赋予权限

```
mkdir -p  /data/mysql             
chown mysql:mysql -R /data/mysql 	
```

### 五、配置my.cnf文件

#### 1、设置配置etc/my.cnf

```
[mysqld]
bind-address=0.0.0.0
port=3306
user=mysql
basedir=/usr/local/mysql
datadir=/data/mysql
socket=/tmp/mysql.sock
log-error=/data/mysql/mysql.err
pid-file=/data/mysql/mysql.pid
#character config
character_set_server=utf8mb4
symbolic-links=0
explicit_defaults_for_timestamp=true
```
#### 2、解释
1.[mysqld]:这是一个节标题,表示下面的配置选项都是针对 mysqle守护进程的。mysqld是MySQL服务器的
守护进程。
2.bind-address=0.0.0.0.0.0:这告诉MySQL服务器监所所有可用的网络接口。0.0.0.0.意味着MySQL将接受来自任何IP
地址的连接。如果你只想让它从本地机器接受连接,可以使用127.0.0.1
3.port=3306:设置MySQL服务器监听的端口号。默认情况下,MySSQL使用3306端口。
user=mysql:指定运行MySQL服务器进程的系统用户。这里,MySQL将以mysq|用户身份运行。
5.basedir=/software/mysql:指定MySQL安装的基目录,也就是包含MySQL二进制文件和其他相关文件的目录。
6.datadir=/data/mysql:指定MySQL用于存储数据库数据的目录。
7.socket=/tmp/mysql.sock:这是MySQL服务器使用的UNIX套接字文件的路径。当客户端和服务器在同一台机器
上时,UNIX套接字通常比TCP/IP连接更快。



### 六、初始化数据库

1、进入mysql/bin下

```
cd usr/local/mysql/bin/
```

2、需要安装libaio库(第一次安装需要)

```
sudo yum install libaio
```

3、初始化

```
./mysqld --defaults-file=/etc/my.cnf --basedir=/usr/local/mysql/ --datadir=/data/mysql/ --user=mysql --initialize
```

4、查看密码

```
cat /data/mysql/mysql.err
```

5、将mysql.server放置到/etc/init.d/mysql中

```
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql
```

mysql.server 脚本放置到 /etc/init.d/mysql 目录中的主要作用是使得你可以使用标准的系统服务管理工具来管理 MySQL服务。    
在 Linux 系统中，/etc/init.d/ 目录存放的是一系列系统服务的管理（启动、停止等）脚本。    
mysql.server 是一个官方针对 Unix 和类 Unix 系统二进制版本安装包中包含的脚本，它主要用于启动、查看和停止 mysqld
进程服务。    
当 mysql.server 脚本被放置在 /etc/init.d/mysql 目录中后，你可以使用 service 命令来管理 MySQL 服务，例如：    
service mysql start：启动 MySQL 服务。    
service mysql stop：停止 MySQL 服务。    
service mysql restart：重启 MySQL 服务。

6、启动、查看服务

```
service mysql start
ps -ef|grep mysql
```

7、设置mysql环境变量

```
export PATH=$PATH:/usr/local/mysql/bin
```

/etc/profile 在最后添加一行：

```
export PATH=$PATH:/usr/local/mysql/bin
```

刷新配置：

```
source /etc/profile
```

8、检查mysql

```
whereis mysql;
whereis mysqldump;
```

### 七、配置mysql

1、登录mysql

```
mysql -u root -p

```

2、创建用户密码

```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '@root123456';
```

3、刷新配置

```
flush privileges;
```

### 八、配置远程连接

```
use mysql;
update user set host='%' where user='root';
flush privileges;
```

