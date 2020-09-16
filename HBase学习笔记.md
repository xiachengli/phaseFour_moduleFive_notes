#### 初始HBase

##### HBase简介

HBase是一个海量列式非关系型数据库系统，可以提供超大规模数据集的实时随机读写

特点：

- 海量存储： 底层基于HDFS存储海量数据 
- 列式存储：HBase表的数据是基于列族进行存储的，一个列族包含若干列 
- 极易扩展：底层依赖HDFS，当磁盘空间不足的时候，只需要动态增加DataNode服务节点就可以 
- 高并发：支持高并发的读写请求
- 稀疏：稀疏主要是针对HBase列的灵活性，在列族中，你可以指定任意多的列，在列数据为空的情 况下，是不会占用存储空间的
- 数据的多版本：HBase表中的数据可以有多个版本值，默认情况下是根据版本号去区分，版本号就 是插入数据的时间戳 
- 数据类型单一：所有的数据在HBase中是以字节数组进行存储

应用：

- 交通方面：船舶GPS信息，每天有上千万左右的数据存储
- 金融方面：消费信息、贷款信息、信用卡还款信息等 
- 电商方面：电商网站的交易信息、物流信息、游览信息等 
- 电信方面：通话信息 

总结：：HBase适合海量明细数据的存储，并且后期需要有很好的查询性能

##### HBase数据模型

###### 逻辑架构

![image-20200916125130241](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200916125130241.png)

###### 物理架构

![image-20200916125217370](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200916125217370.png)

##### HBase整体架构

![image-20200916125333098](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200916125333098.png)

zookeeper

- 实现master的高可用
- 保存HBase的元数据信息
- 对master和RegionServer实现了监控

Master

- 为RegionServer分配Region
- 维护整个集群的负载均衡
- 维护集群的元数据信息
- 发现失效的region，并将失效的region分配到正常的regionServer上

RegionServer

- 负责管理region
- 接受客户端的读写数据请求
- 切分在运行中变大的region

Region

- 每个region由多个store组成
- 每个store保存一个列族
- 每个store由一个MemStore和多个StoreFile组成。MemStore是store在内存中的内容，写到文件后就是storeFile

##### HBase shell基本操作

- 进入Hbase客户端命令操作界面

  hbase shell

- 查看帮助命令

  help

- 查看当前数据库中有哪些表

  list

- 创建一张lagou表， 包含base_info、extra_info两个列族

  create 'lagou','base_info','extra_info'

- 添加数据操作 

  put 'lagou' ,'rk1','base_info:name','wang'

- 查询数据 

  get 'lagou' ,'rk1'

- 清空表数据

  truncate 'lagou'

- 删除表(先disable,在drop)

  disable 'lagou'

  drop 'lagou'

#### HBase原理深入

##### HBase读数据流程

![image-20200916130619004](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200916130619004.png)

- 首先从zk找到meta表的region位置，然后读取meta表中的数据，meta表中存储了用户表的region信息 
- 根据要查询的namespace、表名和rowkey信息。找到写入数据对应的region信息 
- 找到这个region对应的regionServer，然后发送请求 
- 查找对应的region 
- 先从memstore查找数据，如果没有，再从BlockCache上读取
- 如果BlockCache中也没有找到，再到StoreFile上进行读取

##### HBase写数据流程

![image-20200916130823891](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200916130823891.png)

- 首先从zk找到meta表的region位置，然后读取meta表中的数据，meta表中存储了用户表的region信息 
- 根据namespace、表名和rowkey信息。找到写入数据对应的region信息 
- 找到这个region对应的regionServer，然后发送请求 
- 把数据分别写到HLog（write ahead log）和memstore各一份 
- memstore达到阈值后把数据刷到磁盘，生成storeFile文件 
- 删除HLog中的历史数据 

##### HBase的刷写及合并机制

flush刷写机制：

- 当memstore的大小超过这个值的时候，会flush到磁盘,默认为128M

  ```xml
  <property>    <name>hbase.hregion.memstore.flush.size</name>    <value>134217728</value> </property
  ```

- 当memstore中的数据时间超过1小时，会flush到磁盘

  ```
  <property>    <name>hbase.regionserver.optionalcacheflushinterval</name>    <value>3600000</value> </property>
  
  ```

