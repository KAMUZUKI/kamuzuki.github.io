## Docker 安装

#### 1、前提说明

> CentOS Docker  安装

>前提条件

目前，CentOS 仅发行版本中的内核支持 Docker。Docker 运行在CentOS 7 (64-bit)上，

要求系统为64位、Linux系统内核版本为 3.8以上，这里选用Centos7.x

> 查看自己的内核

uname命令用于打印当前系统相关信息（内核版本号、硬件架构、主机名称和操作系统类型等）。

```sh
cat /etc/redhat- relase

uname - r
```

#### 2、安装步骤

https://docs.docker.com/engine/install/centos/

1.确定你是CentOS7及以上版本

· cat /etc/redhat-release

2.卸载旧版本

https://docs.docker.com/engine/install/centos/

3.CentOS7能上外网

```sh
yum -y install gcc

yum -y install gcc-c++
```

4.安装需要的软件包

```sh
yum install -y yum-utils
```

5.设置stable镜像仓库

```sh
# 官方使用
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

#国内推荐
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

```

6.更新yum软件包索引

```sh
yum makecache fast
```

7.安装DOCKER CE

```sh
yum -y install docker-ce docker-ce-cli containerd.io
```

8.启动docker

```sh
systemctl start docker
```

9.测试

```sh
docker version
docker run hello-world
```

10.卸载

```sh
systemctl stop docker 
yum remove docker-ce docker-ce-cli containerd.io
rm -rf /var/lib/docker
rm -rf /var/lib/containerd
```

#### 3、阿里云镜像加速

> 是什么

https://promotion.aliyun.com/ntms/act/kubernetes.html

阿里云控制台 获取 容器镜像服务

```sh
mkdir -p /etc/docker

#使用自己的镜像加速链接
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://pjzmyyya.mirror.aliyuncs.com"]
}
EOF
```

```sh
#重启服务器
systemctl daemon-reload
systemctl restart docker
```

