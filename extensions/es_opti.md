# Elasticsearch 优化实践

Elasticsearch（简称ES）是一个基于Apache Lucene(TM)的开源搜索引擎。无论在开源还是专有领域，Lucene可以被认为是迄今为止最先进、性能最好的、功能最全的搜索引擎库。

Elasticsearch 作为开源首选的分布式搜索分析引擎，通过一套系统轻松满足用户的日志实时分析、全文检索、结构化数据分析等多种需求，大幅降低大数据时代挖掘数据价值的成本。

但是，Lucene只是一个库。想要使用它，你必须使用Java来作为开发语言并将其直接集成到你的应用中，更糟糕的是，Lucene非常复杂，你需要深入了解检索的相关知识来理解它是如何工作的。

Elasticsearch 也使用Java开发并使用Lucene作为其核心来实现所有索引和搜索的功能，但是它的目的是通过简单的RESTful API来隐藏Lucene的复杂性，从而让全文搜索变得简单。

不过，Elasticsearch 不仅仅是Lucene和全文搜索，还能这样去描述它：

- 分布式的实时文件存储，每个字段都被索引并可被搜索
- 分布式的实时分析搜索引擎
- 可以扩展到上百台服务器，处理PB级结构化或非结构化数据

而且，所有的这些功能被集成到一个服务里面，你的应用可以通过简单的RESTful API、各种语言的客户端甚至命令行与之交互。

上手ES非常容易。它提供了许多合理的缺省值，并对初学者隐藏了复杂的搜索引擎理论。它开箱即用（安装即可使用），只需很少的学习既可在生产环境中使用。

Elasticsearch 在 Apache 2 license 下许可使用，可以免费下载、使用和修改。

以下是 ElasticSearch 中几个常用的概念：

- Cluster（集群）：ES可以运行一个独立的实例提供搜索服务。不过，为了处理大型数据集，实现容错和高可用性，ES可以运行在一群服务器上。这些服务器的集合称为集群。
- Node（节点）：组成集群的每一台机器称之为一个节点。
- Index（索引）：在Elasticsearch中存储数据的行为就叫做索引(indexing)。一个索引(index)就像是传统关系数据库中的数据库，它是相关文档存储的地方，index的复数是indices 或indexes。
- Shard（分片）：当有大量的文档时，由于内存的限制、磁盘处理能力不足、无法足够快的响应客户端的请求等，一个节点可能不够。这种情况下，数据可以分为较小的分片。每个分片放到不同的服务器上。
当你查询的索引分布在多个分片上时，ES会把查询发送给每个相关的分片，并将结果组合在一起，而应用程序并不知道分片的存在。即这个过程对用户来说是透明的。
- Replica（副本）：为提高查询吞吐量或实现高可用性，可以使用分片副本。
副本是一个分片的精确复制，每个分片可以有零个或多个副本。ES中可以有许多相同的分片，其中之一被选择更改索引操作，这种特殊的分片称为主分片。
当主分片丢失时，如：该分片所在的数据不可用时，集群将副本提升为新的主分片。

### 一 集群配置

###### 1 操作系统配置优化

禁用 swap，一旦发生内存交换，肯定会带来巨大的性能问题。在 ElasticSearch 的高效读写需求下，不允许使用交换内存。

```bash
sed -i '/swap/s/^/#/' /etc/fstab
swapoff -a
```

提高最大打开文件数限制和最大用户进程数

```bash
echo "* - nofile 65536" >> /etc/security/limits.conf
echo "* - nproc 131072" >> /etc/security/limits.conf
```

Elasticsearch 对各种文件混合使用了 NioFs 和 MMapFs 。确保配置最大映射数量，以便有足够的虚拟内存可用于 mmapped 文件。

```bash
echo "vm.max_map_count = 262144" >> /etc/sysctl.conf
sysctl -p
```

###### 2 节点个数

ES内部维护集群通信，不是基于zookeeper的分发部署机制，所以，无需奇数。 但是discovery.zen.minimum_master_nodes的值要设置为：候选主节点的个数/2+1，才能有效避免脑裂。

