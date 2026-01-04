## prometheus 远程存储方案
### 一、概述
Prometheus默认将数据储存在本地的TSDB(时序数据库&)中,这种设计较大地简化了Promethes的部署
难度,但与此同时也存在着一些问题。 

首先是数据持久化 的问题,原生的TSDB对于大数据量的保存及重查询支持不太友好,并不适用于保存长期的
大量数据;另外,该数据库的可靠性也较弱,在使用过程中容易易出现数据损坏等故障,且无法支持集群的架
构。

为了满足这方面的需求,Prometheus提供了remote_write和rennote_read的特性,支持将数据存储到远端和从
远端读取数据的功能。当配置remote_write特性后,Prometheus会将采集到的指标数据通过HTTP的形式发送
给适配器(Adaptor),由适配器进行数据的存入。而remote_read特性则会向适配器发起查询请求,适配器根
据请求条件从第三方存储服务中获取响应的数据。   

在Prometheus社区中，目前已经有不少远程存储的支持方案，下面列出了其中的部分方案，完整内容可参见官网。     
AppOptics: write
Chronix: write      
Cortex: read and write      
CrateDB: read and write     
Elasticsearch: write        
Gnocchi: write      
Graphite: write     
InfluxDB: read and write        
Kafka: write        
OpenTSDB: write     
PostgreSQL/TimescaleDB: read and write      
Splunk: read and write      
在这些解决方案中，有些只支持写入操作，不支持读取，有些则支持完整的读写操作。

### 二、实践方案
参考文献：


PostgreSQL 方案：
https://cloud.tencent.com/developer/article/1530267
Tsdb 方案：
https://cloud.tencent.com/developer/article/1400412