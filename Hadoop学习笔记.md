#### 大数据简介

##### 大数据的定义

  大数据是指在一定时间范围内无法用常规软件进行处理的数据集合

##### 大数据的特点（5V）

- volume大量

  数据量非常大

- velocity高速

  数据的创建，存储，分析都要求被高速处理

- variety多样

  数据的形式和来源多样化

- veracity真实

  数据具有真实性

- value低价值

  数据的价值密度相对较低

##### 大数据的应用场景

仓储物流、电商零售、汽车、电信、生物医学、人工智能、智慧城市

#### Hadoop简介

##### 什么是Hadoop

Hadoop 是一个适合大数据分布式存储和计算的框架。狭义上说Hadoop就是一个框架平台，广义上Hadoop代表一个生态圈

| Hadoop生态圈                |
| --------------------------- |
| Hadoop(HDFS+MapReduce+Yarn) |
| Hive数据仓库工具            |
| HBase海量列式非关系型数据库 |
| Flume数据采集工具           |
| Sqoop ETL工具               |
| Kafka消息中间件             |
| ......                      |

##### Hadoop的起源

- Hadoop最早起源于Nutch，Nutch的创始人是Doug Cutting
- GFS(Google File System)：处理海量网页的存储      --> HDFS
- MapReduce(Google的分布式计算框架)：处理海量网页的索引计算问题   -->  Hadoop MapReduce
- BigTable：大型分布式数据库  --> HBase

##### Hadoop的特点

![image-20200914185935052](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200914185935052.png)

##### Hadoop的发行版本

- Apache Hadoop 原始版本

  官网地址：http://hadoop.apache.org/ 

  优点：拥有全世界的开源贡献，代码更新版本比较快 ，学习非常方便

  缺点：版本的升级，版本的维护，以及版本之间的兼容性

  Apache所有软件的下载地址（包括各种历史版本）：http://archive.apache.org/dist/ 

- 软件收费版本ClouderaManager CDH版本 --生产环境使用 

  官网地址：https://www.cloudera.com/ 

  Cloudera是美国一家大数据公司在Apache开源Hadoop的版本上，通过自己公司内部的各种 补丁，实现版本之间的稳定运行，大数据生态圈的各个版本的软件都提供了对应的版本，解决了版 本的升级困难，版本兼容性等各种问题，生产环境强烈推荐使用 

- 免费开源版本HortonWorks HDP版本--生产环境使用 

  官网地址：https://hortonworks.com/ 

  hortonworks主要是雅虎主导Hadoop开发的，核心产品软件HDP（ambari），HDF免费开源，并且提供一整套的web管理界面，供我们可以通 过web界面管理我们的集群状态，web管理界面软件HDF网址（http://ambari.apache.org/） 

##### Hadoop的优缺点

优点：

- Hadoop具有存储和处理数据能力的高可靠性
- 高扩展性，集群可方便的扩展到数以千计的节点中
- 高效性，Hadoop能够在节点之间进行动态的移动数据，并保证各个节点的动态平衡
- 高容错性，Hadoop能够自动保存数据的多个副本，并且能够自动将失败的任务重新分配

缺点：

- Hadoop不适用于低延迟数据访问
- Hadoop不能高效存储大量小文件
- Hadoop不支持多用户写入并任意修改文件

#### Hadoop的重要组成

Hadoop主要由HDFS、MapReduce、Yarn、和Common模块组成

- HDFS：一个高可靠、高吞吐量的分布式文件系统

  ![image-20200914192810495](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200914192810495.png)

  NameNode（NN）：存储文件的元数据。比如：文件名、文件目录结构、文件属性、以及每个文件的块列表和块所在的DataNode等

  SecondaryNameNode（2NN）：辅助NameNode更好的工作，用于监控HDFS状态，每隔一段时间获取HDFS元数据快照

  DataNode（DN）：在本地文件系统存储文件块数据，以及块数据的校验

  注：NN、2NN、DN既是角色名称、进程名称、也是电脑节点名称

