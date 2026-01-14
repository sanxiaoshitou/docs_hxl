# MongoDB


MySQL	MongoDB
数据库（DataBase）	数据库（DataBase）
数据表（Table）	数据集合（Collection）
数据行（Row）	数据文档（Document）
列/字段（Column）	字段（Field）
索引（Index）	索引（Index）

## 库和集合操作

* 查询所有数据库：show dbs;或show databases;
* 切换/创建数据库：use 库名;
* 查看目前所在的数据库：db;
* 查看MongoDB目前的连接信息：db.currentOp();
* 查看当前数据库的统计信息：db.stats();
* 删除数据库：db.dropDatabase("库名");
* 查看库中的所有集合：show collections;或show tables;
* 显式创建集合：db.createCollection("集合名");
* 查看集合统计信息：db.集合名.stats();
* 查看集合的数据大小：db.集合名.totalSize();
* 删除指定集合：db.集合名.drop();

## 集合操作