###### 3 磁盘性能
Elasticsearch最大的瓶颈往往是磁盘读写性能，尤其是随机读取性能。使用SSD（PCI-E接口SSD卡/SATA接口SSD盘）通常比机械硬盘（SATA盘/SAS盘）查询速度快5~10倍.

### 二 ES 开启高性能写模式

- 现象

Skywalking 日志出现 429 Too many requests

- 造成原因

因为接入Skywalking 的实例太多，导致Skywalking 在Bulk（批量）写入 ES 的时候，ES 性能不够报错。

综合来说,提升写入速度从以下几方面入手:

- 加大 translog flush ,目的是降低 iops,writeblock
- 加大 index refresh间隔, 目的除了降低 io, 更重要的降低了 segment merge 频率
- 调整 bulk 线程池和队列
- 优化磁盘间的任务均匀情况,将 shard 尽量均匀分布到物理主机的各磁盘
- 优化节点间的任务分布,将任务尽量均匀的发到各节点
- 优化 lucene 层建立索引的过程,目的是降低 CPU 占用率及 IO
- 针对Skywalking，提炼出如下优化参数，在elasticsearch.yml文件中新增后重启 ElasticSearch 生效：

```bash
# 锁定物理内存地址，防止elasticsearch内存被交换出去,也就是避免es使用swap交换分区，https://www.elastic.co/guide/en/elasticsearch/reference/7.4/setup-configuration-memory.html
bootstrap.memory_lock: true
 
# 增大队列大小，建立索引的过程偏计算密集型任务,应该使用固定大小的线程池配置,来不及处理的放入队列,线程数量配置为 CPU 核心数+1,避免过多的上下文切换.队列大小可以适当增加.
thread_pool.index.queue_size: 1000
thread_pool.write.queue_size: 1000
```

Elasticsearch近实时的本质是：最快1s写入的数据可以被查询到。 如果refresh_interval设置为1s，势必会产生大量的segment，检索性能会受到影响。 所以，非实时的场景可以调大，设置为30s，甚至-1。

### 三 设置合理的磁盘限额水位线

为了保护节点数据安全ES 会定时(cluster.info.update.interval，默认 30 秒)检查一下各节点的数据目录磁盘使用情况。根据单个分片的数据大小，合理调整 watermark.low  的值。如果单个分片数据大小超过磁盘空间的 10%，建议将 watermark.low 调整到 70% 。同样的，对于大量的小分片可以调高 watermark.low 来提高磁盘的使用率，降低成本。

```bash
cluster.routing.allocation.disk.threshold_enabled      默认true, 启用磁盘空间阈值检查
cluster.routing.allocation.disk.watermark.low          默认85%, 分片不会分配(除新建的索引的主分片不受影响)
cluster.routing.allocation.disk.watermark.high         默认90%, 达到该值,ES会做relocate,影响所有分片
cluster.routing.allocation.disk.watermark.flood_stage  默认95%, index.blocks.read_only_allow_delete,当磁
```

当磁盘空间水位线超过 95%会导致索引只读。需要通过下面的参数取消只读的状态。

```bash
PUT _all/_settings
{
  "index": {
    "blocks": {
      "read_only_allow_delete": "false"
    }
  }
}
```

### 四 删除索引的副本分片

在测试环境中，往往对数据并没有很高的可用性要求，但是对 ElasticSearch 的性能有一定的要求。副本分片在一定程度上会占用 ElasticSearch 的堆内存资源，并且在集群重启时，索引的恢复耗时也会受副本分片的影响。因此，在测试环境，可以将索引的副本分片设置为 0。

```bash
ES_HOST=10.15.120.16; curl -sSL "http://$ES_HOST:9200/_cat/shards?v"|awk '{if($3=="r") print $1}'|sort|uniq|xargs -I {} echo "curl -sSL -XPUT http://$ES_HOST:9200/{}/_settings -d '{\"index\":{\"number_of_replicas\":0}}' -H 'Content-Type: application/json'" > delete_replicas.sh
# 检查 delete_replicas.sh 文件中的索引名称，防止误删数据，确认无误后执行脚本
bash ./delete_replicas.sh
```

// TODO: 待补充