- MapReduce计算=Map计算+Reduce计算

  Map阶段即“分”的阶段，并行处理输入数据

  Reduce阶段即“和”的阶段，对Map阶段结果进行汇总

  ![image-20200914195636871](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200914195636871.png)

- Hadoop Yarn：作业调度与集群资源管理

  ![image-20200914195825713](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200914195825713.png)

  - ResourceManager(rm)：处理客户端请求、启动/监控ApplicationMaster、监控NodeManager、资源分配与调度
  - NodeManager(nm)：单个节点上的资源管理、处理来自ResourceManager的命令，处理来自ApplicationMaster的命令
  - ApplicationMaster(am)：数据切分、申请资源、分配任务、任务监控与容错
  - container：任务运行环境的抽象，封装了CPU、内存等多维资源以及环境变量、启动命令等任务运行相关的信息

- Hadoop Common：工具模块（Configuration、RPC、序列化机制、日志操作）

##### HDFS

##### 相关概念

HDFS通过统一的命名空间目录树来定位文件

- 典型的Master/Slave架构

  HDFS的架构是典型的Master/Slave结构

  NameNode是集群的主节点，DataNode是集群的从节点

- 分块存储

  HDFS中的文件在物理上是分块存储的，块的大小可以通过配置参数来规定

- 命名空间

  HDFS支持传统的层次型文件组织结构。用户可以创建目录，然后将文件保存在这些目录里

- NameNode元数据管理

  NameNode的元数据记录每一个文件所对应的block信息

  元数据：文件的目录结构及分块位置信息称为元数据

- DataNode数据存储

  存储数据。一个block块可能会有多个DataNode存储；DataNode会定时向NameNode汇报自己持有的block信息

- 副本机制

  所有的block都会有副本，每个block大小和副本系数都是可配置的，副本数量默认是

- 一次写入，多次读出

  HDFS适合一次写入，多次读出的场景，不支持文件的随机修改（支持追加写入，不支持随机修改）。正因为如此，HDFS适合用来做大数据分析的底层存储服务，并不适合用来做网盘等应用（修改不方便，延迟大，网络开销大，成本太高）

##### HDFS架构

![image-20200914225655285](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200914225655285.png)

- NameNode：集群的管理者，Master节点
  - 维护命名空间
  - 维护副本策略
  - 记录文件块的映射信息
  - 负责处理客户端读写请求
- DataNode：接收NameNode命令，执行实际操作，Slave节点
  - 保存实际的数据块
  - 负责数据块的读写
- Client:客户端 
  - 上传文件到HDFS的时候，Client负责将文件切分成Block,然后进行上传 
  - 请求NameNode交互，获取文件的位置信息 
  - 读取或写入文件，与DataNode交互 
  - Client可以使用一些命令来管理HDFS或者访问HDFS

##### 客户端操作

- Shell命令行操作HDFS

  - 基本语法

    ```
    bin/hadoop fs 具体命令  OR  bin/hdfs dfs 具体命令
    ```

  - 常用命令

    ```
    -help 输出帮助信息
    -ls 显示目录信息
    -mkdir 在HDFS上创建目录
    -moveFromLocal 从本地剪切粘贴到HDFS
    -appendToFile 追加一个文件到已经存在的文件末尾
    -cat 显示文件内容
    -chgrp、-chmod、-chown 修改文件所属权限
    -copyFromLocal 从本地文件系统中拷贝文件到HDFS路径中
    -copyToLocal 从HDFS拷贝到本地
    -cp 拷贝
    -mv 移动文件
    -get 等同于copyToLocal 下载文件到本地
    -put 等同于copyFromLocal 上传文件到HDFS
    -tail 显示文件的末尾
    -rm 删除文件或文件夹
    -rmdir 删除空目录
    -du 统计文件夹的大小信息
    -setrep 设置HDFS中文件的副本数量
    ```

##### HDFS读写解析

