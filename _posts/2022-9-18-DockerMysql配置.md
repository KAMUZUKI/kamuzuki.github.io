## DockerMysql配置

#### 1、编码配置

<mark>此方法基于容器卷配置mysql</mark>

> 查看数据卷位置

```shell
docker inspect mysql
```

```shell
-v /mu/mysql/log:/var/log/mysql 
-v /mu/mysql/data:/var/lib/mysql 
-v /mu/mysql/conf:/etc/mysql/conf.d
```

> 新建my.cnf

```shell
cd /mu/mysql/conf
vim my.cnf
```

> 添加编码配置

```cnf
[mysqld]
# 服务端使用的字符集默认为utf8mb4
character-set-server=utf8mb4
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8mb4
[client]
# 设置mysql客户端连接服务端时默认使用的端口
port=3306
default-character-set=utf8mb4
```

> 重启Mysql容器

```shell
docker restart [container_id]
```

> 进入容器查看编码

```shell
show variables like'%char%';
```

```txt
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8mb4                    |
| character_set_connection | utf8mb4                    |
| character_set_database   | utf8mb4                    |
| character_set_filesystem | binary                     |
| character_set_results    | utf8mb4                    |
| character_set_server     | utf8mb4                    |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)
```

<mark>如上显示即为修改成功</mark>
