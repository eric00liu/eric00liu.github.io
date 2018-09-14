---
title: storm系统搭建
date: 2018/7/6 17:32:22
tags:
- storm
- 异常检测
- nimubs
- k8s
- docker
---
### 通过k8s搭建storm系统
参考这篇说明：https://github.com/kubernetes/examples/tree/master/staging/storm

### 运行第一个storm topology
- 启动装有storm的docker
```
docker run -it -w /opt/apache-storm mattf/storm-base sh
```
- 配置ZK和nimbus
```
/configure.sh  zkClusterIp nimbusClusterIp
```
- 提交storm topology

```
bin/storm jar examples/storm-starter/storm-starter-topologies-0.9.3.jar storm.starter.RollingTopWords  production-topology remote

Topology_name        Status     Num_tasks  Num_workers  Uptime_secs
-------------------------------------------------------------------
production-topology  ACTIVE     15         1            18
```

### 搭建storm本地开发环境
搭建storm说明: https://github.com/apache/storm/tree/master/examples/storm-starter
- 下载storm命令 http://storm.apache.org/downloads.html
- 下载storm代码 http://mirror.bit.edu.cn/apache/storm/apache-storm-1.2.2/apache-storm-1.2.2.tar.gz
- 按照搭建storm说明，在storm-1.2.2根目录执行      
```    
mvn clean install -DskipTests=true
```     
执行完后，example/storm-starter/target里jar已经生成，运行该topology (7.9 执行成功)
```
storm jar target/storm-starter-*.jar org.apache.storm.starter.RollingTopWords production-topology
```

但是这个运行不了，/data0/stormcode/apache-storm-1.2.2/bin/storm jar target/storm-kafka-client-examples-1.2.2.jar org.apache.storm.kafka.trident.TridentKafkaClientWordCountNamedTopics 10.71.26.31:9092


### 另一个storm程序单词统计, 运行过程
- 下载storm http://storm.apache.org/downloads.html
- 下载程序源码包 https://codeload.github.com/storm-book/examples-ch02-getting_started/legacy.zip/master
- 解压之后, 需要确保pom.xml中所依赖的storm版本，跟命令行中storm的版本一致，且还要看src/main/java/目录下源代码中引用的storm包，是该版本storm中所有的。http://mvnrepository.com/search?q=storm     

```     
pom.xml
        <dependency>
            <groupId>org.apache.storm</groupId>
            <artifactId>storm-core</artifactId>
            <version>1.2.2</version>
            <scope>provided</scope>
       </dependency>
       
       
src/main/java/TopologyMain.java

import org.apache.storm.Config;
import org.apache.storm.LocalCluster;
import org.apache.storm.topology.TopologyBuilder;
import org.apache.storm.tuple.Fields;
   
   
```

- 部署到storm集群中, storm jar target/Getting-Started-0.0.1-SNAPSHOT.jar TopologyMain src/main/resources/words.txt 

- 编译  mvn compile
- 运行  mvn exec:java -Dexec.mainClass="TopologyMain" -Dexec.args="src/main/resources/words.txt"