###### HDFS读数据流程

![image-20200915003128926](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200915003128926.png)

- 客户端向NameNode请求下载文件，NameNode通过查询元数据，找到文件块所在的DataNode地址
- 就近且随机挑选DataNode服务器，请求读取数据
- DataNode开始传输数据给客户端，从磁盘里面读取数据输入流，以Packet为单位来做校验
- 客户端以Packet为单位接收，先在本地缓存，然后写入目标文件

######  HDFS写数据流程

![image-20200915003945953](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200915003945953.png)

- 客户端向NameNode请求上传文件，NameNode检查目标文件是否已存在，父目录是否存在
- NameNode返回是否可以上传
- 客户端请求第一个Block上传到哪几个DataNode服务器上
- NameNode返回3个DataNode节点，分别为dn1、dn2、dn3
- 客户端通过FSDataOutputStream模块请求dn1上传数据，dn1收到请求会继续调用dn2，然后dn2调用dn3，将这个通信管道建立完成
- dn1、dn2、dn3逐级应答客户端
- 客户端开始往dn1上传第一个Block（先从磁盘读取数据到一个本地内存缓存），以Packet为单位，dn1收到一个Packet就会传给dn2，dn2传给dn3，dn1每传一个Packet会放入确认队列等待确认
- 当一个Block传输完成之后，客户端再次请求NameNode上传第二个Block服务器

##### NN与2NN

###### HDFS元数据管理机制

问题：NameNode如何存储和管理元数据？

计算机中存储数据有两种方式：内存或磁盘

元数据存储在磁盘中：存储在磁盘里无法快速响应客户端的请求，但是安全性较高

元数据存储在内存中：可以高效响应客户端请求，缺点是数据易丢失

解决方案：内存+磁盘组合使用；NameNode内存+FsImange文件

问题：磁盘和内存中的元数据如何划分

答：磁盘中的数据与内存中的数据合并为完整数据。NameNode引入一个edits文件（仅追加写入），edits值记录客户端的增删改操作

元数据管理流程：

![image-20200915020534644](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200915020534644.png)

- 第一阶段：NmeNode启动
  - 第一次启动NameNode格式化后，创建Fsimage和edits文件（如果不是第一次启动，加载edits文件和Fsimage文件到内存）
  - 客户端对元数据进行增删改的请求
  - NameNode记录操作日志，更新滚动日志
  - NameNode在内存中对数据进行增删改
- 第二阶段：SecondaryNameNode工作
  - SecondaryNameNode询问NameNode是否需要CheckPoint。直接带回NameNode是否执行检查点操作结果
  - SecondaryNameNode请求执行CheckPoint
  - NameNode滚动正在写的edits日志
  - 将滚动前的edits日志和镜像文件拷贝到SecondaryNameNode
  - SecondaryNameNode加载edits和FSimage到内存中并合并 
  - 生成新的镜像文件fsimage.chkpoint
  - 拷贝fsimage.chkpoint到NameNode
  - NameNode将fsimage.chkpoint重命名为fsimage

###### Fsimage与Edits文件解析

NameNode格式化后，会产生以下文件：

![image-20200915142916914](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200915142916914.png)

- Fsimage文件

  是namenode中关于元数据的镜像，一般称为检查点，包含HDFS文件系统所有目录以及文件相关信息（Block数量，副本数量，权限等信息）

- Edits文件

  存储了客户端对HDFS文件系统的增删改操作记录

- seen_txid

  保存一个数字，该数字对应最后一个edits文件名的数字

- VERSION

  记录namenode的一些版本号信息

#####  限额与归档、集群安全模式

- HDFS文件限额配置

  - 数量限额

    ```
    hdfs dfsadmin -setQuota 2 /user/root/lagou
    
    # 清除文件数量限制
    hdfs dfsadmin -clrQuota /user/root/lagou
    
    # 查看hdfs文件限额数量
    hdfs dfs -count -q -h /user/root/lagou
    ```

  - 空间大小限额

    ```
    hdfs dfsadmin -setSpaceQuota 4k /user/root/lagou 
    
    # 取消空间限额
    hdfs dfsadmin -clrSpaceQuota /user/root/lagou
    ```

