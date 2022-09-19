# 【Spring Cloud Ribbon】

## 1.Ribbon 概述
Spring Cloud Ribbon 是一个基于 HTTP 和 TCP 的**客户端`负载均衡`工具**，它基于 **Netflix**
**Ribbon** 实现。通过 Spring Cloud 的封装，可以让我们轻松地将面向服务的 REST 模版请求
自动转换成客户端负载均衡的服务调用。 轮询 hash 权重 ...
简单的说 Ribbon 就是 netfix 公司的一个开源项目，主要功能是提供**客户端负载均衡算法和**
**服务调用**。Ribbon 客户端组件提供了一套完善的配置项，比如连接超时，重试等。
在 Spring Cloud 构建的微服务系统中， Ribbon 作为服务**消费者**的负载均衡器，有两种使
用方式，一种是和 **`RestTemplate`** 相结合，另一种是和 OpenFeign 相结合。OpenFeign 已经
默认集成了 Ribbon,关于 OpenFeign 的内容将会在下一章进行详细讲解。Ribbon 有很多子
模块，但很多模块没有用于生产环境!

https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#spring-integration

```java
    /**
     * 测试发送 get 请求
     */
    @Test
    void testGet() {
        RestTemplate restTemplate = new RestTemplate();
        String url = "http://localhost:8080/testGet?name=cxs";
        ResponseEntity<String> result = restTemplate.getForEntity(url, String.class);
        System.out.println(result.getStatusCodeValue());
    }
    /**
     * 测试发送 post 表单参数
     */
    @Test
    void testPost() {
        RestTemplate restTemplate = new RestTemplate();
        String url = "http://localhost:8080/testPost";
        LinkedMultiValueMap<String, String> map = new LinkedMultiValueMap<>();
        map.add("name", "cxs");
        map.add("age", "18");
        ResponseEntity<String> result = restTemplate.postForEntity(url, map, String.class);
        System.out.println(result.getStatusCodeValue());
    }
    /**
     * 测试发送 post JSON 参数
     */
    @Test
    void testPost2() {
        RestTemplate restTemplate = new RestTemplate();
        String url = "http://localhost:8080/testPost2";
        User user = new User();
        user.setName("cxs");
        user.setAge(18);
        user.setHobby("编码");
        ResponseEntity<String> result = restTemplate.postForEntity(url, user, String.class);
        System.out.println(result.getStatusCodeValue());
    }
```

## 2.负载均衡

负载均衡，英文名称为 `Load Balance`（LB）http:// lb://（负载均衡协议） ，其含义
就是指将负载（工作任务）进行平衡、分摊到多个操作单元上进行运行，例如 Web 服务器、
企业核心应用服务器和其它主要任务服务器等，从而协同完成工作任务。
负载均衡构建在原有网络结构之上，它提供了一种透明且廉价有效的方法扩展服务器和网络设
备的带宽、加强网络数据处理能力、增加吞吐量、提高网络的可用性和灵活性。

#### 2.1 服务器的负载均衡
> Nginx，F5

![image-20220628203807100](assets/image-20220628203807100.png)

## 3.Ribbon 快速入门
#### 3.1 本次调用设计图

![image-20220628204009827](assets/image-20220628204009827.png)

#### 3.2 项目搭建
**consumer 和 provider-1 和 provider-2 都是 eureka-client**

**注意这三个依赖是 eureka-client**

> 注意 provider-1 和 provider-2 的 spring.application.name=provider
>
> **都是 provider ，如果开始写错更改后，eureka 显示注册的名字未改变或没有找到实例**
>
> **需要重新编译项目重启**

**注意启动类的注解和配置文件的端口以及服务名称**

#### 3.3 创建 provider-1 和 provider-2

![image-20220628204802393](assets/image-20220628204802393.png)

#### 3.4 编写 provider-1 和 provider-2

![image-20220628204818856](assets/image-20220628204818856.png)

![image-20220628204827148](assets/image-20220628204827148.png)

#### 3.5 创建 consumer

![image-20220628204839931](assets/image-20220628204839931.png)

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
    <version>2.2.9.RELEASE</version>
</dependency>
```

#### 3.6 编写 consumer 的启动类

```java
/**
 * @author MUZUKI
 */
@SpringBootApplication
@EnableEurekaClient
public class ConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }

    /**
     * 这个RestTemplate 已经变了
     * LoadBalance 他就会被ribbon来操作
     * @return
     */
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

#### 3.7 编写 consumer 的 TestController

