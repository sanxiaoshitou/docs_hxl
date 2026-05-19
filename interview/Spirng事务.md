# Spring 事务与常见面试题

## 一、Spring 事务概述

### 1.1 什么是事务？
事务是数据库操作的最小工作单元，是作为单个逻辑工作单元执行的一系列操作；这些操作作为一个整体一起向系统提交，要么都执行、要么都不执行；事务是一组不可再分割的操作集合。

### 1.2 事务的 ACID 特性

| 特性 | 说明 |
|------|------|
| **原子性（Atomicity）** | 事务是一个不可分割的工作单位，事务中的所有操作要么全部成功，要么全部失败回滚 |
| **一致性（Consistency）** | 事务执行前后，数据库从一个一致性状态变到另一个一致性状态 |
| **隔离性（Isolation）** | 多个事务并发执行时，一个事务的执行不应影响其他事务 |
| **持久性（Durability）** | 事务一旦提交，对数据库的改变就是永久性的 |

## 二、@Transactional 注解详解

### 2.1 常用属性

| 属性 | 类型 | 说明 |
|------|------|------|
| `propagation` | Propagation | 事务传播行为，默认 `REQUIRED` |
| `isolation` | Isolation | 事务隔离级别，默认 `DEFAULT`（使用数据库默认级别） |
| `readOnly` | boolean | 是否为只读事务，默认 `false` |
| `timeout` | int | 事务超时时间（秒），默认 `-1`（不超时） |
| `rollbackFor` | Class[] | 需要回滚的异常类型，默认运行时异常 |
| `noRollbackFor` | Class[] | 不需要回滚的异常类型 |


## 三、事务传播行为（Propagation）

Spring 定义了 7 种传播行为：

| 传播行为 | 说明 |
|---------|------|
| **REQUIRED（默认）** | 当前有事务就加入，没有则新建事务 |
| **SUPPORTS** | 当前有事务就加入，没有则以非事务方式执行 |
| **MANDATORY** | 当前必须有事务，否则抛出异常 |
| **REQUIRES_NEW** | 无论当前是否有事务，都新建一个事务，挂起当前事务 |
| **NOT_SUPPORTED** | 以非事务方式执行，挂起当前事务 |
| **NEVER** | 必须以非事务方式执行，当前有事务则抛出异常 |
| **NESTED** | 当前有事务则创建嵌套事务（savepoint），没有则新建事务 |

### 3.1 重点详解

#### REQUIRED（默认）
```java
// 方法A调用方法B，方法B使用 REQUIRED
// 如果A有事务，B加入A的事务
// 如果A没有事务，B创建新事务
@Transactional
public void methodA() {
    methodB(); // B加入A的事务
}

@Transactional
public void methodB() {
    // 与A在同一个事务中
}
```

#### REQUIRES_NEW
```java
// 方法A调用方法B，方法B使用 REQUIRES_NEW
// B会开启一个新事务，A的事务被挂起
@Transactional
public void methodA() {
    methodB(); // B新建独立事务，A的事务挂起
    // B如果回滚，不影响A
}

@Transactional(propagation = Propagation.REQUIRES_NEW)
public void methodB() {
    // 独立的新事务
}
```

#### NESTED
```java
// 类似 REQUIRES_NEW，但 B 回滚不影响 A
// 基于 JDBC Savepoint 实现
@Transactional
public void methodA() {
    methodB(); // B创建嵌套事务（Savepoint）
    // B回滚只回滚到Savepoint，A可继续执行
}

@Transactional(propagation = Propagation.NESTED)
public void methodB() {
    // 嵌套事务
}
```

---

## 四、事务隔离级别（Isolation）

| 隔离级别 | 脏读 | 不可重复读 | 幻读 | 说明 |
|---------|------|-----------|------|------|
| **READ_UNCOMMITTED** | ✓ | ✓ | ✓ | 最低级别，允许读取未提交数据 |
| **READ_COMMITTED** | ✗ | ✓ | ✓ | 只允许读取已提交数据（Oracle默认） |
| **REPEATABLE_READ** | ✗ | ✗ | ✓ | 可重复读（MySQL默认） |
| **SERIALIZABLE** | ✗ | ✗ | ✗ | 最高级别，完全串行化 |
| **DEFAULT** | - | - | - | 使用数据库默认隔离级别 |

