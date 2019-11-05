---
title: influxdb使用
date: 2018/7/6 17:32:22
tags:
- storm
- 异常检测
- nimubs
- k8s
- docker
---
### 通过bin文件直接运行
下载：https://dl.influxdata.com/influxdb/releases/influxdb-1.5.4-static_linux_amd64.tar.gz
解压之后，./influxd -config influxdb.conf 即可运行


### 通过docker运行
```
docker run -p 8086:8086 -v /data0/influxdb_dockerdata:/var/lib/influxdb \
  --restart=always --name influxdb \
  -e INFLUXDB_DATA_INDEX_VERSION=tsi1 \
  -e GOGC=10 \
  -e INFLUXDB_DATA_MAX-SERIES-PER-DATABASE=0 \
  -e INFLUXDB_DATA_MAX-VALUES-PER-TAG=0 \
  influxdb:latest
```