**不使用 Ribbon的TestController**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.client.discovery.DiscoveryClient;
import org.springframework.util.ObjectUtils;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;
import java.util.List;
import java.util.Random;

/**
 * @Author: 北京动力节点
 */

@RestController
public class TestController {
    @Autowired
    private RestTemplate restTemplate;
    @Autowired
    private DiscoveryClient discoveryClient;
    static Random random = new Random();
    @RequestMapping("/testBalance")
    public String testBalance(String serviceId) {
        // 获取服务列表
        List<ServiceInstance> instances = discoveryClient.getInstances(serviceId);
        if (ObjectUtils.isEmpty(instances)) {
            return "服务列表为空";
        }
        // 如果服务列表不为空，先自己做一个负载均衡
        ServiceInstance serviceInstance = loadBalance(instances);
        String host = serviceInstance.getHost();
        int port = serviceInstance.getPort();
        String url = "http://" + host + ":" + port + "/info";
        System.out.println("本次我调用的是" + url);
        String forObject = restTemplate.getForObject(url, String.class);
        System.out.println(forObject);
        return forObject;
    }
    private ServiceInstance loadBalance(List<ServiceInstance> instances) {
        // 拼接 url 去调用 ip:port 先自己实现不用 ribbon
        ServiceInstance serviceInstance =
                instances.get(random.nextInt(instances.size()));
        return serviceInstance;
    }
}
```

#### 3.8 启动测试

首选确保都注册上去了

![image-20220628205455201](assets/image-20220628205455201.png)

然后访问调用
http://localhost:8083/testBalance?serviceId=provider

#### 3.9 使用 Ribbon 改造
只需要对 consumer 改造即可，改造启动类
**改造 controller**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

/**
 * @author MUZUKI
 */
@SpringBootApplication
@EnableEurekaClient
public class ConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }

    /**
     * 这个RestTemplate 已经变了
     * LoadBalance 他就会被ribbon来操作
     * @return
     */
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

/**
 * @author MUZUKI
 * 测试 ribbon 的负载均衡
 * @param serviceName
 * @return
 */
@RestController
public class ConsumerController {
        @Autowired
        private RestTemplate restTemplate;

        @GetMapping("testRibbon")
        public String testRibbon(String serviceName) {
            return restTemplate.getForObject("http://" + serviceName + "/hello",String.class);
        }
}
```

#### 3.10 改造后测试效果
访问 http://localhost:8083/testRibbon?serviceId=provider

## 4.Ribbon 源码分析

