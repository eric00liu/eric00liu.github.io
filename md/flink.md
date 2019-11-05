---
title: flink搭建
date: 2018/7/13 14:19:43
tags:
- flink
- 异常检测
---

搭建起来比较简单，只需要三个配置文件，使用kubectl create -f 就可以了
通过jobmanager-service的cluster-ip:8081，即可访问flink UI的页面

jobmanager-service.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: flink-jobmanager
spec:
  ports:
  - name: rpc
    port: 6123
  - name: blob
    port: 6124
  - name: query
    port: 6125
  - name: ui
    port: 8081
  selector:
    app: flink
    component: jobmanager
    
```



jobmanager-deployment.yaml
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: flink-jobmanager
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: flink
        component: jobmanager
    spec:
      containers:
      - name: jobmanager
        image: flink:latest
        args:
        - jobmanager
        ports:
        - containerPort: 6123
          name: rpc
        - containerPort: 6124
          name: blob
        - containerPort: 6125
          name: query
        - containerPort: 8081
          name: ui
        env:
        - name: JOB_MANAGER_RPC_ADDRESS  
          value: # 通过kubectl get services 查看jobmanager-service的cluster-ip 
```          
          
taskmanager-deployment.yaml
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: flink-taskmanager
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: flink
        component: taskmanager
    spec:
      containers:
      - name: taskmanager
        image: flink:latest
        args:
        - taskmanager
        ports:
        - containerPort: 6121
          name: data
        - containerPort: 6122
          name: rpc
        - containerPort: 6125
          name: query
        env:
        - name: JOB_MANAGER_RPC_ADDRESS    
          value: # 通过kubectl get services 查看jobmanager-service的cluster-ip  
          
```

## flink 开发教程
一，概述  http://doc.flink-china.org/1.2.0/concepts/programming-model.html
二、滑动窗口 https://flink.apache.org/news/2015/12/04/Introducing-windows.html
三、并行运算和扩容 https://blog.csdn.net/lmalds/article/details/73457767
四、重要概念 http://vinoyang.com/2016/05/02/flink-concepts/
五、stream介绍 https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/operators/index.html "Split"将stream分流到多个stream上

六、split DataStream 使用方法 /Users/liu/code/flink/flink-streaming-java/src/test/java/org/apache/flink/streaming/runtime/operators/StreamOperatorChainingTest.java  https://ci.apache.org/projects/flink/flink-docs-release-1.5/dev/stream/operators/   /Users/liu/code/flink/flink-tests/src/test/java/org/apache/flink/test/streaming/runtime/DirectedOutputITCase.java

## 运行一段时间后，任务异常中断
- taskmanager日志里显示，是链接resourceManager失败导致的         

```
1. Close ResourceManager connection 85654ea551c6c887217cb9ed765e3977.
2. 重连resourceManager
3. Could not resolve ResourceManager address akka.tcp://flink@10.110.22.162:6123/user/resourcemanage
r, retrying in 10000 ms: Ask timed out on [ActorSelection[Anchor(akka.tcp://flink@10.110.22.162:6123/), Path(/user/resourcemanager)]] after [10000 ms]. Sender[null] sent message of type "akka
.actor.Identify"..
4. 一直重试失败后，org.apache.flink.runtime.taskexecutor.exceptions.RegistrationTimeoutException: Could not register at the ResourceManager within the specified maximum registration duration 300000 ms. This indicates a problem with this instance. Terminating now.

```


jobmanager日志里，trashManager 链接resourcemanager时 heartbeat超时        

```     

org.apache.flink.runtime.resourcemanager.StandaloneResourceManager  - The heartbeat of TaskManager with id bba3a47a1aec63436571f0b3657c8515 timed out.
```
timeout之后，又有注册到resourcemanager.slotmanager上了，然后任务RESTARTING    
```
2018-08-07 05:43:13,867 INFO  org.apache.flink.runtime.resourcemanager.StandaloneResourceManager  - Closing TaskExecutor connection 2689b4542574aeffb6c1568a130908c2 because: The heartbeat of Ta
skManager with id 2689b4542574aeffb6c1568a130908c2  timed out.
2018-08-07 05:43:13,867 INFO  org.apache.flink.runtime.resourcemanager.slotmanager.SlotManager  - Unregister TaskManager 59c8e7cbbbbaaff3569e663f6e03da05 from the SlotManager.
2018-08-07 05:43:17,431 INFO  org.apache.flink.runtime.resourcemanager.slotmanager.SlotManager  - Registering TaskManager 2689b4542574aeffb6c1568a130908c2 under ca97f0a26b6d08cdb8522855c124d21c
 at the SlotManager.
 2018-08-07 05:43:18,538 INFO  org.apache.flink.runtime.executiongraph.ExecutionGraph        - Job Monitor run (fe766311552623989de02fda30f32127) switched from state RESTARTING to CREATED.
2018-08-07 05:43:18,538 INFO  org.apache.flink.runtime.executiongraph.ExecutionGraph        - Job Monitor run (fe766311552623989de02fda30f32127) switched from state CREATED to RUNNING.
```









无法再通过重启来恢复了，因为重启策略阻止了再重启

```
2018-08-08 12:09:28,239 INFO  org.apache.flink.runtime.executiongraph.ExecutionGraph        - Could not restart the job Monitor run (2df26ee10d34f33c88b852c9e2c3fff2) because the restart strate
gy prevented it.
```

