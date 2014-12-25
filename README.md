#Spark SQL HBase Connector
 
 
 _Spark SQL HBase Connector_ aim to query HBase Table by using Spark SQL.
 
 It leverages the functionality of [Spark SQL](http://spark.apache.org/sql/) 1.2+ external datasource API .

> 作为一个中国人，我还是写几句中文 ：）
Spark1.2发布之后，Spark SQL支持了External Datasource API，我们才能方便的编写扩展来使Spark SQL能支持更多的外部数据源。
鉴于国内用HBase的用户比较多，所以使Spark SQL支持查询HBase还是很有价值的，这也是我写这个Lib的原因。
不过，一个人得力量远不如大家的力量，所以希望大家能多提Issues，多多Commit~ 先谢谢了：）


####Concepts
Let me explain two concepts first, this will help you understand the the connector:

__1、External Table__

For HBase Table:
We call it `external table` because it's outside of spark sql.

__2、Register Table__

As We know for one specific `HBase Table` there should be a `Spark SQL Table` which is a mapping of it in Spark SQL. We call it `registerTable`.


####Using SQL Resiger HBase Table

####1. Query by Spark SQL
  ```scala
   import org.apache.spark.sql.SQLContext  
   import sqlContext._

   val sqlContext  = new SQLContext(sc)  
   val hbaseDDL = s"""
      |CREATE TEMPORARY TABLE hbase_people
      |USING com.shengli.spark.hbase
      |OPTIONS (
      |  registerTableSchema   '(row_key string, name string)',
      |  externalTableName    'people',
      |  externalTableSchema '(rowkey:rowkey string, profile:name string)'
      |)""".stripMargin
	  
	sqlContext.sql(hbaseDDL)
	sql("select row_key,name from hbase_people").collect()
```

Let's see the result:

__select__

```
scala> sql("select row_key,name from hbase_people").collect()
14/12/26 00:42:13 INFO scheduler.TaskSchedulerImpl: Removed TaskSet 3.0, whose tasks have all completed, from pool 
14/12/26 00:42:13 INFO scheduler.DAGScheduler: Job 3 finished: collect at SparkPlan.scala:81, took 0.140132 s
res11: Array[org.apache.spark.sql.Row] = Array([rowkey001,Sheng,Li], [rowkey002,Li,Lei], [rowkey003,Jim Green], [rowkey004,Lucy], [rowkey005,HanMeiMei])
```

__count(1)__

```scala
scala> sql("select count(1) from hbase_people").collect()
14/12/26 01:05:39 INFO scheduler.DAGScheduler: Job 1 finished: collect at SparkPlan.scala:81, took 0.442987 s
res6: Array[org.apache.spark.sql.Row] = Array([5])
```

####2. Query by SQLContext API

Firstly, import `import com.shengli.spark.hbase._`
Secondly, use `sqlContext.hbaseTable` _API_ to generate a `SchemaRDD`
The `sqlContext.hbaseTable` _API_ need serveral parameters.

```scala
hbaseTable(registerTableSchema: String, externalTableName: String, externalTableSchema: String)__
```

Let me give you a detail example:

```scala
scala> import com.shengli.spark.hbase._
import com.shengli.spark.hbase._

scala> val hbaseSchema = sqlContext.hbaseTable("(row_key string, name string)","people","(rowkey:rowkey string, profile:name string)")
......
14/12/26 00:13:09 INFO spark.SparkContext: Created broadcast 3 from newAPIHadoopRDD at HBaseRelation.scala:113
hbaseSchema: org.apache.spark.sql.SchemaRDD = 
SchemaRDD[13] at RDD at SchemaRDD.scala:108
== Query Plan ==
== Physical Plan ==
PhysicalRDD [row_key#4,name#5], MapPartitionsRDD[16] at map at HBaseRelation.scala:121
```

We've got a hbaseSchema so that we can query it with DSL or register it as a temp table query with sql, do whatever you like:

```scala
scala> hbaseSchema.select('name).collect()
14/12/26 00:14:30 INFO util.RegionSizeCalculator: Calculating region sizes for table "people".
14/12/26 00:14:30 INFO spark.SparkContext: Starting job: collect at SparkPlan.scala:81
14/12/26 00:14:30 INFO scheduler.DAGScheduler: Got job 1 (collect at SparkPlan.scala:81) with 1 output partitions (allowLocal=false)
14/12/26 00:14:30 INFO scheduler.DAGScheduler: Final stage: Stage 1(collect at SparkPlan.scala:81)
14/12/26 00:14:30 INFO scheduler.DAGScheduler: Parents of final stage: List()
......
14/12/26 00:14:30 INFO scheduler.DAGScheduler: Job 1 finished: collect at SparkPlan.scala:81, took 0.205903 s
res9: Array[org.apache.spark.sql.Row] = Array([Sheng,Li], [Li,Lei], [Jim Green], [Lucy], [HanMeiMei])
```

##HBase Data

Let's take look at the `HBase Table` named `person`

The `schema` of the table `person`:

__column family__: `profile`, `career`

__coloumns__:`profile:name`, `profile:age`,`carrer:job`


```java
1.8.7-p357 :024 > scan 'people'
ROW                                  COLUMN+CELL                                                                                               
 rowkey001                           column=career:job, timestamp=1419517844784, value=software engineer                                       
 rowkey001                           column=profile:age, timestamp=1419517844665, value=25                                                     
 rowkey001                           column=profile:name, timestamp=1419517844501, value=Sheng,Li                                              
 rowkey002                           column=career:job, timestamp=1419517844813, value=teacher                                                 
 rowkey002                           column=profile:age, timestamp=1419517844687, value=26                                                     
 rowkey002                           column=profile:name, timestamp=1419517844544, value=Li,Lei                                                
 rowkey003                           column=career:job, timestamp=1419517844832, value=english teacher                                         
 rowkey003                           column=profile:age, timestamp=1419517844704, value=24                                                     
 rowkey003                           column=profile:name, timestamp=1419517844568, value=Jim Green                                             
 rowkey004                           column=career:job, timestamp=1419517844853, value=doctor                                                  
 rowkey004                           column=profile:age, timestamp=1419517844724, value=23                                                     
 rowkey004                           column=profile:name, timestamp=1419517844589, value=Lucy                                                  
 rowkey005                           column=career:job, timestamp=1419517845664, value=student                                                 
 rowkey005                           column=profile:age, timestamp=1419517844744, value=18                                                     
 rowkey005                           column=profile:name, timestamp=1419517844606, value=HanMeiMei                                             
5 row(s) in 0.0260 seconds
```

###Note:

####Package

In the root directory,  use `sbt package` to package the lib.

####Dependency

__1. hbase-site.xml__

You need place `hbase-site.xml` under the spark classpath. Also need to configure it correctly first.
Below is my hbase-site.xml:

```scala
<configuration>
 <property>
     <name>hbase.rootdir</name>
     <value>file:///Users/shengli/software/data/hbase</value>
 </property>
 <property>
     <name>hbase.cluster.distributed</name>
         <value>true</value>
 </property>
 <property>
      <name>hbase.zookeeper.property.clientPort</name>
               <value>2181</value>
  </property>

 <property>
     <name>hbase.zookeeper.quorum</name>
     <value>localhost</value>
  </property>
  <property>
      <name>hbase.defaults.for.version.skip</name>
          <value>true</value>
  </property>
</configuration>
```

You can simply do it with `ln -s ~/software/hbase/conf/hbase-site.xml ~/git_repos/spark`

__2. Add hbase related libs into spark classpath__

Below is how I start the spark shell:

```scala
bin/spark-shell --master spark://192.168.2.101:7077 --jars /Users/shengli/software/hbase/lib/hbase-client-0.98.8-hadoop2.jar,/Users/shengli/software/hbase/lib/hbase-server-0.98.8-hadoop2.jar,/Users/shengli/software/hbase/lib/hbase-common-0.98.8-hadoop2.jar,/Users/shengli/software/hbase/lib/hbase-protocol-0.98.8-hadoop2.jar,/Users/shengli/software/hbase/lib/protobuf-java-2.5.0.jar,/Users/shengli/software/hbase/lib/htrace-core-2.04.jar,/Users/shengli/git_repos/spark-hbase/target/scala-2.10/spark-hbase_2.10-0.1.jar
```

__3. class not found issues__

The below provides the mapping of the classes and their respective jars
Class Name	Jar Name
TableSplit	hbase-server.jar
HTable	hbase-client.jar
MasterProtos	hbase-protocol.jar
org.cloudera.htrace.Trace	htrace-core-2.01.jar

- https://support.pivotal.io/hc/en-us/articles/203025186-Hive-Query-from-Tableau-failed-with-error-Execution-Error-return-code-2-from-org-apache-hadoop-hive-ql-exec-mr-MapRedTask

###Contact Me
如果有任何疑问，可以通过以下方式联系我：
- WeiBo: http://weibo.com/oopsoom
- Blog: http://blog.csdn.net/oopsoom
- 邮箱：<victorshengli@126.com> 常用
- 邮箱：<victorsheng117@gmail.com> 需要VPN，不常使用