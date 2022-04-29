# MongoDB分片集群部署



- 集群中所有机器彼此网络互联畅通
- 使用hostname， 不用IP
- ```In production environments, sharded clusters should employ at minimum [x.509](https://www.mongodb.com/docs/manual/core/security-x.509/) security for internal authentication and client access.```

## 架构图

![img](D:\WorkSpace\technologyNotes\pictures\MongoDB分片集群部署\1620.png)

## Create the Config Server Replica Set

- 生产环境中，一个`config server replica set `至少要包含3个
- 

## Create the Shard Replica Sets



## Start a `mongos` for the Sharded Cluster