- regionServer的全局memstore的大小，超过该大小会触发flush到磁盘的操作,默认是堆大小的40%

  ```
  <property>    <name>hbase.regionserver.global.memstore.size</name>    <value>0.4</value> </property>
  
  ```

- 手动flush

  ```
  flush tableName
  ```

阻塞机制：

以上介绍的是Store中memstore数据刷写磁盘的标准，但是Hbase中是周期性的检查是否满足以上标准满足则进行刷写，但是如果在下次检查到来之前，数据疯狂写入Memstore中，会出现什么问 题呢？ 会触发阻塞机制，此时无法写入数据到Memstore，数据无法写入Hbase集群

compact合并机制：在hbase中主要存在两种类型的compac合并 

- minor compact 小合并

  - 在将Store中多个HFile(StoreFile)合并为一个HFile

    这个过程中，删除和更新的数据仅仅只是做了标记，并没有物理移除，这种合并的触发频率很高

- major compact 大合并 

  - 合并Store中所有的HFile为一个HFile 

     这个过程有删除标记的数据会被真正移除，同时超过单元格maxVersion的版本记录也会被删除。合并频率比较低，默认7天执行一次，并且性能消耗非常大，建议生产关闭(设置为 0)，在应用空闲时间手动触发。一般可以是手动控制进行合并，防止出现在业务高峰期

##### Region拆分机制

Region中存储的是大量的rowkey数据 ,当Region中的数据条数过多的时候,直接影响查询效率.当Region过大的时候.HBase会拆分Region , 这也是Hbase的一个优点 

- 拆分策略

  - ConstantSizeRegionSplitPolicy
  - IncreasingToUpperBoundRegionSplitPolicy 
  - SteppingSplitPolicy
  - KeyPrefixRegionSplitPolicy
  - DelimitedKeyPrefixRegionSplitPolicy
  - DisabledRegionSplitPolicy

- RegionSplitPolicy的应用

  Region拆分策略可以全局统一配置，也可以为单独的表指定拆分策略

  - 全局配置（hbase-site.xml）

    ```
    <property>
      <name>hbase.regionserver.region.split.policy</name>
      <value>org.apache.hadoop.hbase.regionserver.IncreasingToUpperBoundRegionSplitPolicy</value>
    </property>
    
    ```

  - 单表指定

    - 通过Java API为单独的表指定Region拆分策略

      tableDesc.setValue(HTableDescriptor.SPLIT_POLICY, IncreasingToUpperBoundRegionSplitPolicy.class.getName());

    - 通过HBase Shell为单个表指定Region拆分策略

      create 'test2', {METADATA => {'SPLIT_POLICY' => 'org.apache.hadoop.hbase.regionserver.IncreasingToUpperBoundRegionSplitPolicy'}},{NAME => 'cf1'}

##### HBase表的预分区

当一个table刚被创建的时候，Hbase默认的分配一个region给table。也就是说这个时候，所有的读写请求都会访问到同一个regionServer的同一个region中，这个时候就达不到负载均衡的效 果了，集群中的其他regionServer就可能会处于比较空闲的状态。解决这个问题可以用pre-splitting,在创建table的时候就配置好，生成多个region

##### Region合并

- 通过Merge类冷合并Region

  通过org.apache.hadoop.hbase.util.Merge类来实现，不需要进入hbase shell，直接执行（需要先关闭hbase集群）：

  ```
  hbase org.apache.hadoop.hbase.util.Merge student \ student,,1595256696737.fc3eff4765709e66a8524d3c3ab42d59. \ student,aaa,1595256696737.1d53d6c1ce0c1bed269b16b6514131d0.
  
  ```

- 通过online_merge热合并Region 

  与冷合并不同的是，online_merge的传参是Region的hash值，而Region的hash值就是Region名称的最后那段在两个.之间的字符串部分

  需要进入hbase shell

  ```
  merge_region 'c8bc666507d9e45523aebaffa88ffdd6','02a9dfdf6ff42ae9f0524a3d8f4c7777'
  ```

#### HBase API应用和优化

##### HBase协处理器

