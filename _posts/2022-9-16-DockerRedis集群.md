## 一、Docker Redis集群

#### 1、新建6个docker容器redis实例

```sh
docker run -d --name redis-node-1 --net host --privileged=true -v /data/redis/share/redis-node-1:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6381

docker run -d --name redis-node-2 --net host --privileged=true -v /data/redis/share/redis-node-2:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6382

docker run -d --name redis-node-3 --net host --privileged=true -v /data/redis/share/redis-node-3:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6383

docker run -d --name redis-node-4 --net host --privileged=true -v /data/redis/share/redis-node-4:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6384

docker run -d --name redis-node-5 --net host --privileged=true -v /data/redis/share/redis-node-5:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6385

docker run -d --name redis-node-6 --net host --privileged=true -v /data/redis/share/redis-node-6:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6386
```

> 命令分步解释

| docker run                              | 创建并运行docker容器实例      |
| --------------------------------------- | -------------------- |
| --name redis-node-6                     | 容器名字                 |
| --net host                              | 使用宿主机的IP和端口，默认       |
| --privileged=true                       | 获取宿主机root用户权限        |
| -v /data/redis/share/redis-node-6:/data | 容器卷，宿主机地址:docker内部地址 |
| redis:6.0.8                             | redis镜像和版本号          |
| --cluster-enabled yes                   | 开启redis集群            |
| --appendonly yes                        | 开启持久化                |
| --port 6386                             | redis端口号             |

#### 2、进入容器redis-node-1并为6台机器构建集群关系

```sh
docker exec -it redis-node-1 /bin/bash
```

```sh
# 注意，进入docker容器后才能执行一下命令，且注意自己的真实IP地址
redis-cli --cluster create 127.0.0.1:6381 127.0.0.1:6382 127.0.0.1:6383 127.0.0.1:6384 127.0.0.1:6385 127.0.0.1:6386 --cluster-replicas 1
```

**--cluster-replicas 1 表示为每个master创建一个slave节点**

#### 3、链接进入6381作为切入点，查看集群状态

```sh
redis-cli -p 6381
cluster info
cluster nodes
```

## 二、主从容错切换迁移案例

#### 1、数据读写存储

> 启动6机构成的集群并通过exec进入

```
redis-cli -p 6381
```

> 对6381新增两个key

**经过CRC16 对于不属于6381的key将不会存储**

```
set k1 v1
set k2 v2
set k3 v3
```

> 防止路由失效加参数 -c 并新增两个key

**key会按照计算后的值被存储至不同的redis中**

```sh
redis-cli -p 6381 -c
```

>  查看集群信息

```sh
redis-cli --cluster check 127.0.0.1:6381
```

#### 2、容错切换迁移

· 主6381和从机切换，先停止主机6381

· 6381主机停了，对应的真实从机上位

· 6381作为1号主机分配的从机以实际情况为准，具体是几号机器就是几号

· 再次查看集群信息

```
6381宕机了，6385上位成为了新的master。
备注：本次脑图笔记6381为主下面挂从6385。
每次案例下面挂的从机以实际情况为准，具体是几号机器就是几号
```

## 三、扩容与缩容

> 添加节点机器

```shell
#将新增的6387作为master节点加入集群
#redis-cli --cluster add-node 自己实际IP地址:6387 自己实际IP地址:6381
#6387 就是将要作为master新增节点
#6381 就是原来集群节点里面的领路人，相当于6387拜拜6381的码头从而找到组织加入集群
redis-cli --cluster add-node 127.0.0.1:6387 127.0.0.1:6381
```

> 重新分派槽号

```shell
#命令:redis-cli --cluster reshard IP地址:端口号
redis-cli --cluster reshard 127.0.0.1:6381
```

> 删除节点机器

```shell
#命令：redis-cli --cluster del-node ip:从机端口 从机6388节点ID
redis-cli --cluster del-node 127.0.0.1:6388 5d149074b7e57b802287d1797a874ed7a1a284a8
```
