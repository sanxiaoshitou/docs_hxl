# Spring 事务
事务是一个最小的不可再分的工作单元。 
一个事务对应一套完整的业务操作。事务管理是指这些操作要么全部成功执行，要么全部回滚，从而保证数据的一致性和完整性

## 一、事务特性 ACID
### 1. 原子性（Atomicity）
事务是最小的执行单位，不允许分割。事务的原子性确保动作要么全部完成，要么完全不起作用。

### 2. 一致性（Consistency）
执行事务前后，数据保持一致，多个事务对同一个数据读取的结果是相同的。

### 3. 隔离性（Isolation）
并发访问数据库时，一个用户的事务不被其他事务所干扰，各并发事务之间数据库是独立的。

### 4. 持久性（Durability）
一个事务被提交后，它对数据库中数据的改变是持久的，即使数据库发生故障也不应该对其有任何影响。

## 二、事务隔离级别

Spring 支持 5 种事务隔离级别：

### 1. DEFAULT
使用后端数据库默认的隔离级别（大多数数据库默认是 READ_COMMITTED）。

### 2. READ_UNCOMMITTED（读未提交）
允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读。性能最高，但安全性最低。

### 3. READ_COMMITTED（读已提交）
允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生。这是 Oracle 等数据库的默认隔离级别。

### 4. REPEATABLE_READ（可重复读）
对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。这是 MySQL 的默认隔离级别。

### 5. SERIALIZABLE（串行化）
最高的隔离级别，完全服从 ACID 的隔离级别，确保不发生脏读、不可重复读以及幻读。也是最慢的事务隔离级别，因为它通常是通过完全锁定事务相关的数据库表来实现的。

### 并发问题对照表

| 隔离级别 | 脏读 | 不可重复读 | 幻读 |
|---------|------|-----------|------|
| READ_UNCOMMITTED | √ | √ | √ |
| READ_COMMITTED | × | √ | √ |
| REPEATABLE_READ | × | × | √ |
| SERIALIZABLE | × | × | × |

## 三、事务传播行为
Spring 提供了 7 种事务传播行为：

### 1. REQUIRED（默认）
如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。

### 2. SUPPORTS
如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务方式继续运行。

### 3. MANDATORY
如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。

### 4. REQUIRES_NEW
创建一个新的事务，如果当前存在事务，则把当前事务挂起。

### 5. NOT_SUPPORTED
以非事务方式运行，如果当前存在事务，则把当前事务挂起。

### 6. NEVER
以非事务方式运行，如果当前存在事务，则抛出异常。

### 7. NESTED
如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；

## 四、Spring 事务使用方式
### 1. 声明式事务（推荐）

#### @Transactional 注解常用属性
```java
@Transactional(
        propagation = Propagation.REQUIRED,// 传播行为 
        isolation = Isolation.DEFAULT, // 隔离级别 
        timeout = 30, // 超时时间（秒） 
        readOnly = false, // 是否只读 
        rollbackFor = Exception.class, // 触发回滚的异常 
        noRollbackFor = {} // 不触发回滚的异常 
)
public void businessMethod() {  }
```

### 2. 编程式事务## 
```java
@Service 
public class ProgrammaticTransactionService {
    @Autowired
    private TransactionTemplate transactionTemplate;
    
    @Autowired
    private OrderMapper orderMapper;
    
    public void createOrder(Order order) {
        transactionTemplate.execute(status -> {
            try {
                orderMapper.insert(order);
                // 其他业务操作
                return null;
            } catch (Exception e) {
                status.setRollbackOnly(); // 手动回滚
                throw e;
            }
        });
    }
}
```



## 五、@Transactional 失效场景

### 1. 方法非 public
@Transactional 只能应用于 public 方法，其他修饰符会被忽略。

### 2. 自调用问题
同一个类中，非事务方法调用事务方法，事务会失效。

**解决方案：**
- 将方法拆分到不同的 Service 类中
- 注入自身代理对象调用
- 使用 AopContext.currentProxy() 获取代理对象

### 3. 异常被捕获
在方法内部捕获了异常且未抛出，事务不会回滚。
**解决方案：**
- 指定 rollbackFor：`@Transactional(rollbackFor = Exception.class)`

### 5. 数据库引擎不支持
MySQL 的 MyISAM 引擎不支持事务，需要使用 InnoDB 引擎。

### 6. 方法被 final 或 static 修饰
Spring 事务基于 AOP 实现，final 和 static 方法无法被代理，事务会失效。

## 六、最佳实践

### 1. 事务粒度控制
- 事务范围尽可能小，只包含必要的数据库操作
- 避免在事务中进行远程调用、文件 IO 等耗时操作

### 2. 合理设置超时时间
对于复杂业务，设置合理的超时时间，避免长时间占用数据库连接。

### 3. 明确指定回滚异常
建议显式指定 rollbackFor，避免因异常类型导致的事务不回滚问题。

### 4. 避免大事务
- 不要在事务中查询大量数据
- 避免在事务中进行批量处理（可以分批提交）
- 减少事务中的锁竞争

### 5. 注意事务传播行为
根据业务场景选择合适的传播行为：
- 大部分场景使用默认的 REQUIRED
- 需要独立事务时使用 REQUIRES_NEW
- 需要同步执行时使用 NESTED

## 七、常见问题

### 1. 如何在一个事务中调用另一个事务？

### 2. 如何实现部分回滚？

使用 Savepoint（保存点）机制：
```java
@Transactional 
public void partialRollback() { 
    Object savepoint = TransactionAspectSupport.currentTransactionStatus().createSavepoint();
    try {
        // 第一部分操作
        mapper.insert(record1);
        
        // 第二部分操作（可能失败）
        mapper.insert(record2);
    } catch (Exception e) {
        // 回滚到保存点
        TransactionAspectSupport.currentTransactionStatus()
            .rollbackToSavepoint(savepoint);
    }
}
```

### 3. 读写分离场景下的事务注意事项
- 写操作必须在主库执行
- 读操作可以在从库执行，但要考虑主从延迟
- 事务中的读操作应该在主库执行，保证数据一致性








