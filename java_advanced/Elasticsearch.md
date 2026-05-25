# Elasticsearch 知识点总结

> 本文档整理了 Elasticsearch（简称 ES）的核心概念、常用操作以及常见面试题，适用于面试复习与技术交流。

---

## 一、ES 基础概念

### 1.1 什么是 Elasticsearch？
Elasticsearch 是一个基于 **Apache Lucene** 构建的开源**分布式搜索与分析引擎**，具备以下特点：
- **近实时搜索（NRT）**：写入数据后通常 1 秒内可搜索
- **分布式**：数据自动分片（Shard）与副本（Replica），水平扩展能力强
- **RESTful API**：通过 HTTP 接口进行 CRUD、搜索与分析操作
- **Schema-Free**：默认自动推断字段类型，也支持显式定义 Mapping
- **高可用**：自动故障转移，主分片与副本分片机制

### 1.2 核心术语对比（ES vs 关系型数据库）

| RDBMS | Elasticsearch |
| :--- | :--- |
| Database | **Index（索引）** |
| Table | **Type（类型，7.x 已废弃）** |
| Row | **Document（文档）** |
| Column | **Field（字段）** |
| Schema | **Mapping（映射）** |
| Index | **Inverted Index（倒排索引）** |
| SQL | **DSL（查询语言）** |

### 1.3 核心组件

- **Cluster（集群）**：由一个或多个节点组成，共同持有全部数据
- **Node（节点）**：单个 ES 实例，分为 Master、Data、Ingest、Coordinating 等角色
- **Index（索引）**：逻辑上类似数据库，是文档的集合
- **Shard（分片）**：索引的数据被切分为多个分片，每个分片是一个独立的 Lucene 索引
- **Replica（副本）**：分片的复制，提供数据冗余与读性能扩展

---

## 二、倒排索引原理

### 2.1 什么是倒排索引？
倒排索引（Inverted Index）是 ES 实现快速全文检索的核心数据结构。

**示例**：假设有两篇文档：
- Doc1: "I love Elasticsearch"
- Doc2: "Elasticsearch is powerful"

**倒排索引结构**：

| Term | Doc IDs |
| :--- | :--- |
| Elasticsearch | Doc1, Doc2 |
| love | Doc1 |
| powerful | Doc2 |

### 2.2 索引构建过程
1. **分词（Analysis）**：将文档内容拆分为 Term
2. **去重与排序**：对所有 Term 进行排序
3. **建立映射**：记录每个 Term 出现在哪些文档中
4. **记录位置信息**：支持短语查询、高亮等功能

---

## 三、分片与集群架构

### 3.1 分片（Shard）机制
- **主分片（Primary Shard）**：数据写入的目标分片，数量在创建索引时确定，**不可更改**
- **副本分片（Replica Shard）**：主分片的拷贝，可动态调整数量，默认 1 个副本

**分片分配原则**：
- 主分片与副本分片不会分配到同一节点
- 分片数量建议：单分片大小控制在 **20GB ~ 50GB**
- 分片数并非越多越好，过多会导致集群开销增大

### 3.2 节点角色

| 节点类型 | 作用 |
| :--- | :--- |
| **Master Node** | 负责集群状态管理、元数据变更（如创建/删除索引） |
| **Data Node** | 存储数据，执行 CRUD、搜索、聚合操作 |
| **Ingest Node** | 预处理文档（如字段转换、数据增强），执行 Pipeline |
| **Coordinating Node** | 接收客户端请求，转发到数据节点并汇总结果 |

### 3.3 集群状态
- **Green**：所有主分片与副本分片均正常分配
- **Yellow**：所有主分片已分配，但至少有一个副本分片未分配
- **Red**：至少有一个主分片未分配，部分数据不可用

---

## 四、Mapping 与字段类型

### 4.1 动态 Mapping vs 显式 Mapping
- **动态 Mapping**：写入文档时自动推断字段类型（如字符串推断为 `text` + `keyword`）
- **显式 Mapping**：手动定义字段类型，可精确控制索引行为

### 4.2 常见字段类型

| 类型 | 说明 |
| :--- | :--- |
| `text` | 全文检索字段，会被分词 |
| `keyword` | 精确匹配字段，不分词，适用于聚合、排序、过滤 |
| `integer` / `long` | 整数类型 |
| `float` / `double` | 浮点类型 |
| `date` | 日期类型，支持多种格式 |
| `boolean` | 布尔类型 |
| `object` | 嵌套对象 |
| `nested` | 独立索引的嵌套对象，解决 object 数组的跨对象匹配问题 |

### 4.3 多字段（Multi-fields）
```json
{
  "properties": {
    "title": {
      "type": "text",
      "analyzer": "ik_max_word",
      "fields": {
        "keyword": {
          "type": "keyword"
        }
      }
    }
  }
}
```
- `title`：用于全文搜索（分词）
- `title.keyword`：用于精确匹配、排序、聚合

---