#### 4.1 Ribbon 要做什么事情？
**先通过 "http://" + serviceId + "/info" 我们思考 ribbon 在真正调用之前需要做什么？**
**restTemplate.getForObject(“http://provider/info”, String.class);**
想要把上面这个请求执行成功，我们需要以下几步

1. 拦截该请求；

2. 获取该请求的 URL 地址:http://provider/info

3. 截取 URL 地址中的 provider

4. 从服务列表中找到 key 为 provider 的服务实例的集合(服务发现)

5. 根据负载均衡算法选出一个符合的实例

6. 拿到该实例的 host 和 port，重构原来 URL 中的 provider

7. 真正的发送 restTemplate.getForObject(“http://ip:port/info”，String.class)

#### 4.2 Ribbon 负载均衡的测试
新增 controller

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.client.loadbalancer.LoadBalancerClient;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author MUZUKI
 */
@RestController
public class TestChoose {
    @Autowired
    private LoadBalancerClient loadBalancerClient;

    @RequestMapping("/testChoose")
    public String testChoose(String serviceId) {
        ServiceInstance choose = loadBalancerClient.choose(serviceId);
        System.out.println(choose.getHost() + ":" + choose.getPort());
        return choose.toString();
    }
}

```

访问：http://localhost:8083/testChoose?serviceId=provider

#### 4.3 从 choose 方法入手，查看 Ribbon 负载均衡的源码

![image-20220628210855117](assets/image-20220628210855117.png)



**走进 getServer()方法**

![image-20220628210909474](assets/image-20220628210909474.png)

**在 chooseServer()里面得到 rule 是哪个对象**

![image-20220628210920483](assets/image-20220628210920483.png)

**发现当前的 rule 是 ZoneAvoidanceRule 对象，而他只有一个父类 PredicateBasedRule**

![image-20220628210933608](assets/image-20220628210933608.png)

**最终进入 PredicateBasedRule 类的 choose()方法**

![image-20220628210947267](assets/image-20220628210947267.png)

![image-20220628210951926](assets/image-20220628210951926.png)

**com.netflix.loadbalancer.AbstractServerPredicate#incrementAndGetModulo**

![image-20220628211003896](assets/image-20220628211003896.png)

#### 4.4 负载均衡之前的服务列表是从何而来呢？

Ribbon 里面有没有服务列表？

Ribbon 只做负载均衡和远程调用

服务列表从哪来？ 从 eureka 来

Ribbon 有一个核心接口 ILoadBalance（承上(eureka)启下（Rule））

我们发现在负载均衡之前，服务列表已经有数据了

![image-20220628211025663](assets/image-20220628211025663.png)

**重点接口 ILoadBalancer**

![image-20220628211039872](assets/image-20220628211039872.png)

**重点接口 ILoadBalancer**

![image-20220628211051479](assets/image-20220628211051479.png)

**Ribbon 没有服务发现的功能，但是 eureka 有，所以 ribbon 和 eureka 完美结合，我们继续**
**干源码学习**

![image-20220628211102194](assets/image-20220628211102194.png)

**首先关注这两个集合，就是存放从 eureka 服务端拉取的服务列表然后缓存到本地**

![image-20220628211112289](assets/image-20220628211112289.png)

**我们去看 DynamicServerListLoadBalancer 类如何获取服务列表，然后放在 ribbon 的缓**
**存里面**

![image-20220628211137658](assets/image-20220628211137658.png)

**ServerList<T extends Server> 实现类（DiscoveryEnabledNIWSServerList)**

![image-20220628211148875](assets/image-20220628211148875.png)

**再回到 BaseLoadBalancer 中真正的存放服务列表**

![image-20220628211211340](assets/image-20220628211211340.png)

![image-20220628211215098](assets/image-20220628211215098.png)

**最后我们得知，只有在初始化 DynamicServerListLoadBalancer 类时，去做了服务拉取和**
**缓存**
**也就是说并不是服务一启动就拉取了服务列表缓存起来，流程图如下:**

![image-20220628211227985](assets/image-20220628211227985.png)

#### 4.5 Ribbon 把 serverList 缓存起来，脏读怎么处理？（选学）
根据上面缓存服务列表我们得知，ribbon 的每个客户端都会从 eureka-server 中把服务列
表缓存起来
主要的类是 BaseLoadBalancer，那么有新的服务上线或者下线，这么保证缓存及时同步呢

![image-20220628211242078](assets/image-20220628211242078.png)

**Ribbon 中使用了一个 PING 机制**
**从 eureka 中拿到服务列表，缓存到本地，ribbon 搞了个定时任务，隔一段时间就去循环 ping**
**一下每个服务节点是否存活**

![image-20220628211300199](assets/image-20220628211300199.png)

**我们查看 IPing 这个接口**

![image-20220628211309992](assets/image-20220628211309992.png)

**我们就想看 NIWSDiscoveryPing**

![image-20220628211318290](assets/image-20220628211318290.png)

**跟着 isAlive 一直往上找，看哪里去修改本地缓存列表**

![image-20220628211327935](assets/image-20220628211327935.png)

![image-20220628211330982](assets/image-20220628211330982.png)

**查看 notifyServerStatusChangeListener 发现只是一个空壳的接口，并没有对缓存的服务**
**节点做出是实际操作，那么到底在哪里修改了缓存列表的值呢？**
**我们发现在 ribbon 的配置类中 RibbonClientConfiguration 有一个更新服务列表的方法**

![image-20220628211353542](assets/image-20220628211353542.png)

![image-20220628211357215](assets/image-20220628211357215.png)

**定时任务在哪里开始执行的呢？我们查找 doUpdate()方法**

![image-20220628211407997](assets/image-20220628211407997.png)



**解决脏读机制的总结：**

1. **Ping**
2. **更新机制**
**都是为了解决脏读的现象而生的**
**测试发现：更新机制和 ping 有个重回，而且在 ping 的时候不能运行更新机制，在更新的时**
**候不能运行 ping 机制，导致我们很难测到 ping 失败的现象！**
**Ping 机制做不了事情**

#### 4.6 Ribbon 负载均衡的实现和几种算法【重点】

**在 ribbon 中有一个核心的负载均衡算法接口 IRule**

![image-20220628211459997](assets/image-20220628211459997.png)



**1.RoundRobinRule--轮询  请求次数 % 机器数量**

**2.RandomRule--随机**

**3.权重**

**4. iphash**

**3.AvailabilityFilteringRule --会先过滤掉由于多次访问故障处于断路器跳闸状态的服**
**务，还有并发的连接数量超过阈值的服务，然后对于剩余的服务列表按照轮询的策略进行访问**

**4.WeightedResponseTimeRule--根据平均响应时间计算所有服务的权重，响应时间越快服**
**务权重越大被选中的概率越大。刚启动时如果同统计信息不足，则使用轮询的策略，等统计信**
**息足够会切换到自身规则**

**5.RetryRule-- 先按照轮询的策略获取服务，如果获取服务失败则在指定的时间内会进行重**
**试，获取可用的服务**

**6.BestAvailableRule --会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后**
**选择一个并发量小的服务**

**7.ZoneAvoidanceRule -- 默认规则，复合判断 Server 所在区域的性能和 Server 的可用**
**行选择服务器。**



> **Ribbon 默认使用哪一个负载均衡算法：**

**ZoneAvoidanceRule ：区间内亲和轮询的算法！通过一个 key 来区分**

**负载均衡算法：随机 轮训 权重 iphash （响应时间最短算法，区域内亲和（轮训）算法）**

![image-20220628211520063](assets/image-20220628211520063.png)

## 5.如何修改默认的负载均衡算法
#### 5.1 修改 yml 配置文件（指定某一个服务使用什么算法）

```yml
provider: # 提供者的服务名称 , 那么访问该服务的时候就会按照自定义的负载均衡算法
	ribbon:
		NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
# 几种算法的全限定类名
```

#### 5.2 测试调用该服务（这里使用随机规则）

#### 5.3 配置此消费者调用任何服务都用某种算法

```java
@Bean
public IRule myRule() {
    // 指定调用所有的服务都用此算法
    return new RandomRule();
}
```

## 6.Ribbon 的配置文件和常用配置
**Ribbon 有很多默认的配置，查看 DefaultClientConfigImpl**

![image-20220628211634092](assets/image-20220628211634092.png)

```yml
ribbon: # 全局的设置
	eager-load:
		enabled: false # ribbon 一启动不会主动去拉取服务列表，当实际使用时才去拉取 是否立即加载
http:
	client:
		enabled: false # 在 ribbon 最后要发起 Http 的调用调用，我们认为是RestTemplate 完成的，其实最后是 HttpURLConnection 来完成的，这里面设置为 true ，可以把 HttpUrlConnection->HttpClient
	okhttp:
		enabled: false #HttpUrlConnection 来完成的，这里面设置为 true ，可以把 HttpUrlConnection->OkHttpClient( 也是发 http 请求的，它在移动端的开发用的多 )
provider: # 提供者的服务名称 , 那么访问该服务的时候就会按照自定义的负载均衡算法
	ribbon:
		NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
# 修改默认负载均衡算法，几种算法的全限定类名
# NFLoadBalancerClassName: #loadBalance 策略
# NFLoadBalancerPingClassName: #ping 机制策略
# NIWSServerListClassName: # 服务列表策略
# NIWSServerListFilterClassName: # 服务列表过滤策略ZonePreferenceServerListFilter 默认是优先过滤非一个区的服务列表
```

## 7.Ribbon 总结（后面的代码中 不会出现 ribbon）
**Ribbon 是客户端实现负载均衡的远程调用组件，用法简单**
**Ribbon 源码核心：**
**ILoadBalancer 接口：起到承上启下的作用**

1. **承上：从 eureka 拉取服务列表**

2. **启下：使用 IRule 算法实现客户端调用的负载均衡**

  

  - 设计思想：每一个服务提供者都有自己的 ILoadBalancer
    userService---》客户端有自己的 ILoadBalancer
    TeacherService---》客户端有自己的 ILoadBalancer

  - 在客户端里面就是 Map<String,ILoadBalancer> iLoadBalancers
    Map<String,ILoadBalancer> iLoadBalancers 消费者端
    服务提供者的名称 value （服务列表 算法规则 ）

    

  > 如何实现负载均衡的呢？

  iloadBalancer loadbalance = iloadBalancers.get(“user-service”)
  List<Server> servers = Loadbalance.getReachableServers();//缓存起来
  Server server = loadbalance .chooseServer(key) //key 是区 id，--》IRule 算法
  chooseServer 下面有一个 IRule 算法
  IRule 下面有很多实现的负载均衡算法

  你就可以使用 eureka+ribbon 做分布式项目



参考： www.bjpowernode.com 动力节点