- HDFS归档技术

  主要解决HDFS集群存在大量小文件的问题

  Hadoop存档文件HAR文件，是一个高效的文件存档工具，HAR文件是由一组文件通过archive 工具创建而来，在减少了NameNode内存使用的同时，可以对文件进行透明的访问个一个独立的文件

- HDFS安全模式

  ```
  hdfs dfsadmin  -safemode
  ```

   安全模式是HDFS所处的一种特殊状态，在这种状态下，文件系统只接受读数据请求，而不接 受删除、修改等变更请求

##### MapReduce

MapReduce的核心思想是分而治之，充分利用了并行处理的优势

MapReduce的任务处理过程分为两个阶段

- Map阶段：“分”，把复杂任务分解为若干个“简单的任务”来并行处理；map阶段的任务彼此间没有依赖关系，可以进行并行计算

- Reduce：“和”，对map阶段的结果进行全局汇总

  ![image-20200916090131756](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200916090131756.png)

###### MapReduce原理分析

- MapReduce运行机制详解

  ![image-20200916090943815](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200916090943815.png)

  1. InputFormat组件通过getSplits()对输入目录中的文件进行逻辑切片得到splits（有多少个splits就有多少个MapTask），split与block的对应关系默认是一对一

  2. 将输入文件切分为splits之后，由RecordReader对象进行读取。读取一行数据，返回<key,value>(key表示每行首字符偏移值，value表示这一行文本内容)

  3. 执行map函数（RecordReader读取一行调用一次）

  4. 将map的每条结果通过context.write进行collect数据收集。在collect中，会先对其进行分区处理，默认使用HashPartitioner

     MapReduce提供Partitioner接口，它的作用就是根据key或value及reduce的数量来决定当前的这对 输出数据最终应该交由哪个reduce task处理。默认对key hash后再以reduce task数量取模。默认的 取模方式只是为了平均reduce的处理能力，如果用户自己对Partitioner有需求，可以订制并设置到 job上

  5. 将数据写入内存，内存中这片区域叫做环形缓冲区，缓冲区的作用是批量收集map结果，减少磁盘IO 的影响

     - 环形缓冲区其实是一个数组，数组中存放着key，value的序列化数据和key，value的元数据信息
     - 缓冲区默认100MB。当map task的输出结果很多时，就可能会撑爆内存，所以 需要在一定条件下将缓冲区中的数据临时写入磁盘，然后重新利用这块缓冲区。这个从内存往磁盘 写数据的过程被称为Spill，中文可译为溢写。这个溢写是由单独线程来完成，不影响往缓冲区写 map结果的线程。溢写线程启动时不应该阻止map的结果输出，所以整个缓冲区有个溢写的比例 spill.percent。这个比例默认是0.8，也就是当缓冲区的数据已经达到阈值（buffer size * spill percent = 100MB * 0.8 = 80MB），溢写线程启动，锁定这80MB的内存，执行溢写过程。Map task的输出结果还可以往剩下的20MB内存中写，互不影响

  6. 当溢写线程启动后，会对锁定空间内的key进行排序

     - 如果job设置过Combiner，那么现在就是使用Combiner的时候了。将有相同key的key/value对的 value加起来，减少溢写到磁盘的数据量。Combiner会优化MapReduce的中间结果，所以它在整 个模型中会多次使用
     - 那哪些场景才能使用Combiner呢？从这里分析，Combiner的输出是Reducer的输入，Combiner 绝不能改变终的计算结果。Combiner只应该用于那种Reduce的输入key/value与输出key/value 类型完全一致，且不影响终结果的场景。比如累加，大值等。Combiner的使用一定得慎重， 如果用好，它对job执行效率有帮助，反之会影响reduce的终结果

  7. 合并溢写文件：每次溢写会在磁盘上生成一个临时文件（写之前判断是否有combiner），如果 map的输出结果真的很大，有多次这样的溢写发生，磁盘上相应的就会有多个临时文件存在。当 整个数据处理结束之后开始对磁盘中的临时文件进行merge合并，因为终的文件只有一个，写入 磁盘，并且为这个文件提供了一个索引文件，以记录每个reduce对应数据的偏移量

