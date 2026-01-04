## DDD 领域驱动
DDD的CQRS模型

### 一、


### 二、
DDD的CQRS模型这种：src/main/java      
├── com.example.order       
├── application # 应用层(整体流程控制)       
│ ├── service       
│ └── dto       
├── domain # 领域层（核心）        
│ ├── model # 实体、值对象        
│ ├── service # 领域服务 (逻辑核心)     
│ ├── event # 领域事件      
│ └── repository # 仓储接口     
├── infrastructure # 基础设施层      
│ ├── persistence # 数据库实现       
│ ├── message # 消息队列        
│ └── external # 外部服务调用     
└── interface/controller# 用户接口层     
├── web # REST API      
└── rpc # gRPC/Dubbo        