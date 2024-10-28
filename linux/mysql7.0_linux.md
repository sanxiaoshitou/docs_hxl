## linux系统  yum安装mysql 7.0 过程详解

### 一、下载rpm安装包
```
wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
rpm -ivh mysql-community-release-el7-5.noarch.rpm
```

### 二、安装mysql 服务器
```
yum install mysql-server
yum install mysql-connector-java
```

### 三、修改权限
```
chown root:root -R /var/lib/mysql
```

### 四、启动mysql
```
systemctl enable mysqld
systemctl start mysql
```

### 五、设置账号密码、远程权限和建库建表权限
1、登录mysql
```
#登录mysql （第一次登录 不需要密码 直接回车键）
mysql -uroot -p
#切到mysql数据库
use mysql;
#查询所有的用户
select host,user,password from user;
```
2、修改root 密码
```
#保留一个root 账号 本地连接权限 设置密码
update user set password=password("root") where user="root" and host="localhost";
```

3、新建账号 设置权限
```
#创建账号
create user mysql@"%" identified by "mysql";
#设置远程连接权限
grant usage on *.* to mysql@"localhost" identified by "mysql" with grant option;
 
#添加权限，添加所有权限
grant all privileges on *.* to mysql@"%" identified by "mysql";
#立即启用修改
flush privileges;
```
### 六、mysql卸载
mysql 卸载不干净 会影响重新安装mysql，现介绍卸载详情过程。    
1、停止mysql服务    
service mysqld stop 或者 systemctl stop mysql    
2、卸载mysql软件包
```
# 1.查询软件包
rpm -aq | grep mysql
mysql-community-release-el7-5.noarch
mysql-community-server-5.6.45-2.el7.x86_64
mysql-community-common-5.6.45-2.el7.x86_64
mysql-community-client-5.6.45-2.el7.x86_64
mysql-connector-java-5.1.25-3.el7.noarch
mysql-community-libs-5.6.45-2.el7.x86_64
 
#2.卸载所以包  命令：rpm -ev 包名 --nodeps       忽略依赖--nodeps 
rpm -ev mysql-community-release-el7-5.noarch --nodeps
...
 
#3.删除mysql安装文件   删除命令：rm -rf 文件路径
find / -name "mysql"
/etc/logrotate.d/mysql
/var/lib/mysql
/var/lib/mysql/mysql
/usr/share/mysql
/usr/bin/mysql
/usr/lib64/mysql
 
rm -rf /var/lib/mysql
...
 
#mysql 卸载完毕
```