- ReduceTask工作机制

  ![image-20200916092609485](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200916092609485.png)

  Reduce大致分为copy、sort、reduce三个阶段，重点在前两个阶段

  - Copy阶段，简单地拉取数据。Reduce进程启动一些数据copy线程(Fetcher)，通过HTTP方式请求 maptask获取属于自己的文件
  - Merge阶段。这里的merge如map端的merge动作，只是数组中存放的是不同map端copy来的数 值。Copy过来的数据会先放入内存缓冲区中，这里的缓冲区大小要比map端的更为灵活。merge 有三种形式：内存到内存；内存到磁盘；磁盘到磁盘。默认情况下第一种形式不启用。当内存中的 数据量到达一定阈值，就启动内存到磁盘的merge。与map 端类似，这也是溢写的过程，这个过 程中如果你设置有Combiner，也是会启用的，然后在磁盘中生成了众多的溢写文件。第二种 merge方式一直在运行，直到没有map端的数据时才结束，然后启动第三种磁盘到磁盘的merge 方式生成终的文件
  - 合并排序。把分散的数据合并成一个大的数据后，还会再对合并后的数据排序
  - 对排序后的键值对调用reduce方法，键相等的键值对调用一次reduce方法，每次调用会产生零个 或者多个键值对，后把这些输出的键值对写入到HDFS文件中

- ReduceTask并行度

  ```
  job.setNumReduceTasks(4);
  ```

  注：

  - ReduceTask=0，表示没有reduce阶段，输出文件数和mapTask数量一致
  - 默认数量为1
  - 如果数据分布不均匀，可能在Reduce阶段产生倾斜

