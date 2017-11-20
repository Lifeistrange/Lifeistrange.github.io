---
title: JanusGraph配置说明
date: 2017-11-20 14:27:39
categories: JanusGraph
tags: 
    - JanusGraph
    - graph
    - database
---

本文章主要对JanusGraph的配置选项做一个详细的描述。其中索引后端为es，存储后端为hbase。

## 1.可变等级

每一个设置选项都有一个明确的可变等级标识该设置在数据库启动后能否被修改。下面是可变等级的列表：

FIXED

当数据库打开后，这些设置选项就不能被改变了，在这个数据库的整个生命周期中。

GLOBAL_OFFLINE

这些选项只在整个数据库集群实例停止时可以被修改。

GLOBAL

这些选项只能在整个数据库集群上修改。

MASKABLE

这些选项是全局的，但是可以被本地的配置文件覆盖

LOCAL

这些选项只能在本地配置文件配置

可以在[Section 4.3, “Global Configuration”](http://docs.janusgraph.org/latest/configuration.html#configuration-global)查看怎样改变非本地配置选项

## 2.伞式命名空间

被星号标记的命名空间是伞式命名空间，这表示着它们可以容纳任意数量的子命名空间，每个子命名空间都通过他的名字来做唯一标识。这个伞式命名空间列出的配置选项只适用于其子命名空间。伞式命名空间用于配置多个相同类型的系统组件，因此具有相同的配置选项。

举例来说，`log`命名空间就是一个伞式命名空间因为JanusGraph可以对接多种日志后端，比如`user`日志，每个都有相同的核心配置选项。设置`user`日志一个事务记录100次，可以使用下面的设置选项：

```ini
log.user.send-batch-size = 100
```

## 3.配置命名空间与选项

### 3.1.属性自定义*

自定义属性序列化与处理

| Name                                   | Description | Datatype | Default Value      | Mutability     |
| -------------------------------------- | ----------- | -------- | ------------------ | -------------- |
| attributes.custom.[X].attribute-class  | 注册自定义属性类    | String   | (no default value) | GLOBAL_OFFLINE |
| attributes.custom.[X].serializer-class | 注册自定义属性的序列化 | String   | (no default value) | GLOBAL_OFFLINE |

### 3.2.缓存

修改JanusGraph的缓存行为配置选项

| Name                      | Description                              | Datatype | Default Value      | Mutability     |
| ------------------------- | ---------------------------------------- | -------- | ------------------ | -------------- |
| cache.db-cache            | 当该选项开启时，JanusGraph的数据库级别缓存会开启，并且为全事务共享。开启该选项会加快遍历图的速度通过在内存中的热图数据，但是也会增加读取不常用数据的可能性。关闭该选项会强制每个事务在操作前独立的从存储中读取图数据。 | Boolean  | false              | MASKABLE       |
| cache.db-cache-clean-wait | 该选项控制数据库级别的缓存会保留多长时间，单位为毫秒。这个选项只在分布式存储后端中起作用，能够使其立即写入，又不立即可见。 | Integer  | 50                 | GLOBAL_OFFLINE |
| cache.db-cache-size       | 该选项控制数据库级别的缓存大小，值在0-1之间作为JVM堆的百分比。当超过1时就会被解释为byte大小。 | Double   | 0.3                | MASKABLE       |
| cache.db-cache-time       | 默认的数据库级的缓存实体超时时间，单位为毫秒。缓存数据存在时间到达该值后就会被删除，即使仍有可用空间。当该值设置为0时，会关闭超时（缓存数据将会一直存在，直到内存压力上限） | Long     | 10000              | GLOBAL_OFFLINE |
| cache.tx-cache-size       | 最近使用的顶点的事务级的缓存的最大大小。                     | Integer  | 20000              | MASKABLE       |
| cache.tx-dirty-size       | 未提交脏顶点事务级高速缓存的初始大小。该选项对频繁写入或需求性能的事务有较大影响。 如果要设置该选项，那么它应该大致匹配每个事务的中间顶点点。 | Integer  | (no default value) | MASKABLE       |

### 3.3.集群

多机器部署配置选项

| Name                   | Description                              | Datatype | Default Value | Mutability |
| ---------------------- | ---------------------------------------- | -------- | ------------- | ---------- |
| cluster.max-partitions | 在分区的图中的最大虚拟分区数量。该选项应当比我们所期望的集群最大节点数大。该选项必须大于一并且是2的N次幂 | Integer  | 32            | FIXED      |

### 3.4.计算

图计算配置

| Name                 | Description                              | Datatype | Default Value | Mutability |
| -------------------- | ---------------------------------------- | -------- | ------------- | ---------- |
| computer.result-mode | 该选项表示图计算如何返回其计算结果，`persist`是将其写入图，`localtx`是将其写入本地事务，或者`none` （默认） | String   | none          | MASKABLE   |

### 3.5.图

| Name                            | Description                              | Datatype           | Default Value      | Mutability |
| ------------------------------- | ---------------------------------------- | ------------------ | ------------------ | ---------- |
| graph.allow-stale-config        | 是否允许本地与存储后端的以下配置包含冲突：FIXED, GLOBAL_OFFLINE, GLOBAL。这些配置通过存储后端管理全局且不会因为本地的配置不同而被覆盖。这类的冲突通常标志这配置错误。当这个选项设置为true，JanusGraph将记录这些选项的冲突，但是会继续正常使用存储后端的值来操作每个冲突选项。当该选项设置为false，JanusGraph将记录这些冲突，但是它将跑出异常并停止启动。 | Boolean            | true               | MASKABLE   |
| graph.graphname                 | 这个配置选项是一个可选项，你可以在你打开图时再提供。你所提供的这个String值将会成为你的图的名称。如果你使用ConfigurationManagament APIs，在你使用ConfiguredGraphFactory APIs时能通过该String来表示你的图。 | String             | (no default value) | LOCAL      |
| graph.set-vertex-id             | 如果启用该选项，用户来提供顶点ID，如果禁用该选项，顶点id由JanusGraph自动生成。当JanusGraph与另一个存储系统混用时生成长id会非常有用，但同时会禁用一些JanusGraph的高级功能导致数据不一致。**高级功能-使用请谨** | Boolean            | false              | FIXED      |
| graph.timestamps                | 这个选项为写入存储于索引时使用的时间戳粒度。设置整个图库集群的时间粒度。为了避免潜在的不准确，这个设置应该与后端系统匹配。一些JanusGraph存储后端声明了一个由底层服务中的设计而反应出的最佳时间粒度。当后端提供了一个最佳默认值，而该选项没有明确的在配置文件中声明，这个后端的默认被使用，并且与该配置相关的默认值被忽略。一个明确的声明会覆盖一般与后端的默认值。 | TimestampProviders | MICRO              | FIXED      |
| graph.unique-instance-id        | 唯一代表该JanusGraph实例的值。在访问相同的存储于索引后端的实例中，该值必须唯一。它可以自动的通过串联hostname，pid，与一个固定值来生成。建议不要配置该选项。 | String             | (no default value) | LOCAL      |
| graph.unique-instance-id-suffix | 当该值设置而`unique-instance-id`未设置时，JanusGraph实例的唯一标志会通过串联hostname与该值提供的数字生成。这是一个遗留选项，目前只在JVM的ManagementFactory.getRuntimeMXBean().getName()在多进程间不唯一时使用。 | Short              | (no default value) | LOCAL      |

### 3.6. gremlin

Gremlin 配置选项

| Name          | Description              | Datatype | Default Value                         | Mutability |
| ------------- | ------------------------ | -------- | ------------------------------------- | ---------- |
| gremlin.graph | 该选项为gremlin服务器所使用的图工厂实现。 | String   | org.janusgraph.core.JanusGraphFactory | LOCAL      |

### 3.7. ids

图元素id配置选项

| Name                 | Description                              | Datatype | Default Value  | Mutability     |
| -------------------- | ---------------------------------------- | -------- | -------------- | -------------- |
| ids.block-size       | 全局在chunks中保留的图元素id的大小。设置太小将会使较慢的预留请求提交被频繁阻塞。设置太高将导致在图库实例关闭时留下大部分ID都未被使用的块浪费。 | Integer  | 10000          | GLOBAL_OFFLINE |
| ids.flush            | 当设置为true时，顶点与边都会在创建时被立刻分配ID。当设置为false时，只有当事务提交后才会分配ID。在图库分区工作时必须关闭该选项。 | Boolean  | true           | MASKABLE       |
| ids.num-partitions   | 分配给顶点的分区块的数量                             | Integer  | 10             | MASKABLE       |
| ids.placement        | 顶点放置策略或者class全名                          | String   | simple         | MASKABLE       |
| ids.renew-percentage | 当最近保留的ID块只剩下该选项所标百分比时（值为0-1之间），JanusGraph异步的开始准备另一个块，这样可以帮助避免事务提交时ID保留因为块太小而阻塞。 | Double   | 0.3            | MASKABLE       |
| ids.renew-timeout    | 这个值表示JanusGraph的ID池管理等待分配新ID块的重试时间，单位为毫秒 | Duration | 120000 ms      | MASKABLE       |
| ids.store-name       | 该值为ID KCVStore 的名称。IDS_STORE_NAME仅用于向后兼容Titan，不应该在正常操作或者新图形中使用。 | String   | janusgraph_ids | GLOBAL_OFFLINE |

### 3.8. ids.authority

图库元素ID的保留与分配设置

| Name                                     | Description                              | Datatype              | Default Value | Mutability     |
| ---------------------------------------- | ---------------------------------------- | --------------------- | ------------- | -------------- |
| ids.authority.conflict-avoidance-mode    | 该设置帮助分离JanusGraph实例共享同一个图存储后端，避免当预约ID块时的争夺，提高整体吞吐量。 | ConflictAvoidanceMode | NONE          | GLOBAL_OFFLINE |
| ids.authority.conflict-avoidance-tag     | 冲突避免标记用来JanusGraph实例分配ID                 | Integer               | 0             | LOCAL          |
| ids.authority.conflict-avoidance-tag-bits | 配置JanusGraph为冲突避免标记所保留的分配的元素ID的位数        | Integer               | 4             | FIXED          |
| ids.authority.randomized-conflict-avoidance-retries | 在放弃和抛出异常之前，系统尝试使用随机冲突避免标签尝试ID块保留的次数。     | Integer               | 5             | MASKABLE       |
| ids.authority.wait-time                  | 系统等待存储后端确认ID块保留的毫秒数。                     | Duration              | 300 ms        | GLOBAL_OFFLINE |

### 3.9. index *

独立索引后端的配置选项

| Name                          | Description                              | Datatype | Default Value      | Mutability     |
| ----------------------------- | ---------------------------------------- | -------- | ------------------ | -------------- |
| index.[X].backend             | JanusGraph使用的后端索引，能够扩展与优化JanusGraph的查询功能。该设置为可选项。JanusGraph可以使用多个异构索引后端。于是，这个选项可以出现多次，只要索引与后端之间的用户定义的名称在外观上是唯一的。对于存储后端也一样，这需要设置为一个JanusGraph的内置简写标准名称（lucene, elasticsearch, es, solr）或者完整的包与一个自定义/第三方的索引提供实现的classname。 | String   | elasticsearch      | GLOBAL_OFFLINE |
| index.[X].conf-file           | 单独的索引后端配置文件路径                            | String   | (no default value) | MASKABLE       |
| index.[X].directory           | 本地存储索引文件夹                                | String   | (no default value) | MASKABLE       |
| index.[X].hostname            | 索引后端服务器hostname列表。只对es，solr等的索引后端有效      | String[] | 127.0.0.1          | MASKABLE       |
| index.[X].index-name          | 索引后端需要的索引名称                              | String   | janusgraph         | GLOBAL_OFFLINE |
| index.[X].map-name            | 是否使用属性的键来作为索引中的字段名。该选项必须明确，索引的属性的键名字必须是有效的字段名。重命名属性的键不会重命名字段，开发者有责任避免字段冲突。 | Boolean  | true               | GLOBAL         |
| index.[X].max-result-set-size | 在不声明限制返回多少条的情况下返回的最大条数。如果索引后端支持滚动，该选项则为每一批返回的数量， | Integer  | 50                 | MASKABLE       |
| index.[X].port                | 链接索引后端的接口                                | Integer  | (no default value) | MASKABLE       |

### 3.10. index.[X].elasticsearch

ES设置

| Name                                     | Description                              | Datatype | Default Value | Mutability     |
| ---------------------------------------- | ---------------------------------------- | -------- | ------------- | -------------- |
| index.[X].elasticsearch.bulk-refresh     | ES的批量API刷新设置，当请求所做的更改需要立即可见时用来刷新API      | String   | false         | MASKABLE       |
| index.[X].elasticsearch.health-request-timeout | 当JanusGraph初始化了ES后端，JanusGraph会等待ES集群的健康达到至少黄色的成都。该选项应当使用小写的“s”，比如3s或者60s | String   | 30s           | MASKABLE       |
| index.[X].elasticsearch.interface        | 链接ES的接口。TRANSPORT_CLIENT和NODE以前也支持，但是现在迁移至REST_CLIENT，详情请看JanusGraph的更新介绍 | String   | REST_CLIENT   | MASKABLE       |
| index.[X].elasticsearch.scroll-keep-alive | es保持多长时间的scroll上下文，单位为秒                  | Integer  | 60            | GLOBAL_OFFLINE |
| index.[X].elasticsearch.use-all-field    | JanusGraph是否需要把`all`字段加入`mapping`。当该选项开启，`mapping`将会引入一个`copy_to`参数代表`all`字段。该字段从es6.x后开始使用，当在es6.x使用通配符字段时是必须的。 | Boolean  | true          | GLOBAL_OFFLINE |
| index.[X].elasticsearch.use-deprecated-multitype-index | JanusGraph是否应该将这些索引组合到一个单独的es索引中。（es5.x与之前的版本支持） | Boolean  | false         | GLOBAL_OFFLINE |

### 3.11. index.[X].elasticsearch.create

创建索引设置

| Name                                     | Description                              | Datatype | Default Value | Mutability |
| ---------------------------------------- | ---------------------------------------- | -------- | ------------- | ---------- |
| index.[X].elasticsearch.create.sleep     | 在初次完成一个（阻塞的）索引创建与第一次使用该索引间的最小时间间隔，单位为毫秒。该选项只适用于第一次创建一个索引，通常是第一次在ES上启动JanusGraph。如果JanusGraph设置的索引已经存在，这个选项就不会造成任何影响。 | Long     | 200           | MASKABLE   |
| index.[X].elasticsearch.create.use-external-mappings | 是否应该使用一个额外的`mapping`当注册index.JanusGraph时 | Boolean  | false         | MASKABLE   |

### 3.12. log *

JanusGraph的日志系统配置选项

| Name                    | Description                              | Datatype | Default Value      | Mutability     |
| ----------------------- | ---------------------------------------- | -------- | ------------------ | -------------- |
| log.[X].backend         | 定义要用的日志后端                                | String   | default            | GLOBAL_OFFLINE |
| log.[X].fixed-partition | 是否即使后端存储分区，日志也都写在一个固定的分区上。这可能造成负载不均衡，应当只在少量的日志时使用。 | Boolean  | false              | GLOBAL_OFFLINE |
| log.[X].key-consistent  | 在存储后端读写日志信息时是否要求一致性。                     | Boolean  | false              | MASKABLE       |
| log.[X].max-partitions  | 日志使用的最大分区数。该设置真实与虚拟分区都会使用。必须大于0且为2的n次幂。  | Integer  | (no default value) | FIXED          |
| log.[X].max-read-time   | 从后端读取日志信息的最长等待时间，单位为毫秒。                  | Duration | 4000 ms            | MASKABLE       |
| log.[X].max-write-time  | 往后端持久化存储的最长等待时间，单位为毫秒。                   | Duration | 10000 ms           | MASKABLE       |
| log.[X].num-buckets     | 为了负载均衡把日志分拆成的块的数量                        | Integer  | 1                  | GLOBAL_OFFLINE |
| log.[X].read-batch-size | 一次读取日志信息的最大数量，通过日志后端实现的批量读取。             | Integer  | 1024               | MASKABLE       |
| log.[X].read-interval   | 批量读取日志的时间间隔，单位为毫秒                        | Duration | 5000 ms            | MASKABLE       |
| log.[X].read-lag-time   | 从后端读取日志使用的最大时间，单位为毫秒。如果一次写入操作在该时间内为变为可视的，那么日志读取器就会放弃该信息。 | Duration | 500 ms             | MASKABLE       |
| log.[X].read-threads    | 读取日志使用的进程数。                              | Integer  | 1                  | MASKABLE       |
| log.[X].send-batch-size | 对于支持批量传输的日志后端批量发送信息的最大数量。                | Integer  | 256                | MASKABLE       |
| log.[X].send-delay      | 日志信息在批量发送前最大暂存时间，单位为毫秒。                  | Duration | 1000 ms            | MASKABLE       |
| log.[X].ttl             | 给所有日志实体设置TTL，这意味着所有的日志都有了一个到期时间。需要日志后端实现TTL。 | Duration | (no default value) | GLOBAL         |

### 3.13. metrics

指标报告设置选项 

| Name                 | Description                 | Datatype | Default Value  | Mutability |
| -------------------- | --------------------------- | -------- | -------------- | ---------- |
| metrics.enabled      | 是否启用后端的基本定时和操作计数监视          | Boolean  | false          | MASKABLE   |
| metrics.merge-stores | 是否为边缘存储、顶点索引、边缘索引和id存储做聚合测量 | Boolean  | true           | MASKABLE   |
| metrics.prefix       | 报告的janusgraph指标的默认名称前缀。     | String   | org.janusgraph | MASKABLE   |

### 3.14. metrics.console

控制台显示的指标报告设置选项

| Name                     | Description     | Datatype | Default Value      | Mutability |
| ------------------------ | --------------- | -------- | ------------------ | ---------- |
| metrics.console.interval | 控制台报告时间间隔，单位为毫秒 | Duration | (no default value) | MASKABLE   |

### 3.15. metrics.csv

输出到CSV文件的指标报告设置选项

| Name                  | Description     | Datatype | Default Value      | Mutability |
| --------------------- | --------------- | -------- | ------------------ | ---------- |
| metrics.csv.directory | 输出CSV文件夹路径      | String   | (no default value) | MASKABLE   |
| metrics.csv.interval  | 输出CSV时间间隔，单位为毫秒 | Duration | (no default value) | MASKABLE   |

### 3.16. query

查询程序的设置选项

| Name                           | Description                              | Datatype | Default Value | Mutability |
| ------------------------------ | ---------------------------------------- | -------- | ------------- | ---------- |
| query.batch                    | 在使用存储后端时，是否批量执行遍历查询。在后端有长延迟时能够有效提升查询效率。  | Boolean  | false         | MASKABLE   |
| query.fast-property            | 是否在第一个顶点属性访问中预读取所有属性。在检索顶点的所有属性后查询同样的顶点的属性时，该选项可以消除对同一顶点的后端调用。 | Boolean  | true          | MASKABLE   |
| query.force-index              | JanusGraph是否应该在查询没有使用索引时抛出异常。这样做限制了JanusGraph图查询的部分功能，但是可以避免在巨大的图中慢查询。推荐在生产中使用该选项。 | Boolean  | false         | MASKABLE   |
| query.ignore-unknown-index-key | 是否忽略未定义的类型在用户提供的索引查询中。                   | Boolean  | false         | MASKABLE   |
| query.smart-limit              | Whether the query optimizer should try to guess a smart limit for the query to ensure responsiveness in light of possibly large result sets. Those will be loaded incrementally if this option is enabled.是否在可能查询出一个巨大的结果集时尝试智能限制返回条数。如果启用此选项，将缓慢加载该选项。 | Boolean  | true          | MASKABLE   |

### 3.17. schema

schema相关配置选项

| Name           | Description                              | Datatype | Default Value | Mutability |
| -------------- | ---------------------------------------- | -------- | ------------- | ---------- |
| schema.default | 设置在图中使用DefaultSchemaMaker。如果设置为`none`，将会关闭自动图表创建。默认使用`blueprints`的兼容表创建器，多标签的边与单键的属性。 | String   | default       | MASKABLE   |

### 3.18. storage

存储后端的设置。有一些设置只针对对应的后端。

| Name                         | Description                              | Datatype | Default Value      | Mutability |
| ---------------------------- | ---------------------------------------- | -------- | ------------------ | ---------- |
| storage.backend              | JanusGraph初始持久化提供者。该选项为必填。该选项可以设置为在JanusGraph中的内置短语来表示标准存储后端（berkeleyje, cassandrathrift, cassandra, astyanax, embeddedcassandra, cql, hbase, inmemory）或者完整的包与一个自定义/第三方的存储提供实现的classname。 | String   | (no default value) | LOCAL      |
| storage.batch-loading        | 是否打开批量读取                                 | Boolean  | false              | LOCAL      |
| storage.buffer-size          | 改动持久化的批量大小。                              | Integer  | 1024               | MASKABLE   |
| storage.conf-file            | 单独的存储后端配置文件路径                            | String   | (no default value) | LOCAL      |
| storage.connection-timeout   | 链接远端数据库实例默认超时时间，单位为毫秒。                   | Duration | 10000 ms           | MASKABLE   |
| storage.directory            | 存储后端需要的本地存储的本地文件夹路径。                     | String   | (no default value) | LOCAL      |
| storage.drop-on-clear        | 在删除存储时是drop掉图数据库（true）还是删除所有行（false）。要注意，有些存储后端在删除存储时只能drop数据库。还需要注意，在清理存储时索引也会被删掉。 | Boolean  | true               | MASKABLE   |
| storage.hostname             | 存储后端服务器hostname列表。该选项只对一些存储后端有效，如cassandra，hbase | String[] | 127.0.0.1          | LOCAL      |
| storage.page-size            | JanusGraph中断请求会返回一些从分布式存储后端的一系列请求的小块的结果，每一块都包含一些元素。 | Integer  | 100                | MASKABLE   |
| storage.parallel-backend-ops | JanusGraph是否应该尝试去并行存储操作。                 | Boolean  | true               | MASKABLE   |
| storage.password             | 存储后端的验证密码                                | String   | (no default value) | LOCAL      |
| storage.port                 | 存储后端的链接端口                                | Integer  | (no default value) | LOCAL      |
| storage.read-only            | 只读数据库                                    | Boolean  | false              | LOCAL      |
| storage.read-time            | 后端成功完成读操作的最长等待时间，单位为毫秒。如果后端读操作暂时失败了。JanusGraph将会重试该操作，直到等待时间耗尽。 | Duration | 10000 ms           | MASKABLE   |
| storage.root                 | 存储后端需要的本地存储根目录。如果你不填写`storage.directory`而填写`graph.graphname`，你的数据将会存储在`<STORAGE_ROOT>/<GRAPH_NAME>`目录。 | String   | (no default value) | LOCAL      |
| storage.setup-wait           | 存储后端在服务器模式下启动后，后端管理等待存储后端可用的时间。          | Duration | 60000 ms           | MASKABLE   |
| storage.transactions         | 在支持事务的存储后端启用事务。                          | Boolean  | true               | MASKABLE   |
| storage.username             | 存储后端的验证用户名。                              | String   | (no default value) | LOCAL      |
| storage.write-time           | 存储后端成功完成写操作的最大等待时间。如果存储后端写失败了，JanusGraph会重试该操作直到时间耗尽。 | Duration | 100000 ms          | MASKABLE   |

### 3.19. storage.hbase

Hbase存储设置。

| Name                                | Description                              | Datatype | Default Value      | Mutability |
| ----------------------------------- | ---------------------------------------- | -------- | ------------------ | ---------- |
| storage.hbase.compat-class          | HBaseCompat实现的包与classname。HBaseCompat用于忽略与HBase API的版本不同。当该选项未设置时，JanusGraph在运行时会调用HBase的`VersionInfo.getVersion()`并读取相应的类。设置该选项强制JanusGraph使用实例化特定类的方式替代反射。 | String   | (no default value) | MASKABLE   |
| storage.hbase.compression-algorithm | 一个HBase的压缩算法。枚举字符串将会被在列族创建。压缩算法会在HBase集群安装与应用。JanusGraph不能自己在HBase安装设置新压缩算法。 | String   | GZ                 | MASKABLE   |
| storage.hbase.region-count          | 创建JanusGraph的HBase表时初始化区域数量。             | Integer  | (no default value) | MASKABLE   |
| storage.hbase.regions-per-server    | 创建JanusGraph的HBase表时初始化每个区域服务器的区域数量。     | Integer  | (no default value) | MASKABLE   |
| storage.hbase.short-cf-names        | 是否缩短JanusGraph的列族名称至一个字符来节约存储空间。         | Boolean  | true               | FIXED      |
| storage.hbase.skip-schema-check     | 假设JanusGraph的HBase表与列族已经存在。当该选项为`true`时，JanusGraph将不会检查已存在的表与列族，也不会尝试在任何情况下创建他们。这在JanusGraph没有HBase的管理员权限时很有用。 | Boolean  | false              | MASKABLE   |
| storage.hbase.table                 | JanusGraph的表名。当`storage.hbase.skip-schema-check`为`false`时，JanusGraph会自动创建不存在的表。如果该选项未填但`graph.graphname`有值时，表名会使用`graph.graphname`。 | String   | janusgraph         | LOCAL      |

### 3.20. storage.lock

最终一致性的锁的存储。

| Name                              | Description                              | Datatype | Default Value      | Mutability     |
| --------------------------------- | ---------------------------------------- | -------- | ------------------ | -------------- |
| storage.lock.backend              | 使用的锁的类型                                  | String   | consistentkey      | GLOBAL_OFFLINE |
| storage.lock.clean-expired        | 是否从存储后端删除超时的锁                            | Boolean  | false              | MASKABLE       |
| storage.lock.expiry-time          | 一个锁从使用到过期的时间。未发布的锁会在超时后失效并发布。这个值应当比最长的事务时间要大，以确保未正确获得锁的应用在完成前到期且要足够小以减小死锁概率。 | Duration | 300000 ms          | GLOBAL_OFFLINE |
| storage.lock.local-mediator-group | 该选项使用LocalLockMediator实例在两个并行的JanusGraph图实例在相同的进程中同时链接到相同的存储后端时的早期检测。对于有相同值的该选项的JanusGraph实例，将会在内存中尝试去发现他们之间的锁争夺，在开始正常的分布式锁步骤前。JanusGraph一开始会为该选项生成一个适当的默认值。修改这个默认项只有在测试的时候有用。 | String   | (no default value) | LOCAL          |
| storage.lock.retries              | 系统尝试获取锁的次数，如果失败会跑出异常。                    | Integer  | 3                  | MASKABLE       |
| storage.lock.wait-time            | 系统等待从存储后端获取回应的时间，单位为毫秒。同样也是在验证应用程序成功之前，等待所有锁应用的时间。这个值应当是平均一直写入时间的小倍数。 | Duration | 100 ms             | GLOBAL_OFFLINE |

### 3.21. storage.meta *

包含在存储后端的元数据检索配置

| Name                        | Description                              | Datatype | Default Value | Mutability |
| --------------------------- | ---------------------------------------- | -------- | ------------- | ---------- |
| storage.meta.[X].timestamps | 是否在检索中包含时间戳，如果存储后端自动在实体上注释时间戳。           | Boolean  | false         | GLOBAL     |
| storage.meta.[X].ttl        | Whether to include ttl in retrieved entries for storage backends that support storage and retrieval of cell level TTL是否在检索中包含ttl，如果存储后端支持存储于检索字段级别的ttl | Boolean  | false         | GLOBAL     |
| storage.meta.[X].visibility | Whether to include visibility in retrieved entries for storage backends that support cell level visibility是否在检索中包含能见度，如果存储后端支持字段级别的能见度 | Boolean  | true          | GLOBAL     |

### 3.22. tx

事务处理的配置选项

| Name               | Description                              | Datatype | Default Value | Mutability |
| ------------------ | ---------------------------------------- | -------- | ------------- | ---------- |
| tx.log-tx          | 是否事务变动会被日志记录到JanusGraph的预写事务日志中，该操作可能用来还原部分失败的事务。 | Boolean  | false         | GLOBAL     |
| tx.max-commit-time | 一个事务尝试去往所有后端提交的最大尝试时间，单位为毫秒。该选项对分布式的预写日志程序在一个事务失败时（比如超时了）做决定有所帮助。该选项必须比允许的最大写入时间要长。 | Duration | 10000 ms      | GLOBAL     |

### 3.23. tx.recovery

事务恢复过程的配置选项

| Name                | Description                   | Datatype | Default Value | Mutability |
| ------------------- | ----------------------------- | -------- | ------------- | ---------- |
| tx.recovery.verbose | 事务会务系统是否应该打印恢复的事务与其他活动在标准输出中。 | Boolean  | false         | MASKABLE   |

> 原文翻译自[Chapter 13. Configuration Reference](http://docs.janusgraph.org/latest/config-ref.html)
