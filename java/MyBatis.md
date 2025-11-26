## MyBatis 


### MyBatis 拦截器
1、实现接口：Interceptor
MyBatis的拦截器机制基于JDK动态代理，所有自定义拦截器都要实现Interceptor接口
```java
public interface Interceptor {
// 拦截逻辑的核心方法
Object intercept(Invocation invocation) throws Throwable;
// 生成代理对象（通常直接用Plugin.wrap()）
Object plugin(Object target);
// 读取配置参数（如从mybatis-config.xml中获取）
void setProperties(Properties properties);
}
```
2、拦截目标与签名配置
MyBatis允许拦截4个核心组件的方法，通过@Intercepts和@Signature注解指定拦截目标
Executor           SQL执行器（最常用）  update、query、commit、rollback      
StatementHandler   SQL语句处理器（控制SQL生成）  prepare、parameterize    
ParameterHandler   参数处理器（处理SQL参数） setParameters      
ResultSetHandler   结果集处理器（处理查询结果） handleResultSets     


示例：