### 4.1 三大并发问题

- **脏读**：一个事务读取到了另一个事务未提交的数据
- **不可重复读**：同一事务中，两次读取同一数据，结果不一致（被其他事务修改并提交）
- **幻读**：同一事务中，两次查询同一范围的数据，结果集数量不一致（被其他事务插入/删除）

```java
@Transactional(isolation = Isolation.REPEATABLE_READ)
public void queryData() {
    // 设置隔离级别为可重复读
}
```

---

## 五、Spring 事务实现原理

### 5.1 基于 AOP 的动态代理
Spring 声明式事务基于 AOP 实现，通过代理对象拦截方法调用：

1. **解析 `@Transactional`**：Spring 在 Bean 创建时解析类/方法上的事务注解
2. **创建代理对象**：如果 Bean 包含 `@Transactional` 方法，Spring 会为其创建代理对象
3. **方法拦截**：调用代理对象的方法时，先进入 `TransactionInterceptor`
4. **事务管理**：`TransactionInterceptor` 调用 `PlatformTransactionManager` 开启/提交/回滚事务

### 5.2 两种代理方式

| 代理方式 | 条件 | 特点 |
|---------|------|------|
| **JDK 动态代理** | 目标类实现了接口 | 基于接口代理，要求 `@Transactional` 放在接口方法上才一定生效 |
| **CGLIB 代理** | 目标类未实现接口 | 基于类代理，通过继承目标类生成子类 |

> Spring Boot 2.x 默认使用 CGLIB 代理（`spring.aop.proxy-target-class=true`）

### 5.3 核心组件

```
@Transactional
   ↓
TransactionInterceptor（AOP 拦截器）
   ↓
PlatformTransactionManager（事务管理器）
   ↓
DataSourceTransactionManager / JtaTransactionManager
   ↓
Connection（数据库连接）
```

---

## 六、事务失效场景（重点面试题）

### 6.1 非 public 方法
`@Transactional` 只能作用于 public 方法，非 public 方法事务不生效。

```java
// 错误：private 方法事务不生效
@Transactional
private void updateData() {
    // 事务不会生效
}
```

### 6.2 同类方法自调用
通过 `this` 调用同类中的 `@Transactional` 方法，事务不生效（绕过了代理对象）。

```java
@Service
public class UserService {
    
    public void methodA() {
        this.methodB(); // ❌ 事务不生效！
    }
    
    @Transactional
    public void methodB() {
        // 事务不会生效
    }
}
```

**解决方案**：
- 注入自身代理对象调用
- 使用 `AopContext.currentProxy()`
- 将方法拆分到不同类中

```java
@Service
public class UserService {
    
    @Autowired
    private UserService self; // 注入自身代理
    
    public void methodA() {
        self.methodB(); // ✓ 事务生效
    }
    
    @Transactional
    public void methodB() {
        // 事务生效
    }
}
```

### 6.3 异常被 catch 未抛出
方法内部捕获异常并未向外抛出，事务无法感知异常，不会回滚。

```java
@Transactional
public void updateData() {
    try {
        userDao.update(user);
        int i = 1 / 0; // 异常
    } catch (Exception e) {
        // ❌ 异常被吞掉，事务不会回滚！
        log.error("error", e);
    }
}
```

**解决方案**：捕获后重新抛出，或手动回滚
```java
@Transactional
public void updateData() {
    try {
        userDao.update(user);
        int i = 1 / 0;
    } catch (Exception e) {
        log.error("error", e);
        throw new RuntimeException(e); // 重新抛出
        // 或：TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
    }
}
```

### 6.4 非运行时异常未配置 rollbackFor
默认情况下，`@Transactional` 只在遇到运行时异常（`RuntimeException`）和 Error 时回滚。遇到受检异常（`Exception`）不回滚。

```java
// 错误：遇到 Exception 不会回滚
@Transactional
public void updateData() throws Exception {
    userDao.update(user);
    throw new Exception("error"); // ❌ 不会回滚！
}

// 正确：指定 rollbackFor
@Transactional(rollbackFor = Exception.class)
public void updateData() throws Exception {
    userDao.update(user);
    throw new Exception("error"); // ✓ 会回滚
}
```

