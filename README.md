# Fire-Spark
    让Spark开发变得更简单
### BUILD和使用方法
##### 1.构建方法

```
    $git clone https://github.com/GuoNingNing/fire-spark.git
    $cd fire-spark
    $mvn install clean package -DskipTests

    构建完成之后会将jar安装到m2的对应路径下，使用时在自己的项目的pom.xml文件里添加
    
    <dependency>
        <groupId>org.fire.spark.streaming</groupId>
    	<artifactId>fire-spark</artifactId>
    	<version>2.1.0_kafka-0.10</version>
    </dependency>
```

##### 2.配置说明

```

可使用script 中的create_default_conf.sh创建默认配置文件,使用方法:
$bash create_default_conf.sh >my.properties


生成的标准配置内容
此配置文件可以自定义添加任何以spark开头的配置参数,spark程序本身的依赖参数也在此配置文件配置

######################################################
#                                                    #
#               spark process run.sh                 #
#                   user config                      #
#                                                    #
######################################################
#必须设置,执行class的全包名称
spark.run.main=z.cloud.t3.Demon

#必须设置,包含main class的jar包
#jar必须包含在lib.path当中
spark.run.main.jar=t3-1.0-SNAPSHOT.jar

#提供给执行class的命令行参数,多个参数之间用逗号隔开,参数中不能包含空格等空白符
#Ex:param1,param2,..
spark.run.self.params=--checkpointPath,/tmp/checkpoint

#可以是绝对路径,也可以是相对此配置文件的相对路径
#相对路径会自动补全
spark.lib.path=lib

######################################################
#                                                    #
#                spark self config                   #
#                                                    #
######################################################
#执行集群设置,不用设置,一般使用YARN
spark.master=yarn

#YARN部署模式
#default=cluster
spark.submit.deployMode=cluster

#spark-streaming每个批次间隔时间
#default=300
spark.batch.duration=30

#spark网络序列化方式,默认是JavaSerializer,可针对所有类型但速度较慢
#这里使用推荐的Kryo方式
#kafka-0.10必须使用此方式
spark.serializer=org.apache.spark.serializer.KryoSerializer

#++++++++++++++++++++++Driver节点相关配置+++++++++++++++++++++++++++
#Driver节点使用内存大小设置
#default=1G
spark.driver.memory=512M

#Driver节点使用的cpu个数设置
#default=1
spark.driver.cores=1

#Driver节点构建时spark-jar和user-jar冲突时优先使用用户提供的,这是一个实验性质的参数只对cluster模式有效
#default=false
spark.driver.userClassPathFirst=false

#++++++++++++++++++++++Executor节点相关配置+++++++++++++++++++++++++
#Executor个数设置
#default=1
spark.executor.instances=1

#Executor使用cpu个数设置
#default=1
spark.executor.cores=1

#Executor使用内存大小设置
#default=1G
spark.executor.memory=512M

#同driver节点配置作用相同,但是是针对executor的
#default=false
spark.executor.userClassPathFirst=true

#++++++++++++++++++++++++Executor动态分配相关配置++++++++++++++++++++
#Executor动态分配的前置服务
#default=false
spark.shuffle.service.enabled=true

#服务对应的端口,此端口服务是配置在yarn-site中的,由NodeManager服务加载启动
#default=7337
spark.shuffle.service.port=7337

#配置是否启用资源动态分配,此动态分配是针对executor的,需要yarn集群配置支持动态分配
#default=false
spark.dynamicAllocation.enabled=true

#释放空闲的executor的时间
#default=60s
spark.dynamicAllocation.executorIdleTimeout=60s

#有缓存的executor空闲释放时间
#default=infinity(默认不释放)
#spark.dynamicAllocation.cachedExecutorIdleTimeout=

#初始化executor的个数,如果设置executor-number,谁小用谁
#default=minExecutors(不设置使用此项配置值)
spark.dynamicAllocation.initialExecutors=1

#executor动态分配可分配最大数量
#default=infinity
spark.dynamicAllocation.maxExecutors=60

#executor动态收缩的最小数量
#default=0
spark.dynamicAllocation.minExecutors=1

#批次调度延迟多长时间开始增加executor
#default=1s
spark.dynamicAllocation.schedulerBacklogTimeout=1s

#同上,但是是针对之后的请求
#default=SchedulerBacklogTimeout(不设置使用此项配置值)
spark.dynamicAllocation.sustainedSchedulerBacklogTimeout=1s

######################################################
#                                                    #
#                  spark process                     #
#                  Source config                     #
#                                                    #
######################################################
#
#spark.source.kafka.consume 后的配置是kafka标准配置,指定之后,将会自动传递给kafkaConsumer
#
#
spark.source.kafka.consume.topics=homework_sync_canal
#
spark.source.kafka.consume.group.id=z.cloud.kafka.consumer.0018
#
spark.source.kafka.consume.bootstrap.servers=kafka1:9092,kafka2:9092
spark.source.kafka.consume.auto.offset.reset=earliest
#offset.store.type是指定如何管理offset的.不指定则匿名消费,kafka表示由kafka自己管理,另有redis和hbase的管理方式
spark.source.kafka.offset.store.type=kafka
#使用redis管理offset时的配置依赖项
#spark.source.kafka.offset.store.type=redis
#spark.source.kafka.offset.store.redis.hosts=localhost
#spark.source.kafka.offset.store.redis.port=6379
#
spark.source.kafka.consume.key.deserializer=org.apache.kafka.common.serialization.StringDeserializer
spark.source.kafka.consume.value.deserializer=org.apache.kafka.common.serialization.StringDeserializer
spark.source.kafka.consume.max.partition.fetch.bytes=10485760
spark.source.kafka.consume.fetch.max.wait.ms=3000
#
#
######################################################
#                                                    #
#                  spark process                     #
#                   Sink config                      #
#                                                    #
######################################################
#目前暂不支持自动的sink配置,此配置需用户在自己的代码里通过sparkConf.get("spark.sink.hdfs.path")的方式获取使用
spark.sink.hdfs.path=/tmp/test
spark.sink.redis.host=192.168.1.1
spark.sink.redis.port=6379
spark.sink.redis.db=0
spark.sink.redis.timeout=30


```
  
##### 3.示例代码  
  
```scala    
package z.cloud.t3

import org.apache.spark.streaming.StreamingContext
import org.fire.spark.streaming.core.FireStreaming
import org.fire.spark.streaming.core.plugins.kafka.KafkaDirectSource

object Demon extends FireStreaming {
    override def handle(ssc : StreamingContext): Unit = {
        val source = new KafkaDirectSource[String,String](ssc)
        //val conf = ssc.sparkContext.getConf
        source.getDStream[(String,String)](m => (m.topic,m.value)).foreachRDD((rdd,time) => {
            rdd.take(10).foreach(println)
            source.updateOffsets(time.milliseconds)
        })
    }
}


```

##### 4.运行示例代码

```
    使用script中的create_default_conf.sh创建标准配置文件
    
    $bash create_default_conf.sh >my.properties
    
    创建配置文件之后按配置文件按提示设置必须的参数
    
    spark.run.main=z.cloud.t3.Demon
    spark.run.main.jar=t3-1.0.jar
    
    设置完成后使用script中的run.sh来提交和停止任务
    启动
    $bash run.sh my.properties
    停止(任意给第二个参数即可kill掉spark任务)
    $bash run.sh my.properties stop
    
```
