#### 架构介绍

| 框架 | linux121           | linux122                     | linux123                    |
| ---- | ------------------ | ---------------------------- | --------------------------- |
| HDFS | NameNode、DataNode | DataNode                     | SecondaryNameNode、DataNode |
| YARN | NodeManager        | ResourceManager、NodeManager | NodeManager                 |

#### 前置准备

- 修改主机名

  ```
  hostnamectl set-hostname newName;
  ```

- 配置jdk运行环境

#### 安装步骤

- 准备安装包hadoop-2.9.2.tar.gz ，并上传至linux121服务器

- 解压安装包到指定目录

  ```
  tar -zxvf hadoop-2.9.2.tar.gz -C /opt/lagou/servers
  ```

- 添加环境变量

  ```
  vim /etc/profile
  
  # HADOOP_HOME
  export HADOOP_HOME=/opt/lagou/servers/hadoop-2.9.2 export PATH=$PATH:$HADOOP_HOME/bin export PATH=$PATH:$HADOOP_HOME/sbin
  ```

- 使环境变量生效

  ```
  source /etc/profile
  ```

- 验证hadoop

  ```
  hadoop version
  ```

- 集群配置

  Hadoop集群配置 = HDFS集群配置+MapReduce集群配置+Yarn集群配置

  - HDFS集群配置

    - hadoop-env.sh 指定JDK路径

      ```
      export JAVA_HOME=/opt/lagou/servers/jdk1.8.0_231
      
      ```

    - core-site.xml指定NameNode节点以及数据存储目录

      ```
      <!-- 指定HDFS中NameNode的地址 -->
              <property>
                      <name>fs.defaultFS</name>                                               <value>hdfs://linux121:9000</value> 
               </property>
              <!-- 指定Hadoop运行时产生文件的存储目录 --> 
               <property>
                      <name>hadoop.tmp.dir</name>                                             <value>/root/software/hadoop-2.9.2/data/tmp</value>    
              </property
      
      ```

    - hdfs-site.xml指定SecondaryNameNode节点

      ```
      <!-- 指定Hadoop辅助名称节点主机配置 --> 
      <property>      								<name>dfs.namenode.secondary.http-address</name>     								 <value>linux123:50090</value> </property>
      <!--副本数量 --> 
      <property>       
      	<name>dfs.replication</name>        		<value>3</value> 
      </property>
      
      ```

    - etc/hadoop/slaves指定DataNode从节点，每个节点配置信息占一行

      ```
      linux121 
      linux122 
      linux123
      ```

    注：该文件中添加的内容结尾不允许有空格，文件中不允许有空行

  - MapReduce集群配置

    - mapred-env.sh指定JDK路径

      ```
      export JAVA_HOME=/opt/lagou/servers/jdk1.8.0_231
      ```

    - mapred-site.xml指定MapReduce计算框架运行Yarn资源调度框架

      ```
      <!-- 指定MR运行在Yarn上 --> 
      <property>        								<name>mapreduce.framework.name</name> 	       <value>yarn</value> 
      </property>
      ```

  - Yarn集群配置

    - yarn-env.sh指定JDK路径

      ```
      export JAVA_HOME=/opt/lagou/servers/jdk1.8.0_231
      ```

    - yarn-site.xml指定ResourceManager所在的计算机节点

      ```
      <!-- 指定YARN的ResourceManager的地址 --> <property>        								<name>yarn.resourcemanager.hostname</name>     <value>linux122</value>
      </property>
      <!-- Reducer获取数据的方式 --> 
      <property>       	 					<name>yarn.nodemanager.aux-services</name>        <value>mapreduce_shuffle</value> 
      </property>
      ```

  - 修改Hadoop安装目录所属用户和用户组

    非必须，Hadoop安装目录默认所属用户和用户组为501 dialout，为避免信息混乱，所以修改一下

    ```
    chown -R root:root /opt/lagou/servers/hadoop-2.9.2
    ```

- 分发配置

  编写集群分发脚本rsync-script

  - rsync远程同步工具

    rsync主要用于备份和镜像。具有速度快、避免复制相同内容和支持符号链接的优点

    - 基本语法

      rsync 选项参数 要拷贝的文件路径/名称 目的用户@主机：目的路径/名称

      ```
      rsync   -rvl    $pdir/$fname        $user@$host:$pdir/$fnameS
      ```

    | 选项 | 功能         |
    | ---- | ------------ |
    | -r   | 递归         |
    | -v   | 显示复制过程 |
    | -l   | 拷贝符号连接 |

     安装rsync

    ```
    三台虚拟机安装rsync（执行安装需要联网）
    yum install -y rsync
    ```

  - 编写rsync-script脚本

    注：在/usr/local/bin这个目录下存放的脚本，root用户可在系统任何地方直接执行

    ```
    cd /usr/local/bin
    vim  rsync-script
    ```

    脚本内容如下：

    ```
    #!/bin/bash
    #1 获取命令输入参数的个数，如果个数为0，直接退出命令 paramnum=$# 
    if((paramnum==0)); then 
    echo no params; 
    exit;
    fi
    
    #2 根据传入参数获取文件名称 
    p1=$1 
    file_name=`basename $p1` 
    echo fname=$file_name
    
    #3 获取输入参数的绝对路径
    pdir=`cd -P $(dirname $p1); pwd` 
    echo pdir=$pdir
    
    #4 获取用户名称
    user=`whoami`
    
    #5 循环执行rsync
    for((host=121; host<124; host++)); do
    echo ------------------- linux$host --------------  rsync -rvl $pdir/$file_name $user@linux$host:$pdir
     
    done
    ```

    修改rsync-script执行权限

    ```
    chmod 777 rsync-script
    ```

    脚本调用形式：rsync-script 文件名称

  - 分发脚本

    ```
    rsync-script /opt/lagou/servers/hadoop-2.9.2
    ```

