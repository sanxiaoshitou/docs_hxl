### mysql 常见索引失效场景


### 一、联合索引不满足最左匹配原则
索引：
```mysql
KEY `union_idx` (`id_no`,`username`,`age`)
```
索引失效sql:
```mysql
explain select * from t_user where username = 'Tom2' and age = 12;
```

### 二、使用了select *

### 三、索引列参与运算
索引失效sql:
```mysql
explain select * from t_user where id + 1 = 2 ;
```

### 四、索引列参使用了函数
索引失效sql:
```mysql
explain select * from t_user where SUBSTR(id_no,1,3) = '100';
```

### 五、错误的Like使用
索引失效sql:
```mysql
explain select * from t_user where id_no like '%00%';
```
方式一：like '%abc'； 不走索引       
方式二：like 'abc%'； 走索引    
方式三：like '%abc%'；不走索引   

### 六、类型隐式转换
索引失效sql:
```mysql
explain select * from t_user where id_no = 1002;
```
id_no字段类型为varchar，但在SQL语句中使用了int类型，导致全表扫描。

### 七、使用OR操作 不当

### 八、两列做比较
索引失效sql:
```mysql
explain select * from t_user where id > age;
```

### 九、不等于比较
索引失效sql:
```mysql
explain select * from t_user where id_no <> '1002';
```

### 十、is not null
查询条件使用is null时正常走索引，使用is not null时，不走索引。

### 十一、not in和not exists
在日常中使用比较多的范围查询有in、exists、not in、not exists、between and等。    
in not in 主键走索引，普通索引失效  