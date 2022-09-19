# 

## Docker安装mysql

#### 1、docker hub上面查找mysql镜像

```sh
docker search mysql
```

#### 2、从docker hub上(阿里云加速器)拉取mysql镜像到本地标签为5.7

```sh
docker pull mysql:5.7
```

#### 3、使用mysql5.7镜像创建容器(也叫运行镜像)

> **简单版**

- 使用mysql镜像

```shell
docker run -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
docker ps
docker exec -it [IMAGE_ID] /bin/bash
mysql -u root -p
```

- 建库建表插入数据

```sql
create database db01;
use db01;
create table demo(id int,name varchar(20);
select * from demo;
```

- 插入中文数据报错

```shell
SHOW VARIABLES LIKE 'character%'
```

更改字符集即可

> **实战版**

- 新建mysql容器实例

```shell
docker run -d -p 3306:3306 --privileged=true -v /mu/mysql/log:/var/log/mysql -v /mu/mysql/data:/var/lib/mysql -v /mu/mysql/conf:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=123456  --name mysql mysql:5.7
```

- 新建my.cnf

```shell
cd /mu/mysql/conf/
vim my.cnf

#添加以下配置
[client]
default_character_set=utf8
[mysqld]
collation_server = utf8_general_ci
character_set_server = utf8

cat my.cnf
```

- 重新启动mysql容器实例再重新进入并查看字符编码

```shell
docker restart msyql
docker exec -it mysql bash
```

- 再新建库新建表再插入中文测试

```sql
create database db01;
use db01;
create table demo(id int,name varchar(20);
select * from demo;
```
