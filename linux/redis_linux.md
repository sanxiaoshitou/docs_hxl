## linux系统  redis安装

### 一、下周rpm包,解压
下载
```
wget http://download.redis.io/releases/redis-5.0.7.tar.gz
```

解压、移动文件夹
```
tar -zvxf redis-5.0.7.tar.gz
mv /root/redis-5.0.7 /usr/local/redis

```

### 二、安装
cd到/usr/local/redis目录，输入命令make执行编译命令
make
```
make PREFIX=/usr/local/redis install
```

注：找不到文件 需要执行 yum install gcc


### 三、修改配置
#### 1、配置说明
>1、daemonize yes表示启用守护进程，默认是no即不以守护进程方式运行。其中Windows系统下不支持启用守护进程方式运行   
>2、port   指定 Redis 监听端口，默认端口为 6379    
>3、bind   绑定的主机地址,如果需要设置远程访问则直接将这个属性备注下或者改为bind * 即可,这个属性和下面的protected-mode控制了是否可以远程访问     
>4、protected-mode 护模式，该模式控制外部网是否可以连接redis服务，默认是yes,所以默认我们外网是无法访问的，如需外网连接rendis服务则需要将此属性改为no。        
>5、timeout 当客户端闲置多长时间后关闭连接，如果指定为 0，表示关闭该功能
>6、loglevel 日志级别，默认为 notice 设置：debug、verbose、notice、warning
>7、databases 设置数据库的数量，默认的数据库是0。整个通过客户端工具可以看得到
>8、rdbcompression 指定存储至本地数据库时是否压缩数据，默认为 yes，Redis 采用 LZF 压缩，如果为了节省 CPU 时间，可以关闭该选项，但会导致数据库文件变的巨大。
>9、dbfilename 指定本地数据库文件名，默认值为 dump.rdb
>10、dir 指定本地数据库存放目录
>11、requirepass 设置Redis 连接密码，如果配置了连接密码，客户端在连接 Redis 时需要通过 AUTH <password> 命令提供密码，默认关闭
>maxclients 设置同一时间最大客户端连接数，默认无限制，Redis 可以同时打开的客户端连接数为 Redis 进程可以打开的最大文件描述符数，如果设置 maxclients 0，表示不作限制。
当客户端连接数到达限制时，Redis 会关闭新的连接并向客户端返回 max number of clients reached 错误信息。
> maxmemory 指定 Redis 最大内存限制，Redis 在启动时会把数据加载到内存中，达到最大内存后，Redis 会先尝试清除已到期或即将到期的 Key，当此方法处理 后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。
Redis 新的 vm 机制，会把 Key 存放内存，Value 会存放在 swap 区。配置项值范围列里XXX为数值。

后台运行
daemonize yes
远程连接设置
bind 0.0.0.0 或*
protected-mode no
密码设置
requirepass 123456

### 四、支持远程，设置密码的配置
vim /usr/local/redis/redis.conf    
修改内容
```
bind 0.0.0.0 或*
protected-mode no
daemonize yes
requirepass 123456
```

### 四、启动和关闭
#### 启动服务
./bin/redis-server ./redis.conf 

### 关闭服务
1、连接redis ./
2、执行命令 shutdown

### 五、设置脚本执行

### 六、辅助命令
1、查询redis进程方试
ps -aux | grep redis
2、采取端口监听查看方式
netstat -lanp | grep 6379


