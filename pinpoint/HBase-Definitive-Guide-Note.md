# HBase权威指南-学习笔记

## 内容提要

### 概要

1. Hadoop将HBase的可伸缩性变得简单

2. 把大型数据集分布到商业服务器集群中

3. 网关服务器访问HBase

### 细节说明

1. 了解HBase框架细节，包括存储格式，预写日志，后台进程

2. HBase集成了MapReduce框架，了解调节集群，设计模式，拷贝表，导入批量数据和删除节点的操作数据存储，先进入到提交日志，然后，才会将这些数据写入内存中的memstore，最后再写入磁盘的HFile。

```
<property>
    <name>hbase.rootdir</name>
    <value>file:///~/hbase</value>
</property>

```

![](/images/pinpoint/Put-Data-Into-HBase-Demo-1.png)

Hbase启动时有时候找不到JAVA\_HOME是因为他没有加载系统环境里面的JAVA\_HOME，需要手动到`conf/hbase_env.sh`的首行加上`JAVA_HOME`。

验证hbase启动成功

```
jps
```
```
sudo tail -f logs/catalina.2016-11-28.log
```

Hbase1.2与jdk8可能不太兼容

## Hbase体系结构

HBase的服务器体系结构遵从简单的主从服务器架构，它由HRegion Server群和HBase Master服务器构成。HBase Master负责管理所有的HRegion Server，而HBase中的所有RegionServer都是通过ZooKeeper来协调，并处理HBase服务器运行期间可能遇到的错误。HBase Master Server本身并不存储HBase中的任何数据，HBase逻辑上的表可能会被划分成多个Region，然后存储到HRegion Server群中。HBase Master Server中存储的是从数据到HRegion Server的映射。因此HBase体系结构如下图所示。

### Client

HBase Client使用HBase的RPC机制与HMaster和HRegion Server进行通信，对于管理类操作，Client与HMaster进行RPC；对于数据读写类操作，Client与HRegionServer进行RPC。

### Zookeeper

Zookeeper Quorum中除了存储了-ROOT-表的地址和HMaster的地址，HRegionServer会把自己以Ephemeral方式注册到Zookeeper中，使得HMaster可以随时感知到各个HRegionServer的健康状态。此外，Zookeeper也避免了HMaster的单点问题。

### HMaster

每台HRegionServer都会与HMaster进行通信，HMaster的主要任务就是要告诉每台HRegion Server它要维护哪些HRegion。

当一台新的HRegionServer登录到HMaster时，HMaster会告诉它等待分配数据。而当一台HRegion死机时，HMaster会把它负责的HRegion标记为未分配，然后再把它们分配到其他的HRegion Server中。

HBase已经解决了HMaster单点故障问题（SPFO），并且HBase中可以启动多个HMaster，那么它就能够通过Zookeeper来保证系统中总有一个Master在运行。HMaster在功能上主要负责Table和Region的管理工作，具体包括：

* 管理用户对Table的增删改查操作
* 管理HRegionServer的负载均衡，调整Region分布
* 在Region Split后，负责新Region的分配
* 在HRegionServer停机后，负责失效HRegionServer上的Region迁移

### HRegion

当表的大小超过设置值得时候，HBase会自动地将表划分为不同的区域，每个区域包含所有行的一个子集。对用户来说，每个表是一堆数据的集合，靠主键来区分。从物理上来说，一张表被拆分成了多块，每一块就是一个HRegion。我们用表名+开始/结束主键来区分每一个HRegion，一个HRegion会保存一个表里面某段连续的数据，从开始主键到结束主键，一张完整的表格是保存在多个HRegion上面。

当Table随着记录数不断增加而变大后，会逐渐分裂成多份splits，成为regions，一个region由[startkey, endkey]表示，不同的region会被Master分配给响应的RegionServer进行管理。

### HRegionServer

所有的数据库数据一般都是保存在Hadoop分布式文件系统上面的，用户通过一系列HRegion服务器获取这些数据，一台机器上面一般只运行一个HRegionServer，且每一个区段的HRegion也只会被一个HRegion服务器维护。如下图所示HRegion Server数据存储关系图。

HRegion Server主要负责响应用户的IO请求，向HDFS文件系统中读写数据，是HBase中最核心的模块。HRegionServer内部管理了一系列HRegion对象，每个HRegion对应了Table中的一个Region，HRegion中由多个HStore组成。每个HStore对应了Table中的一个Column Family的存储，可以看出每个ColumnFamily其实就是一个集中的存储单元，因此最好将具备共同IO特性的column放在一个Column Family中，这样最高效。

