### **配置JAVA_HOME**

>1. 查看jdk的安装地址（拿到的是一个快捷方式地址）**当然有的就是安装地址。跳过2、3即可**

```sh
which java 
/usr/local/btjdk/jdk8/bin/java  #输出内容 
```

>2. 通过命令1输出内容，查看jdk的安装地址（拿到的还是一个快捷方式地址）

```sh
ls -lrt /usr/bin/java 
/usr/bin/java -> /etc/alternatives/java
```

>3. 通过命令2输出内容，拿到jdk的实际地址

```sh
ls -lrt /etc/alternatives/java 
/etc/alternatives/java -> /usr/lib/jvm/java-1.8.0/jre/bin/java  #输出内容
```

>4.进入配置文件/etc/profile(配置环境变量java_home的配置文件)

```sh
vim /etc/profile
```

>5.在配置文件加上下面配置：

```sh
export JAVA_HOME=/usr/lib/jvm/java-1.8.0  #
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```

>6.将环境变量编译生效

```sh
source /etc/profile
```

>7.测试是否配置成功

```sh
javac
```