- Shuffle机制

  map()方法之后，reduce()方法之前这中间的处理过程称为shuffle

  核心机制：数据分区、排序、分组、合并、combine等过程

  - combine

    - Combiner是MR程序中Mapper和Reducer之外的一种组件 
    -  Combiner组件的父类就是Reducer 
    - Combiner和reducer的区别在于运行的位置
    - Combiner是在每一个maptask所在的节点运行
    - . Combiner的意义就是对每一个maptask的输出进行局部汇总，以减小网络传输量
    -  Combiner能够应用的前提是不能影响终的业务逻辑，此外，Combiner的输出kv应该跟reducer 的输入kv类型要对应起来

  - 排序

    MapTask和ReduceTask均会对数据按照key进行排序。该操作属于Hadoop的默认行为。任何应用程序 中的数据均会被排序，而不管逻辑.上是否需要。默认排序是按照字典顺序排序，且实现该排序的方法是 快速排序

  -  WritableComparable 

    Bean对象如果作为Map输出的key时，需要实现WritableComparable 接口并重写compareTo(）fan方法

- MapReduce分区与ReduceTask数量

  - 自定义分区保证分区数量与reduceTask数量保持一致
  - 如果分区数量不止1个，但是reduceTask数量1个，此时只会输出一个文件
  -  如果reduceTask数量大于分区数量，但是输出多个空文件 
  -  如果reduceTask数量小于分区数量，有可能会报错

##### Yarn

###### Yarn架构

![image-20200916101652916](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200916101652916.png)



###### Yarn工作机制

- 作业提交

  1. Client调用job.waitForCompletion方法，向整个集群提交MapReduce作业
  2. Client向RM申请一个作业id
  3. RM给Client返回该job资源的提交路径和作业id
  4. Client提交jar包、切片信息和配置文件到指定的资源提交路径
  5. Client提交完资源后，向RM申请运行MrAppMaster

- 作业初始化

  1. 当RM收到Client的请求后，将该job添加到容量调度器中
  2. 某一个空闲的NM领取到该Job
  3. 该NM创建Container，并产生MRAppmaster
  4. 下载Client提交的资源到本地

- 任务分配

  1. MrAppMaster向RM申请运行多个MapTask任务资源
  2. RM将运行MapTask任务分配给另外两个NodeManager，另两个NodeManager分 别领取任务并创建容器

- 任务运行

  1. MR向两个接收到任务的NodeManager发送程序启动脚本，这两个NodeManager 分别启动MapTask，MapTask对数据分区排序
  2. MrAppMaster等待所有MapTask运行完毕后，向RM申请容器，运行ReduceTask
  3. ReduceTask向MapTask获取相应分区的数据
  4. 程序运行完毕后，MR会向RM申请注销自己

- 进度和状态更新

  YARN中的任务将其进度和状态返回给应用管理器, 客户端每秒(通过 mapreduce.client.progressmonitor.pollinterval设置)向应用管理器请求进度更新, 展示给用 户

- 作业完成

  除了向应用管理器请求作业进度外, 客户端每5秒都会通过调用waitForCompletion()来检查作 业是否完成。时间间隔可以通过mapreduce.client.completion.pollinterval来设置。作业完 成之后, 应用管理器和Container会清理工作状态。作业的信息会被作业历史服务器存储以备 之后用户核查

###### Yarn调度策略

Hadoop作业调度器主要有三种：FIFO、Capacity Scheduler和Fair Scheduler。Hadoop2.9.2默认的资 源调度器是Capacity Scheduler

- FIFO
- Capacity Scheduler
- Fair Scheduler

###### Yarn多租户资源隔离配置

Yarn集群资源设置为A,B两个队列

- A队列设置占用资源70%主要用来运行常规的定时任务

- B队列设置占用资源30%主要运行临时任务

- 两个队列间可相互资源共享，假如A队列资源占满，B队列资源比较充裕，A队列可以使用B队列的 资源，使总体做到资源利用大化

  选择使用Fair Scheduler调度策略

具体配置

1. yarn-site.xml配置

   ```xml
   <!--  指定我们的任务调度使用fairScheduler的调度方式  --> <property>    <name>yarn.resourcemanager.scheduler.class</name>    <value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.fair.FairSch eduler</value>    <description>In case you do not want to use the default scheduler</description> </property>
   ```

2. 创建fair-scheduler.xml文件

   在hadoop安装目录/etc/hadoop创建

   ```xml
   <?xml version="1.0" encoding="UTF-8" standalone="yes"?> <allocations>    <defaultQueueSchedulingPolicy>fair</defaultQueueSchedulingPolicy>    <queue name="root" >        <queue name="default">            <aclAdministerApps>*</aclAdministerApps>            <aclSubmitApps>*</aclSubmitApps>            <maxResources>9216 mb,4 vcores</maxResources>            <maxRunningApps>100</maxRunningApps>            <minResources>1024 mb,1vcores</minResources>            <minSharePreemptionTimeout>1000</minSharePreemptionTimeout>            <schedulingPolicy>fair</schedulingPolicy>            <weight>7</weight>        </queue>        <queue name="queue1">            <aclAdministerApps>*</aclAdministerApps>            <aclSubmitApps>*</aclSubmitApps>            <maxResources>4096 mb,4vcores</maxResources>            <maxRunningApps>5</maxRunningApps>            <minResources>1024 mb, 1vcores</minResources>            <minSharePreemptionTimeout>1000</minSharePreemptionTimeout>            <schedulingPolicy>fair</schedulingPolicy>            <weight>3</weight>         </queue>    </queue>       <queuePlacementPolicy>        <rule create="false" name="specified"/>        <rule create="true" name="default"/>    </queuePlacementPolicy> </allocations>
   ```

   