### 6.5 数据库引擎不支持事务
如 MySQL 的 MyISAM 引擎不支持事务，需要改为 InnoDB。

### 6.6 事务传播行为设置不当
如使用 `Propagation.NOT_SUPPORTED` 或 `Propagation.SUPPORTS` 时，可能以非事务方式执行。

### 6.7 多数据源未指定事务管理器
多数据源场景下，需要指定正确的事务管理器：
```java
@Transactional(transactionManager = "masterTransactionManager")
public void updateData() {
    // ...
}
```

### 6.8 Spring 容器未管理 Bean
非 Spring 管理的对象调用 `@Transactional` 方法，事务不生效。

```java
// 错误：手动 new 的对象
UserService service = new UserService();
service.updateData(); // ❌ 事务不生效
```

---

## 七、常见面试题

### Q1：Spring 事务的实现方式有哪些？

**答**：两种：
1. **编程式事务**：使用 `TransactionTemplate` 或 `PlatformTransactionManager` 手动编码控制事务
2. **声明式事务**：使用 `@Transactional` 注解或 XML 配置，基于 AOP 实现，将事务管理与业务代码解耦

> 推荐使用声明式事务，因为它更简洁、解耦更好。

---

### Q2：`@Transactional` 注解的原理是什么？

**答**：
1. Spring 在 Bean 初始化时扫描 `@Transactional` 注解
2. 如果 Bean 存在事务方法，为其创建代理对象（JDK 动态代理或 CGLIB）
3. 方法调用时，先进入 `TransactionInterceptor` 拦截器
4. 拦截器调用 `PlatformTransactionManager` 开启事务
5. 执行业务方法
6. 如果无异常则提交事务，有异常则回滚事务

---

### Q3：Spring 事务的 7 种传播行为是什么？常用的是哪几个？

**答**：
- **REQUIRED（默认）**：有则加入，无则新建
- **SUPPORTS**：有则加入，无则非事务
- **MANDATORY**：必须有事务，否则报错
- **REQUIRES_NEW**：新建事务，挂起当前事务
- **NOT_SUPPORTED**：非事务执行，挂起当前事务
- **NEVER**：必须非事务，有事务则报错
- **NESTED**：嵌套事务（Savepoint）

> 最常用的是 **REQUIRED** 和 **REQUIRES_NEW**。

---

### Q4：Spring 事务在什么情况下会失效？列举至少 5 种

**答**：
1. **非 public 方法**：`@Transactional` 只能用于 public 方法
2. **同类自调用**：通过 `this` 调用同类 `@Transactional` 方法，绕过代理
3. **异常被 catch**：异常被捕获未抛出，事务无法感知
4. **异常类型不匹配**：默认不回滚受检异常（`Exception`），需配置 `rollbackFor`
5. **数据库引擎不支持**：如 MyISAM 不支持事务
6. **非 Spring 管理的 Bean**：手动 new 的对象事务不生效
7. **传播行为设置错误**：如 `NOT_SUPPORTED`

---

### Q5：REQUIRED 和 REQUIRES_NEW 有什么区别？

**答**：

| 特性 | REQUIRED | REQUIRES_NEW |
|------|----------|--------------|
| 已有事务 | 加入当前事务 | 挂起当前事务，新建独立事务 |
| 无事务 | 新建事务 | 新建事务 |
| 回滚影响 | 与方法A同生共死 | 独立回滚，不影响方法A |

> `REQUIRES_NEW` 适用于需要独立事务的场景，如日志记录，即使主业务回滚，日志也要保存。

---

### Q6：NESTED 和 REQUIRES_NEW 有什么区别？

**答**：
- **REQUIRES_NEW**：完全独立的新事务，两个事务互不影响
- **NESTED**：外部事务的子事务，基于 Savepoint 实现；子事务回滚不影响外部事务，但外部事务回滚会导致子事务回滚

---

### Q7：Spring 事务隔离级别有哪些？MySQL 默认是什么？

**答**：
- READ_UNCOMMITTED
- READ_COMMITTED
- REPEATABLE_READ（MySQL 默认）
- SERIALIZABLE
- DEFAULT（使用数据库默认）

MySQL InnoDB 默认是 **REPEATABLE_READ**（可重复读），通过 MVCC 解决不可重复读，通过间隙锁解决幻读。

---