## 五、分词器（Analyzer）

### 5.1 分析器组成
一个分析器（Analyzer）由三部分组成：
1. **Character Filters**：字符过滤（如去除 HTML 标签、替换字符）
2. **Tokenizer**：分词器（将文本切分为 Token）
3. **Token Filters**：Token 过滤器（如小写转换、停用词过滤、同义词处理）

### 5.2 内置分析器
- **standard**：默认分析器，按单词边界切分，去除标点
- **simple**：按非字母字符切分，小写处理
- **whitespace**：仅按空格切分
- **keyword**：不分词，整体作为一个 Token

### 5.3 中文分词插件
- **IK 分词器**：最常用的中文分词插件
  - `ik_max_word`：细粒度模式，尽可能切分出更多词
  - `ik_smart`：智能模式，按语义切分，避免无意义组合

---

## 六、查询 DSL

### 6.1 Query vs Filter
| 特性 | Query | Filter |
| :--- | :--- | :--- |
| 是否算分 | **是**（计算相关性得分） | **否**（不计算得分） |
| 缓存 | 不缓存 | **可缓存**（Bitset 缓存） |
| 适用场景 | 全文检索、相关性排序 | 精确匹配、范围过滤、布尔条件 |

### 6.2 常见查询类型

**全文查询（Full-text queries）**：
- `match`：对查询字符串分词后执行布尔查询
- `match_phrase`：短语匹配，要求 Token 顺序一致
- `multi_match`：在多个字段上执行 match
- `query_string`：支持 Lucene 语法的查询

**精确查询（Term-level queries）**：
- `term`：精确匹配单个 Term（不分词）
- `terms`：精确匹配多个 Term
- `range`：范围查询（gt, gte, lt, lte）
- `exists` / `missing`：字段是否存在

**复合查询（Compound queries）**：
- `bool`：组合多个查询条件（must、should、must_not、filter）

### 6.3 bool 查询结构
```json
{
  "query": {
    "bool": {
      "must": [ { "match": { "title": "Elasticsearch" } } ],
      "filter": [ { "term": { "status": "published" } } ],
      "must_not": [ { "match": { "title": "deprecated" } } ],
      "should": [ { "match": { "tag": "tutorial" } } ]
    }
  }
}
```

---

## 七、聚合分析（Aggregations）

### 7.1 聚合类型
- **Metric Aggregations**：数值计算
  - `avg`、`sum`、`max`、`min`、`stats`、`cardinality`（去重计数）
- **Bucket Aggregations**：分组聚合
  - `terms`（按字段值分组）、`range`（范围分组）、`date_histogram`（时间直方图）
- **Pipeline Aggregations**：基于其他聚合结果进行二次计算

### 7.2 示例：按状态统计数量
```json
{
  "aggs": {
    "status_stats": {
      "terms": { "field": "status" }
    }
  }
}
```

---

## 八、性能优化

### 8.1 写入优化
1. **使用 Bulk API** 批量写入，减少网络往返
2. **调整 Refresh Interval**：从默认 1s 调大（如 30s），减少段刷新频率
3. **禁用副本（Replica）**：大量导入数据时临时设置 `number_of_replicas: 0`
4. **使用 `_bulk` + 多线程**：提升写入吞吐量
5. **合理设置 Mapping**：避免动态 Mapping 带来的性能开销

### 8.2 查询优化
1. **使用 Filter 替代 Must**：Filter 不计算得分且可缓存
2. **避免深度分页**：使用 `search_after` 或 `scroll` 替代 `from + size`
3. **控制返回字段**：使用 `_source` 过滤，减少数据传输
4. **预热索引**：对常用查询进行预热
5. **合理设计分片数**：避免分片过多或过少

### 8.3 深度分页问题
- `from + size`：默认限制 `index.max_result_window = 10000`，深度分页消耗大量内存
- `scroll`：创建快照进行批量拉取，适合导出大量数据，但不适合实时查询
- `search_after`：基于上一页最后一条记录排序值获取下一页，适合实时翻页

---

## 九、常见面试题

### Q1：Elasticsearch 和 MySQL 的区别是什么？
**答**：
- ES 是基于 Lucene 的**搜索引擎**，适合全文检索、日志分析、实时搜索；MySQL 是**关系型数据库**，适合事务型业务、复杂关联查询。
- ES 采用**倒排索引**实现快速检索；MySQL 使用 B+Tree 索引。
- ES **天生分布式**，分片与副本机制支持水平扩展；MySQL 扩展通常依赖分库分表。
- ES 对**事务支持弱**（不支持 ACID 事务），而 MySQL 支持完整的事务机制。

---

### Q2：什么是倒排索引？为什么比正排索引快？
**答**：
倒排索引以**Term 为 Key**，记录该 Term 出现在哪些文档中。搜索时直接定位包含关键词的文档列表，时间复杂度接近 O(1)。
正排索引以**文档 ID 为 Key**，需要遍历所有文档逐一匹配，效率极低。