HStore存储时HBas存储的核心了，其中由两部分组成，一部分是MemStore，一部分是StoreFiles。MemStore_是Sorted Memory Buffer，用户写入数据首先会放入MemStore，当MemStore满了以后会flush成一个_StoreFile（底层是HFile），当StoreFile文件数增长到一定阈值，会触发Compact合并操作，将多个StoreFile合并成一个StoreFile，合并过程中会进行版本合并和数据删除，因此可以看出HBase其实只有增加数据，所有的更新和删除操作都是后续的compact过程中进行的，这使得用户的写操作只要进入内存中就可以立刻返回，保证了HBase IO的高性能。当StoreFiles Compact后，会逐步形成越来越大的StoreFile，当单个StoreFile大小超过一定的阈值后，会触发Split操作，同时，会把当前的Region Split成2个Region，父Region会下线，新Split出的2个孩子Region会被HMaster分配到响应的HRegion Server上，使得原先1个Region的压力得以分流道2个Region上。下图描述了Compaction和Split的过程。

理解上述HStore的基本原理之后，必须了解一下HLog的功能，因为上述的HStore在系统正常工作的前提下是没有问题的，但是在分布式系统环境中，无法避免系统出错或者宕机，一次一旦HRegion Server意外退出，MemStore中的内存数据将会丢失，这就需要引入HLog了。每一个HRegionServer中都有一个HLog对象，HLog是一个实现Write Ahead Log的类，在每次用户操作写入MemStore的同时，也会写一份数据到HLog文件中，HLog文件定期会滚动出新的，并删除旧的文件，当HRegionServer意外终止后，HMaster会通过Zookeeper感知到，HMaster首先会处理遗留的HLog文件，将其中不同Region的Log数据进行拆分，分别放到相应Region的目录下，然后再将失效的Region重新分配，领取到这些Region的HRegionServer在LoadRegion过程中，会发现有历史HLog需要处理，因此会Replay HLog中的数据到MemStore中，然后flush到StoreFiles，完成数据恢复。

### HBase存储格式

HBase中的所有数据文件都存储在Hadoop HDFS文件系统上，包括上述提到的两种文件类型:

* HFile HBase中的KeyValue数据的存储格式，HFile是Hadoop的二进制格式文件，实际上StoreFile就是对HFile做了轻量级的包装，即StoreFile底层就是HFile。
* HLogFile，HBase中WAL（Write Ahead Log）的存储格式，物理上是Hadoop的Sequence File。

### ROOT表和META表

用户表的Regions元数据被存储在.META.表中，随着Region的增多，.META.表中的数据也会增大，并分裂成多个Regions。为了定位.META.表中各个Regions的位置，把.META.表中的所有Regions的元数据保存在-ROOT-表中，最后由Zookeeper记录-ROOT-表的位置信息。所有客户端访问用户数据前，需要首先访问Zookeeper获得-ROOT-的位置，然后方位-ROOT-表获得.META.表的位置，最后根据.META.表中的信息确定用户数据存放的位置，-ROOT-表永远不会被分割，它只有一个Region，这样可以保证最多需要三次跳转就可以定位任意一个Region。为了加快访问速度，.META.表的Regions全部保存在内存中，如果.META.表中的每一行在内存中占大约1KB，且每个Region限制为128M，下图中的三层结构可以保存Regions的数目为(128M/1KB)*(128/1KB)=2^34个。

## Hbase设计

### 行健设计
1. Rowkey是以字典顺序从大到小排序
得到热点以及时间排序，可以把最大值减去当前数得出的值做行健，这样就能得到从大到小的排序
2. Rowkey尽量散列Rowkey设计，可以使得数据均匀分布在不同机子上，而不是集中在一台机子上，查询时单机负担还是很大，没有起到负载均衡的作用
3. Rowkey长度尽量短

注意：之所以时间不需要分布均匀是因为时间是要排序的，分布的话，hbase不好排序

### 列族设计
列族不要太多，最好不要三个以上

- 由于hbase的flush操作是针对整个表的，如果列族太多，只要其中一个列族在操作大量数据的时候就会flush一次，那些不相关的列族也会flush
- 不同列族会有不同的hfile文件，经常flush将会导致hfile文件增多，引发不必要的compaction（合并）操作

总之，多个列族会造成很多不必要I/O负载，另外还可以对列族进行配置，设计列族

## Hbase使用

### 源生java

#### 表管理入口类：HBaseAdmin
用于创建、删除、修改表，并且对region进行操作

#### 连接池类：HTablePool
用于多线程创建表


## 问题
1. 正常设置的与zookeeper的超时时间，用在分布式的环境当中，很容易超时，这个时候要重新设置zookeeper超时时间，在conf/hbase-site.xml当中
```
<property>
    <name>zookeeper.session.timeout</name>
    <value>120000</value>
</property>
```