### Q8：Spring 默认回滚什么异常？如何修改？

**答**：
- 默认回滚：`RuntimeException` 及其子类、`Error` 及其子类
- 默认不回滚：受检异常 `Exception`

修改方式：
```java
@Transactional(rollbackFor = Exception.class)
public void method() {
    // 遇到 Exception 也会回滚
}
```

---

### Q9：什么是只读事务（readOnly = true）？有什么作用？

**答**：
只读事务表示该事务中只进行查询操作，不进行增删改。

**作用**：
1. 为数据库提供优化提示，可能跳过某些锁操作
2. 在某些场景下提升查询性能
3. Spring 可能在只读事务中使用只读连接

```java
@Transactional(readOnly = true)
public List<User> queryUsers() {
    return userDao.findAll();
}
```

---

### Q10：同一个类中，A 方法（无事务）调用 B 方法（有事务），B 的事务会生效吗？

**答**：**不会生效**。因为 A 调用 B 时是通过 `this` 引用直接调用，没有经过 Spring 的代理对象，因此 `@Transactional` 注解不会被拦截器处理。

**解决方案**：
1. 注入自身代理对象调用
2. 使用 `((UserService)AopContext.currentProxy()).methodB()`
3. 将 B 方法抽取到另一个 Service 类中

---

### Q11：Spring 事务和数据库事务有什么关系？

**答**：
Spring 事务是对数据库事务的封装：
1. Spring 通过 `PlatformTransactionManager` 管理事务
2. 最终通过 JDBC 的 `Connection.setAutoCommit(false)` 开启事务
3. 通过 `Connection.commit()` / `Connection.rollback()` 提交/回滚
4. Spring 提供了声明式事务管理，简化了数据库事务的使用

---

### Q12：事务的超时时间如何设置？有什么注意事项？

**答**：
通过 `@Transactional(timeout = 10)` 设置，单位是秒。

**注意事项**：
1. 超时时间从事务开始时计算
2. 超时后事务会被强制回滚
3. 默认 `-1` 表示不超时
4. 仅对新建事务有效（如 `REQUIRED` 加入已有事务时，超时设置不生效）

---

### Q13：Spring Boot 中如何配置全局事务属性？

**答**：
```java
@Configuration
public class TransactionConfig {
    
    @Bean
    public TransactionInterceptor transactionInterceptor(
            PlatformTransactionManager transactionManager) {
        
        Properties props = new Properties();
        // query 开头的方法只读
        props.setProperty("query*", "PROPAGATION_REQUIRED,readOnly");
        // add/update/delete 开头的方法可读写
        props.setProperty("add*", "PROPAGATION_REQUIRED");
        props.setProperty("update*", "PROPAGATION_REQUIRED");
        props.setProperty("delete*", "PROPAGATION_REQUIRED");
        
        TransactionInterceptor interceptor = new TransactionInterceptor();
        interceptor.setTransactionManager(transactionManager);
        interceptor.setTransactionAttributes(props);
        return interceptor;
    }
}
```

---

### Q14：分布式事务了解吗？Spring 如何处理？

**答**：
分布式事务是指跨多个数据库或服务的事务。

Spring 处理分布式事务的方式：
1. **JTA 事务管理器**：使用 `JtaTransactionManager`，依赖外部事务协调器（如 Atomikos）
2. **Spring + Seata**：阿里巴巴开源的分布式事务解决方案，支持 AT、TCC、Saga、XA 模式
3. **消息队列最终一致性**：通过消息队列实现异步的最终一致性
4. **Saga 模式**：长事务拆分为多个本地事务，通过补偿操作保证一致性

---

## 八、总结

| 知识点 | 要点 |
|--------|------|
| 事务管理 | 声明式优于编程式，基于 AOP |
| 传播行为 | REQUIRED（默认）、REQUIRES_NEW 最常用 |
| 隔离级别 | 理解脏读、不可重复读、幻读 |
| 失效场景 | 非 public、同类自调用、异常被吞、异常类型不匹配 |
| 实现原理 | AOP 代理 + TransactionInterceptor + PlatformTransactionManager |

理解 Spring 事务的核心在于掌握 **AOP 代理机制** 和 **事务传播行为**，同时牢记事务失效的常见场景，这在面试和实际开发中都是高频考点。