---

### Q3：ES 的文档写入流程是怎样的？
**答**：
1. 客户端向 Coordinating Node 发送写请求
2. Coordinating Node 根据 `_routing` 计算目标主分片
3. 数据写入主分片（先写内存缓冲区和 Translog）
4. 主分片将数据同步到副本分片
5. 主分片确认所有副本写入成功后，返回成功响应
6. 内存缓冲区数据在 Refresh（默认 1s）后生成新的段（Segment），变为可搜索
7. Flush 操作将 Segment 写入磁盘并清空 Translog

---

### Q4：Refresh、Flush、Merge 的区别？
**答**：
- **Refresh**：将内存缓冲区中的数据生成新的 Segment，变为可搜索，但**不持久化到磁盘**。默认 1s 执行一次。
- **Flush**：将 Segment **持久化到磁盘**，并清空 Translog。默认 30min 或 Translog 达到阈值时触发。
- **Merge**：后台自动将多个小 Segment **合并为大 Segment**，减少文件句柄和查询开销，同时删除被标记为删除的文档。

---

### Q5：为什么主分片数量创建后不能修改？
**答**：
因为 ES 通过 **hash(_routing) % 主分片数** 来确定文档所属分片。如果主分片数改变，所有文档的哈希结果都会变化，导致数据路由错误，需要**全量重建索引**。

---

### Q6：`term` 查询和 `match` 查询有什么区别？
**答**：
- `term`：**不分词**，直接精确匹配字段中的 Term。适用于 `keyword`、`integer`、`date` 等类型。
- `match`：**先对查询字符串分词**，然后对每个 Term 执行布尔查询（默认 OR）。适用于 `text` 类型。
- 常见坑：对 `text` 字段使用 `term` 查询通常匹配不到结果，因为 `text` 字段已被分词。

---

### Q7：ES 如何实现高可用？
**答**：
1. **副本机制**：每个主分片有至少一个副本分片，数据冗余防止丢失
2. **主节点选举**：Master Node 故障时自动选举新的 Master
3. **故障转移**：节点宕机时，该节点上的主分片在其他节点提升副本为主分片
4. **集群状态监控**：通过 `_cluster/health` 实时监控集群健康状态

---

### Q8：如何处理 ES 的深度分页问题？
**答**：
- **避免使用** `from + size` 进行深度分页（默认最大 10000 条）
- **`scroll`**：创建快照，适合批量导出数据，但不适合实时翻页
- **`search_after`**：基于上一页的排序值进行分页，适合实时翻页场景，无上限限制

---

### Q9：ES 数据一致性如何保证？
**答**：
ES 采用 **quorum 机制**（多数派确认）：
- 写入操作需要大部分分片副本（`quorum = int(primary + number_of_replicas / 2) + 1`）确认后才返回成功
- 通过 `wait_for_active_shards` 参数控制写入时要求的活动分片数
- 读取时可通过 `preference=_primary` 优先从主分片读取最新数据

---

### Q10：为什么要避免索引过大的 Mapping？
**答**：
- Mapping 中字段过多（尤其是动态 Mapping 产生大量无意义字段）会导致 **Cluster State 膨胀**
- Cluster State 变更需要在所有节点同步，过大会导致 Master 节点压力增大、集群响应变慢
- 建议设置 `index.mapping.total_fields.limit` 限制字段数量，并关闭不需要索引的字段（`index: false`）

---

### Q11：`nested` 类型和 `object` 类型有什么区别？
**答**：
- `object`：将嵌套对象扁平化处理，数组中的对象字段会**交叉匹配**（如 `[{name: "A", age: 20}, {name: "B", age: 30}]` 查询 `name:A AND age:30` 会误匹配）
- `nested`：将每个嵌套对象作为**独立的隐藏文档**索引，保持对象内部字段的关联性，避免交叉匹配问题

---

### Q12：如何提升 ES 集群的写入性能？
**答**：
1. 使用 **Bulk API** 批量写入
2. 适当增大 **Refresh Interval**（如 30s）
3. 大批量导入时临时设置 **`number_of_replicas: 0`**
4. 使用多线程并发写入
5. 预定义 Mapping，禁用 `_all` 字段
6. 使用 **Ingest Node** 预处理，减少客户端计算

---

## 十、常用 API 速查

```bash
# 查看集群健康
GET /_cluster/health

# 查看索引列表
GET /_cat/indices?v

# 创建索引
PUT /my_index
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1
  },
  "mappings": {
    "properties": {
      "title": { "type": "text" },
      "status": { "type": "keyword" }
    }
  }
}

# 写入文档
POST /my_index/_doc/1
{ "title": "Hello ES", "status": "active" }

# 搜索文档
GET /my_index/_search
{
  "query": {
    "match": { "title": "hello" }
  }
}

# 删除索引
DELETE /my_index
```

---

> 本文档持续更新中，如有遗漏或错误，欢迎补充指正。