访问HBase的方式是使用scan或get获取数据，在获取到的数据上进行业务运算。但是在数据量非常大的时候，比如一个有上亿行及十万个列的数据集，再按常用的方式移动获取数据就会遇到性 能问题。客户端也需要有强大的计算能力以及足够的内存来处理这么多的数据。 此时就可以考虑使用Coprocessor(协处理器)。将业务运算代码封装到Coprocessor中并在RegionServer上运行，即在数据实际存储位置执行，最后将运算结果返回到客户端。利用协处理器，用户 可以编写运行在 HBase Server 端的代码

Hbase Coprocessor类似以下概念：
触发器和存储过程：一个Observer Coprocessor有些类似于关系型数据库中的触发器，通过它我们可以在一些事件（如Get或是Scan）发生前后执行特定的代码。Endpoint Coprocessor则类似于 关系型数据库中的存储过程，因为它允许我们在RegionServer上直接对它存储的数据进行运算，而非是在客户端完成运算。 MapReduce：MapReduce的原则就是将运算移动到数据所处的节点。Coprocessor也是按照相同的原则去工作的

协处理器类型：

- Observer

  与触发器(trigger)类似：在一些特定事件发生时回调函数（也被称作钩子函数，hook）被执行。这些事件包括一些用户产生的事件，也包括服务器端内部自动产生的事件。 协处理器框架提供的接口如下 RegionObserver：用户可以用这种的处理器处理数据修改事件，它们与表的region联系紧密。
  MasterObserver：可以被用作管理或DDL类型的操作，这些是集群级事件。 WALObserver：提供控制WAL的钩子函数 

- Endpoint 

  类似传统数据库中的存储过程，客户端可以调用这些 Endpoint 协处理器在Regionserver中执行一段代码，并将 RegionServer 端执行结果返回给客户端进一步处理

##### HBase表的RowKey设计

-  长度原则

  rowkey是一个二进制码流，可以是任意字符串，最大长度64kb，实际应用中一般为10-100bytes，以byte[]形式保存，一般设计成定长

- 散列原则

    建议将rowkey的高位作为散列字段，这样将提高数据均衡分布在每个RegionServer，以实现负载均衡

- 唯一原则

  必须在设计上保证其唯一性

##### HBase表的热点

检索habse的记录首先要通过row key来定位数据行。当大量的client访问hbase集群的一个或少数几个节点，造成少数region server的读/写请求过多、负载过大，而其他region server负载却很 小，就造成了“热点”现象

###### 热点的解决方案

- 预分区

   预分区的目的让表的数据可以均衡的分散在集群中，而不是默认只有一个region分布在集群的一个节点上

- 加盐

  在rowkey的前面增加随机数，具体就是给rowkey分配一个随机前缀以使得它和之前的rowkey的开头不同

- 哈希

   哈希会使同一行永远用一个前缀加盐。哈希也可以使负载分散到整个集群，但是读却是可以预测的。使用确定的哈希可以让客户端重构完整的rowkey，可以使用get操作准确获取某 一个行数据

- 反转

  反转固定长度或者数字格式的rowkey。这样可以使得rowkey中经常改变的部分（最没有意义的部分）放在前面。这样可以有效的随机rowkey，但是牺牲了rowkey的有序性

##### HBase的二级索引

  HBase表按照rowkey查询性能是最高的。rowkey就相当于hbase表的一级索引

  为了HBase的数据查询更高效、适应更多的场景，诸如使用非rowkey字段检索也能做到秒级响应，或者支持各个字段进行模糊查询和多字段组合查询等， 因此需要在HBase上面构建二级索 引， 以满足现实中更复杂多样的业务需求

hbase的二级索引其本质就是建立hbase表中列与行键之间的映射关系

常见的二级索引我们一般可以借助各种其他的方式来实现，例如Phoenix或者solr或者ES等 

##### 布隆过滤器在HBase中的应用

 Bloom Filter是一种空间效率很高的随机数据结构，它利用位数组很简洁地表示一个集合，并能判断一个元素是否属于这个集合

hbase 中布隆过滤器来过滤指定的rowkey是否在目标文件，避免扫描多个文件。使用布隆过滤器来判断

布隆过滤器返回true,在结果不一定正确，如果返回false则说明确实不存在

![image-20200916142614390](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200916142614390.png)



