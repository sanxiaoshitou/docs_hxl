# Java 多线程核心参数与线程执行流程

## 一、线程池 7 大核心参数

| 参数 | 说明 |
|------|------|
| **corePoolSize** | 核心线程数。即使空闲也保留在线程池中的线程数量（除非设置了 `allowCoreThreadTimeOut`） |
| **maximumPoolSize** | 最大线程数。线程池允许创建的最大线程数量 |
| **keepAliveTime** | 非核心线程的存活时间。当线程数大于核心数时，多余的空闲线程在终止前等待新任务的最长时间 |
| **unit** | `keepAliveTime` 的时间单位（如 `TimeUnit.SECONDS`） |
| **workQueue** | 任务等待队列。用于保存等待执行的任务的阻塞队列 |
| **threadFactory** | 线程工厂。用于创建新线程，可以自定义线程名称、优先级、守护状态等 |
| **handler** | 拒绝策略。当线程池和队列都满时，对新任务的处理策略 |

---

## 二、新任务提交执行流程

当调用 `execute(Runnable)` 提交新任务时，执行顺序如下：

```text
提交任务
   │
   ▼
当前线程数 < corePoolSize ?
   ├── 是 → 创建新核心线程，执行任务
   │
   └── 否 → 任务进入 workQueue
              │
              ▼
           队列是否已满？
              ├── 否 → 任务在队列中等待执行
              │
              └── 是 → 当前线程数 < maximumPoolSize ?
                            ├── 是 → 创建非核心线程，执行任务
                            │
                            └── 否 → 执行拒绝策略（handler）
```

### 具体流程说明

1. **创建核心线程**：如果当前运行线程数 **< corePoolSize**，即使有空闲线程，也会创建一个新核心线程来执行该任务。
2. **进入队列**：如果当前线程数 **≥ corePoolSize**，任务会被放入 `workQueue` 等待执行。
3. **创建非核心线程**：如果队列已满，且当前线程数 **< maximumPoolSize**，会创建额外的非核心线程来执行任务。
4. **触发拒绝策略**：如果队列已满，且线程数 **≥ maximumPoolSize**，新任务将被拒绝，执行 `RejectedExecutionHandler`。

---

## 三、常用拒绝策略

| 策略 | 说明 |
|------|------|
| `AbortPolicy` | **默认策略**，直接抛出 `RejectedExecutionException` |
| `CallerRunsPolicy` | 由调用线程（提交任务的线程）自己执行该任务 |
| `DiscardPolicy` | 静默丢弃无法处理的任务，不抛异常 |
| `DiscardOldestPolicy` | 丢弃队列中最旧的任务，然后尝试重新提交当前任务 |

---

## 四、核心参数配置建议

- **CPU 密集型**：`corePoolSize = CPU 核心数 + 1`，减少线程切换开销。
- **IO 密集型**：`corePoolSize = CPU 核心数 × 2` 或更大，因为线程经常等待 IO。
- **队列选择**：
  - `LinkedBlockingQueue`：无界队列，适用于任务量波动大的场景（注意内存溢出风险）。
  - `ArrayBlockingQueue`：有界队列，适用于需要控制队列大小的场景。
  - `SynchronousQueue`：不存储任务，直接提交给线程，适用于高吞吐、短任务场景。