- 启动集群

  注：！！！如果集群是第一次启动，需要在NameNode所在节点进行格式化操作（可以单节点启动，也可借助脚本启动）

  ```
   hadoop namenode -format
  ```

  - 单节点启动

    - linux121启动NameNode

      ```
      hadoop-daemon.sh start namenode 
      ```

    - linux121、linux122、linux123启动DataNode

      ```
      hadoop-daemon.sh start datanode
      ```

    - web端查看HDFS界面

      http://linux121:50070/dfshealth.html#tab-overview

    - Yarn集群单节点启动

      ```
       yarn-daemon.sh start resourcemanager
      ```

  - 集群启动

    - 启动HDFS（在NameNode所在节点上执行）

      ```
      sbin/start-dfs.sh 
      ```

    - 启动Yarn（在ResourceManager所在节点上执行）

      ```
      sbin/start-yarn.sh
      ```

- 集群测试

  - 验证HDFS

    ```
    #上传linxu文件到Hdfs
    hdfs dfs -put /root/test.txt  /test/input 
    
    #从Hdfs下载文件到linux本地
    hdfs dfs -get /test/input/test.txt
    ```

  - 验证MapReduce

    - 在HDFS文件系统下创建一个wcinput文件夹

      ```
      hdfs dfs -mkdir /wcinput
      ```

    - 编辑wc.txt

      ```
      cd /root
      vi wc.txt
      
      #wc.txt文件内容
      hadoop mapreduce yarn hdfs hadoop mapreduce mapreduce yarn lagou lagou lagou
      ```

    - 上传wc.txt至/wcinput

      ```
      hdfs dfs -put wc.txt /wcinput
      ```

    - 执行hadoop自带的wordcount程序

      ```
      cd /opt/lagou/servers/hadoop-2.9.2 
      
      hadoop jar
       share/hadoop/mapreduce/hadoop-mapreduce-examples-2.9.2.jar wordcount /wcinput /wcoutput
      ```

    - 查看结果

      ```
      hdfs dfs -cat /wcoutput/part-r-00000
      ```

#### 配置历史服务器

在Yarn中运行的任务产生的日志数据不能查看，为了查看程序的历史运行情况，需要配置一下历史日志服务器

- 配置mapred-site.xml

  ```
  #增加如下配置
  <!-- 历史服务器端地址 -->
  <property>    <name>mapreduce.jobhistory.address</name>    <value>linux121:10020</value> </property>
  <!-- 历史服务器web端地址 -->
  <property>    <name>mapreduce.jobhistory.webapp.address</name>    <value>linux121:19888</value> </property>
  ```

- 分发mapred-site.xml到其它节点

  ```
  rsync-script mapred-site.xml
  ```

- 启动历史服务器

  ```
  sbin/mr-jobhistory-daemon.sh start historyserver
  ```

- 查看JobHistory

  ```
  http://linux121:19888/jobhistory
  ```

#### 配置日志的聚集

日志聚集：应用(Job)运行完成以后，将应用运行日志信息从各个task汇总上传到HDFS系统上

日志聚集功能好处：可以方便的查看到程序运行详情，方便开发调试

注：开启日志聚集功能，需要重新启动NodeManager、ResourceManager和HistoryManager

- 配置yarn-site.xml

  ```
  <!-- 日志聚集功能使能 -->
  <property> <name>yarn.log-aggregation-enable</name> <value>true</value> </property>
   
  <!-- 日志保留时间设置7天 --> 
  <property> <name>yarn.log-aggregation.retain-seconds</name> <value>604800</value> </property>
  ```

- 分发yarn-site.xml到集群其它节点

  ```
  rsync-script yarn-site.xml
  ```

- 关闭NodeManager 、ResourceManager和HistoryManager

  ```
  sbin/yarn-daemon.sh stop resourcemanager
  sbin/yarn-daemon.sh stop nodemanager
  sbin/mr-jobhistory-daemon.sh stop historyserver
  ```

- 启动NodeManager 、ResourceManager和HistoryManager

  ```
  sbin/yarn-daemon.sh start resourcemanager
  sbin/yarn-daemon.sh start nodemanager
  sbin/mr-jobhistory-daemon.sh start historyserver
  ```

- 删除HDFS上已经存在的输出文件

  ```
  bin/hdfs dfs -rm -R /wcoutput
  ```

-  执行WordCount程序

  ```
  hadoop jar share/hadoop/mapreduce/hadoopmapreduce-examples-2.9.2.jar wordcount /wcinput /wcoutput
  ```

- 查看日志，如图所示 

  ```
  http://linux121:19888/jobhistory
  ```

#### 安装过程中遇见的错误

1. 不能解析主机名

![image-20200910210726711](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200910210726711.png)

解决方案：修改三台服务器的/etc/hosts，添加如下配置，绑定ip地址与主机名

```
192.168.6.138 linux121
192.168.6.133 linux122
192.168.6.131 linux123
```

2.无法访问页面

![image-20200910224518077](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200910224518077.png)

解决方案：换个浏览器！