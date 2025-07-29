## ThreadLocal 学习整理

### 一、ThreadLocal
ThreadLocal提供了线程局部变量,每个线程都可以通过get/set访问自己独立的变量副本。


### 二、InheritableThreadLocal
InheritableThreadLocal可以解决父子线程间值传递的问题

InheritableThreadLocal局限性:    
只支持创建新线程时的值传递, 线程池场景下不适用(线程复用)

### 三、TransmittableThreadLocal
#### 1、基础介绍：
TransmittableThreadLocal (TTL)是阿里开源的一个线程间数据传递解决方案,解决了
InheritableThreadLocal在线程池场景下的问题。    

核心特性：    
支持线程池场景下的值传递    
支持任务执行前的自定义逻辑    
支持任务执行后的自定义逻辑       
兼容InheritableThreadLocal

#### 2、TransmittableThreadLocal原理
1）、核心类结构
TransmittableThreadLocal:继承自InheritableThreadLocal      
TtlRunnable/TtlCallable:装饰器模式包装Runnable和CCallable       
Transmitter:提供capture/replay/restore机制    

2）、工作原理     
TTL的核心思想是"捕获-传递-恢复":    
1.捕获(Capture):在任务提交到线程池时,捕获当前线程的所有TTL变量     
2.传递(Transmit):将捕获的值传递给线程池中的线程      
3.恢复(Replay):线程池中的线程在执行任务前,将TTL值恢复      
4.回滚(Restore):任务执行完成后,恢复线程原来的TTL值  

### 3、

