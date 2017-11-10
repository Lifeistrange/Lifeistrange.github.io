---
title: Titan的超级节点解决方案 - A Solution to the Supernode Problem
date: 2017-11-09 16:32:24
tags: 
    - Janus
    - Titan
    - graph
    - database
---
在图论中和网络科学中，一个“超级节点“是指一个顶点有着不成比例的巨大数量的相互独立的边，即使超级节点在自然的图中很少出现（如统计学证明的幂律分布），他们却会在图的分析中多次出现。出现这种现象是因为超级节点关联了太多的其他顶点，以至于它们在图中存在于数量巨大的路径上。因此，任意的遍历都有可能会接触到超级节点。在图计算中，超级节点会导致一系列的性能问题。幸运的是，在属性图中，我们有一个理论且实际的解决方案。

## 现实生活中的超级节点

### P2P文件共享

在2000年左右，线上文件分享主要通过[Napster](http://en.wikipedia.org/wiki/Napster) 或者 [Gnutella](http://en.wikipedia.org/wiki/Gnutella)这类型的服务提供。和Napster不通的是，Gnutella是一个真实的P2P系统，它没有中心文件索引，取而代之的是客户端将搜索发送给调度端来完成。如果这些客户端没有你要找的文件，这个请求就会发送给调度端，然后重复查找过程。这就形成了一个自然图谱，一个超级节点仅有几部之遥。因此，在许多的P2P网络中，超级节点端很快会被大量的搜索请求淹没，就像是一个DoS攻击出现了。

### 社交网络名人

奥巴马总统目前在Twitter上已经拥有了21,322,866个followers。当奥巴马发了tweets时，这条tweet必须实时的推送给2100万的账户。奥巴马这个顶点就可以看做一个超级节点。举一个反例，当[Stephen Mallette](https://twitter.com/spmallette)发tweets时，只有59个流需要更新。Twitter意识到了这个差异并对“奥巴马们”（名人）与“Stephens们”（普通人）在Twitter星球上使用了不同的机制。

##  Blueprints 与顶点查询

[Blueprints](http://blueprints.tinkerpop.com/)是一个基于图的Java接口软件。许多图形数据库，内存图引擎与批量分析框架都在使用Blueprints。在2012年6月，[Blueprints 2.x](https://github.com/tinkerpop/blueprints/wiki/The-Major-Differences-Between-Blueprints-1.x-and-2.x)加入了对“[vertex queries](https://github.com/tinkerpop/blueprints/wiki/Vertex-Query)”的支持，我们最好用一个例子来说明顶点查询。

假设有一个名叫Dan的顶点，他有1110条边。这些边中有10条是Dan认识的人，100条是Dan关注的人，1000条是Dan发的tweet。如果Dan想要查看他所认识的人的列表，而边却没有通过边的类型（原文为标签Label）做索引，我们就需要迭代遍历所有的1110条边来获得这10个Dan所认识的人。如果Dan的边通过边的类型做了索引，那么我们通过查询hash表就会马上得到我们所要找的10个人，--`O(n)` vs. `O(1)`，n为Dan所有的边的数量。

这个通过类型来分类边的思想可以在属性图形中更进一步。属性图支持在顶点与边上设立键值对。比如，一个”认识“边可以有一个值为“工作”，“家庭”或“喜欢”的”类型“属性，有一个“开始”属性来表示这段关系的开始时间。相似的，“喜欢”边可以有一个值为1-5的“打分”属性，“tweet”边可以有一个“时间戳”来表示何时发表。Bluprints的查询允许开发者通过这些属性约束来检索边。比如，获取所有Dan的高分评价，Blueprints的代码如下所示：

```java
dan.query().labels("likes").interval("rating",4,6).vertices()
```

### Titan与以顶点为中心索引

Blueprints只提供了代表顶点查询的接口，它依赖于图系统去使用它特定的约束来展示它的优点。而分布式的图形数据库[Titan](http://thinkaurelius.github.com/titan/)广泛的使用了以顶点为中心的索引来支持来自硬盘与内存的细粒度的边的检索。要证明这些索引的效率，我们做了一个基于[BerkeleyDB](https://github.com/thinkaurelius/titan/wiki/Using-BerkeleyDB)（一个遵循[ACID](http://en.wikipedia.org/wiki/ACID)的Titan载体--详情见 [Titan's storage overview](https://github.com/thinkaurelius/titan/wiki/Storage-Backend-Overview)）的Titan测试。

10个实例均创建了一个名为Dan的人顶点，其中5个建立了索引，5个没有建。每5个实例中的关系数量如下表：

| 边的总数  | ”认识“边 | “喜欢”边  | ”tweet“边 |
| --------- | ----- | ------ | -------- |
| 111   | 1 | 10 | 100  |
| 1,110 | 10| 100| 1000 |
| 11,100| 100   | 1000   | 10000|
| 111,000   | 1000  | 10000  | 100000   |
| 1,110,000 | 10000 | 100000 | 1000000  |

使用基于Groovy的Gremlin来生成之前提到的星图代码如下，其中`i`为图的大小：

```groovy
g = TitanFactory.open('/tmp/supernode')
// index configuration snippet goes here for Titan w/ vertex-centric indices
g.createKeyIndex('name',Vertex.class)
g.addVertex([name:'dan'])
   
r = new Random(100)
types = ['work','family','favorite']
(1..i).each{g.addEdge(g.V('name','dan').next(),g.addVertex(),'knows',[type:types.get(r.nextInt(3)),since:it]); stopTx(g,it)}
(1..(i*10)).each{g.addEdge(g.V('name','dan').next(),g.addVertex(),'likes',[rating:r.nextInt(5)]); stopTx(g,it)}
(1..(i*100)).each{g.addEdge(g.V('name','dan').next(),g.addVertex(),'tweets',[time:it]); stopTx(g,it)}
```

对于有索引的5个实例，我们需要先建立索引（[Titan's type configurations](https://github.com/thinkaurelius/titan/wiki/Type-Definition-Overview)），代码如下：

```groovy
type = g.makeType().name('type').simple().functional(false).dataType(String.class).makePropertyKey()
since = g.makeType().name('since').simple().functional(false).dataType(Integer.class).makePropertyKey()
rating = g.makeType().name('rating').simple().functional(false).dataType(Integer.class).makePropertyKey()
time = g.makeType().name('time').simple().functional(false).dataType(Integer.class).makePropertyKey()
g.makeType().name('knows').primaryKey(type,since).makeEdgeLabel()
g.makeType().name('likes').primaryKey(rating).makeEdgeLabel()
g.makeType().name('tweets').primaryKey(time).makeEdgeLabel()
```

接下来，三个遍历Dan的方法出现了。第一个遍历获取了Dan认识的随机关系的人。第二个遍历获取了Dan给了较高评分（4-5分）的事物。第三个遍历获取了Dan最新的10条tweet。最终，Germlin会将他们编译成适当的顶点查询（[Gremlin's traversal optimizations](https://github.com/tinkerpop/gremlin/wiki/Traversal-Optimization)）。

```groovy
g.V('name','dan').outE('knows').has('type',types.get(r.nextInt(3)).inV
g.V('name','dan').outE('likes').interval('rating',4,6).inV
g.V('name','dan').outE('tweets').has('time',T.gt,(i*100)-10).inV
```

每一个遍历我们都执行了25遍，且每次执行都会重启数据库以避免JVM的缓存机制。不过内存，预热缓存对响应时间也有类似的影响（尽管相对较快）。平均的结果如下图所示，y轴为指数增长。其中绿色，红色，蓝色分别为之前的三种查询。亮色版本为无索引模式，暗色为有索引模式。

![vertex-times-barchart](https://www.datastax.com/wp-content/uploads/2017/01/vertex-times-barchart.png)

最引人注意的或许是找出Dan的10条最近的Tweet。当有了索引后，即使Dan的tweet多至100万，查询所花费的时间也不过是近1.5微秒。而没有索引，这个查询的响应时间则成比例的无限制增长至13秒。这是在相投的结果集下响应时间的4个数量级的差异。这个例子表明以顶点为中心的索引对实时流类型系统的作用。

最后，Titan也支持合成的键索引，先前的图的结构代码片段就给“知道”边分配了“类型“与”开始“的主键。因此，使用索引获取Dan的10个最近的同事比在内存中获取所有同事后用”开始“字段排序要快的多。感兴趣的读者可以通过修改所提供的代码片段来探索各种以顶点为中心的查询组合的运行时间。

### 总结

当我们对不同的边区别对待时，超级节点问题就不在是一个难点。如果所有的点都一视同仁，那么就需要使用线性（`O(n)`）的搜索。然而当使用索引与排序后，就能实现`O(log(n))` 与 `O(1)`的复杂度。我们所得到的结果证明，在我们使用以顶点为中心的索引时，我们查找“喜欢”与“知道”有2-5倍的速度提升，当我们查找“tweet”时甚至有10000倍的速度提升。我们现在来考虑不止一次遍历的情况。

当我们遇到一个复合查询。1毫秒与10秒的运行时间的差距可谓天差地别。

Titan图数据库可以扩展至支持数百亿条边（基于[Cassandra](https://github.com/thinkaurelius/titan/wiki/Using-Cassandra) and [HBase](https://github.com/thinkaurelius/titan/wiki/Using-HBase)）。在这个庞大的图中顶点通常会有上百万条边。在大型图形数据世界中，高效的从硬盘与内存中存储于读取数据是十分重要的。使用Titan，边的过滤被放置在硬盘级，所以只有必要的数据才会读入内存。以顶点为中心的查询与索引智能的利用标签与属性信息克服了超级节点的问题，快速找到所需的边与顶点。



> 相关资料
>
> Rodriguez, M.A., Broecheler, M., "[Titan: The Rise of Big Graph Data](http://www.slideshare.net/slidarko/titan-the-rise-of-big-graph-data)," Public Lecture at Jive Software, Palo Alto, 2012.
>
> Broecheler, M., LaRocque, D., Rodriguez, M.A., "[Titan Provides Real-Time Big Graph Data](http://thinkaurelius.com/2012/08/06/titan-provides-real-time-big-graph-data/)," Aurelius Blog, August 2012.

> 原文翻译自[A Solution to the Supernode Problem](https://www.datastax.com/dev/blog/a-solution-to-the-supernode-problem)
