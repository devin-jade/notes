---
layout: post
title: Spring Cloud 基础
categories: [Spring Cloud]
tags: Spring
---

Spring Cloud

身为Java 开发工程师，Spring Cloud 就行JVM一样，是必须要掌握和学习的东西，此系列是《Spring Cloud 微服务实战》的学习笔记。

微服务架构的九大特征：

> 1. 服务组件化
> 2. 按业务组织团队 
> 3. 做产品态度
> 4. 智能端点与哑管道
> 5. 去中心化治理
> 6. 去中心化管理数据
> 7. 基础设施的自动化，自动化部署，测试
> 8. 容错设计
> 9. 演进式设计

对微服务架构支持最完善，使用最广泛的就是Spring Cloud ，因此需要学习之。

Spring Cloud 是基于Spring boot 实现的微服务架构开发工具。为微服务中的配置管理，服务治理，断路器，只能路由，微代理，控制总线，全局锁，决策竞选，分布式会话，集群状态管理等提供了简单的开发方式。

Spring Cloud 全家桶包含很多子项目：

- Spring Cloud Config：配置管理工具，支付git配置内容，使用它可以实现应用配置的外部存储，并支持客户端配置信息刷新，加密解密配置内容等；
- Spring cloud Netflix ： 核心组件，开发套件的集合
  - Eureka:服务治理组件，服务注册中心，服务注册和服务发现机制
  - Hystrix：容错管理组件，断路器模式，熔断
  - Ribbon:客户端的负载均衡
  - Feign：基于Ribbon和Hystrix的声明式调用组件
  - Zuul：网关组件，只能路由，访问过滤
  - Archaius:外部化配置组件
- Spring Cloud Bus ：事件，消息总线，用于传播集群中的状态变化或事件，已触发后续处理，比如动态刷新配置等
- Spring Cloud Cluster:针对Zookeeper，Redis，Hazelcast，Consul的选举算法和通用状态模式的实现。
- Spring Cloud Cloudfoundry:与Pivotal Cloudfoundry的整合支持
- Spring Cloud Consul:服务发现和配置工具
- Spring Cloud Stream ：通过Redis，Rabbit，或者kafka实现的消费微服务，可以通过简单的声明式模型来发送和接收消息。
- Spring Cloud AWS：用于简化整合Amazon Web Service 的组件
- Spring Cloud Security：安全工具包，提供在Zuul代理中对OAuth2客户端请求的中继器。
- Spring Cloud Sleuth:Spring Cloud应用的分布式跟踪实现，可以完美整合Zipkin。
- Spring Cloud ZooKeeper：基于ZooKeeper的服务发现与配置管理组件。
- Spring Cloud Starters:Spring Cloud的基础组件，它是基于SpringBoot风格项目的基础依赖模块。
- Spring Cloud CLI：用于在Groovy中快速创建Spring Cloud应用的SpringBoot CLI插件。 

CAP理论为：一个分布式系统最多只能同时满足一致性（Consistency）、可用性（Availability）和分区容错性（Partition tolerance）这三项中的两项。

- **Consistency(一致性) :** all nodes see the same data at the same time。即更新操作成功并返回客户端后，所有节点在同一时间的数据完全一致，这就是分布式的一致性。

- **Availability（可用性）：** Reads and writes always succeed。服务一直可用，而且是正常响应时间

- **Partition tolerance（分区容错性）：** the system continues to operate despite arbitrary message loss or failure of part of the system。即分布式系统在遇到某节点或网络分区故障的时候，仍然能够对外提供满足一致性和可用性的服务

三个特性不能同时满足，由于分布式系统的特性，P是必须保证的，因此只能在A，C中选择，客户体验越来越重要的今天，A显得优先级越来越高，因此分布式系统必须保证的是AP，C的通过"最终一致性"来保证，但是数据最终数据一致性带来的时间误差必须在客户的可接受范围内。

Eureka 强调AP，Zookeeper 强调CP



## 服务治理

主要包含：

- 服务注册：需要一个注册中心，服务将自己的主机，端口号，版本号，通信协议等一些附加信息告知注册中心，注册中心维护这每个服务的信息清单，注册中心通过心跳检测机制去监控服务的状态，并拥有屏蔽故障服务的能力；
- 服务发现：注册中心需要提供服务寻址能力，支持通过服务名发起请求调用；

Spring Cloud Eureka：AP：包含服务端和客户端组件，java语言开发的。Eureka服务端就是一个服务注册中心，Eureka客户端完成服务的注册和发现；为了更高效的完成服务发现，Eureka客户端会周期性的拉取注册服务信息，缓存到本，周期性刷新；

Eureka 服务端配置：

server.port=1111
eureka.instance.hostname=localhost
eureka.client.register-with-eureka=false  // 禁止自己向自己注册
eureka.client.fetch-registry=false  // 注册中心无需去检索服务
eureka.client.serviceUrl.defaultZone=http://${eureka.instance.host
r.port}/eureka/ 

Eureka 客户端配置：

spring.application.name=hello-service
eureka.client.serviceUrl.defaultZone=http://localhost:1111/eureka/

高可用的注册中心：Eureka Server 的高可用实际上就是将自己作为服务想其他服务注册中心注册自己，形成一组相互注册的服务注册中心，实现服务清单的互相同步，达到高可用的目的。

Eureka-Client：负责服务发现；

服务注册：服务提供者通过发送Rest请求将自己注册到Eureka Server上，同事带上了自身服务的一些元数据信息。Eureka Server 接收到信息之后，将数据存在一个双层的ConcurrentHashMap中，第一层：key 是服务名，第二层：Key 是服务的实例名： Map 内容见下面日志输出

```
DynamicServerListLoadBalancer:{NFLoadBalancer:name=HELLOSERVICE,current list of
Servers=[PC-201602152056:8082,PC-201602152056:8081],Load
balancer stats=Zone stats:
{defaultzone=[Zone:defaultzone; Instance count:2; Active
connections count: 0;
Circuit breaker tripped count: 0; Active connections per
server: 0.0;]
},Server stats:[[Server:PC-201602152056:8082;
Zone:defaultZone; Total Requests:0;
Successive connection failure:0; Total blackout
seconds:0;Last connection made:Thu Jan
01 08:00:00 CST 1970; First connection made: Thu Jan 01
08:00:00 CST 1970; Active
Connections:0; total failure count in last（1000）msecs:0;
average resp time:0.0; 90
percentile resp time:0.0; 95 percentile resp time:0.0; min
resp time:0.0; max resp
time:0.0; stddev resp time:0.0]
,[Server:PC-201602152056:8081; Zone:defaultZone; Total
Requests:0; Successive
connection failure:0; Total blackout seconds:0;Last
connection made:Thu Jan 01 08:00:00
CST 1970; First connection made: Thu Jan 01 08:00:00 CST1970; Active Connections:0;
total failure count in last（1000）msecs:0; average resp
time:0.0; 90 percentile resp
time:0.0; 95 percentile resp time:0.0; min resp time:0.0;
max resp time:0.0; stddev
resp time:0.0]
]}ServerList:org.springframework.cloud.netflix.ribbon.eureka.Domai
List@cc7240
```

服务同步：当一台机器收到服务注册请求的时候，会将该请求转发给其他的注册中心，从而实现服务同步。

服务续约：注册之后，客户端会维护一个心跳持续告诉Eureka Server，i am live，已完成续约。规定时间内，没有接收到续约心跳，Eureka 会剔除该服务，将该实例从服务列表中剔除。

eureka.instance.lease-renewal-interval-in-seconds=30
eureka.instance.lease-expiration-duration-in-seconds=90 

eureka.instance.lease-renewal-interval-in-seconds 参数用于定义服务续约任务的调用间隔时间，默认为30秒。 eureka.instance.leaseexpirationduration-in-seconds参数用于定义服务失效的时间，默认为90秒。 

获取服务： 消费者启动的时候，会发一个rest请求给服务注册中心，获取上面注册的服务清单，为了性能考虑，Eureka Server 会维护一份只读的服务清单来返回给客户端，客户端拉取后会缓存清单，该缓存清单会每隔30S更新一次。若希望修改缓存清单的更新时间，可以通过eureka.client.registry-fetch-interval-seconds=30参数进行修改，该参数默认值为30，单位为秒 。

服务调用：服务消费拿到清单之后，可以根据服务名获取服务实例名和实例元数据信息，客户端根据自己的需要决定调用哪个实例，Ribbon 默认采用轮询的方式实现负载均衡。对于访问实例的选择，Eureka中有Region和Zone的概念，一个Region中可以包含多个Zone，每个服务客户端需要被注册到一个Zone中，所以每个客户端对应一个Region和一个Zone。 在进行服务调用的时候，优先访问同处一个 Zone 中的服务提供方，若访问不到，就访问其他的Zone 。

服务下线：正常关闭时，客户端会发送一个下线Rest请求给Eureka  Server ，Eureka Server会将服务状态置为下线 down，并把事件传播出去。

失效剔除：Eureka Server在启动的时候会创建一个定时任务，默认每隔一段时间（默认为60秒）将当前清单中超时（默认为90秒）没有续约的服务剔除出去。

自我保护：，服务注册到Eureka Server之后，会维护一个心跳连接，告诉Eureka Server自己还活着。 Eureka Server在运行期间，会统计心跳失败的比例在15分钟之内是否低于85%，如果出现低于的情况（在单机调试的时候很容易满足，实际在生产环境上通常是由于网络不稳定导致）,Eureka Server 会将当前的实例注册信息保护起来，让这些实例不会过期，尽可能保护这些注册信息。 但是，在这段保护期间内实例若出现问题，那么客户端很容易拿到实际已经不存在的服务实例，会出现调用失败的情况，所以客户端必须要有容错机制，比如可以使用请求重试、 断路器等机制 。本地调试的时候可以使用eureka.server.enableself-preservation=false参数来关闭保护机制，以确保注册中心可以将不可用的实例正确剔除 。

Eureka 客户端配置主要分为两个方面：

服务注册相关的配置信息：服务注册中心地址，服务获取的间隔时间，可用区域；

服务实例相关的配置信息：服务实例的名称，IP地址，端口号，健康检查；



由于 Eureka 的通信机制是HTTP和REST接口，所以是平台无关的。



## 客户端负载均衡

Spring Cloud Ribbon: 基于HTTP和 TCP的客户端负载均衡工具。

服务端负载均衡：一般原理是负载均衡设备通过心跳检测机制维护一个可用服务端列表，请求到达后，通过某种算法（线性轮询，权重负载，流量负载）等从列表中选取一台服务端地址进行转发。 按照类似下面架构搭建起来的系统。

![](https://ws1.sinaimg.cn/large/9cd40bd3gy1g79et894e2j20gm0bi759.jpg)

	- 硬件负载均衡：F5，稳定，昂贵
	- 软件负载均衡：Nginx ，便宜

客户端负载均衡：就是把服务端负载均衡的可用服务端列表维护在客户端，这个列表来源于注册中心。



### RestTemplate的四中请求：

#### GET请求：

 - getForEntity函数：返回ResponseEntity，包含HTTP 的重要 元素，比如HttpStatus，HttpHeaders，以及泛类型的请求体对象，如果需要获取业务数据，需要再调用getBody()。

   | 函数                                                         | 参数解释                                                     | 例子                                                         |
   | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
   | getForEntity（String url,ClassresponseType,Object...urlVariables） | 其中 url 为请求的地址，<br />responseType 为请求响应体body的包装类型，<br />urlVariables为url中的参数绑定 | getForEntity（"http://USERSERVICE/user? name=1}",User.class,"didi"） |
   | getForEntity（String url,Class responseType,Map urlVariables） | 这里使用了Map类型，所以使用该方法进行参数绑定时需要在占位符中指定Map 中参数的key值 | getForEntity（"http://USERSERVICE/user? name={name}",String.class,params） |
   | getForEntity（URI url,Class responseType）                   | 该方法使用URI对象来<br/>替代之前的url和urlVariables参数来指定访问地址和参数绑定 | RestTemplate restTemplate=new RestTemplate();<br/>UriComponents uriComponents=UriComponentsBuilder<br/>			.fromUriString("http://USER-SERVICE/user? name={name}")<br/>    		.build()<br/>			.expand("dodo")<br/>			.encode();<br/>URI uri=uriComponents.toUri();<br/>ResponseEntity<String> responseEntity=restTemplate.getForEntity(uri,String.class).getBody() |

- getForObject函数： 对于getForEntity的进一步封装，返回的直接就是最需要的业务数据，少了一个从response中获取body的步骤。

  | 函数                                                         | 参数                                                         | 例子                                                         |
  | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | getForObject（String url,ClassresponseType,Object...urlVariables） | 与getForEntity的方法类似，url参数指定访问的地址，responseType参数定义该方法的返回类型，urlVariables<br/>参数为url中占位符对应的参数 |                                                              |
  | getForObject（String url,Class responseType,MapurlVariables） | 在该函数中，使用Map类型的urlVariables替代上面数组形式的urlVariables，因此使用时在url中需要将占位符的名称与Map类型中的key一一<br/>对应设置 |                                                              |
  | getForObject（URI url,Class responseType）                   | 该方法使用 URI 对象来替代之前的url和urlVariables参数使用     | RestTemplate restTemplate=new RestTemplate（）;<br/>User result=restTemplate.getForObject（uri,User.class） |

#### Post请求 ：

- postForEntity函数：同getForEntity函数，返回ResponseEntity<T>  T是请求响应body的类型，需要getBody来获取业务数据；

  | 函数                                                         | 参数                                                         | 例子 |
  | ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
  | postForEntity（String url,Object request,Class responseType,Object...uriVariables） | 这些函数中的参数用法大部分与 getForEntity 一致，比如，第一个重载函数<br/>和第二个重载函数中的 uriVariables 参数都用来对 url 中的参数进行绑定使<br/>用；responseType参数是对请求响应的body内容的类型定义。 这里需要注意的是新<br/>增加的request参数，该参数可以是一个普通对象，也可以是一个HttpEntity对<br/>象。 如果是一个普通对象，而非HttpEntity对象的时候，RestTemplate会将请求<br/>对象转换为一个HttpEntity对象来处理，其中Object就是request的类型，<br/>request内容会被视作完整的body来处理；而如果request是一个HttpEntity对<br/>象，那么就会被当作一个完成的http 请求对象来处理，这个 request 中不仅包含<br/>了 body 的内容，也包含了header的内容 |      |
  | postForEntity（String url,Object request,Class responseType,Map uriVariables） |                                                              |      |
  | postForEntity（URI url,Object request,Class responseType）   |                                                              |      |

  

- postForObject函数：类似getForObject函数，少了一个getBody。

  

  | 函数                                                         | 参数                                                         | 例子 |
  | ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
  | postForObject（String url,Object request,Class responseType,Object...uriVariables） | 函数除了返回的对象类型不同，函数的传入参数均与postForEntity一<br/>致，因此可参考之前postForEntity的说明 |      |
  | postForObject（String url,Object request,Class responseType,Map uriVariables） |                                                              |      |
  | postForObject（URI url,Object request,Class responseType）   |                                                              |      |

- postForLocation函数：该方法实现了以POST请求提交资源，并返回新资源的URI，比如下面的例子 。

  ```java
  User user=new User("didi",40);
  URI responseURI=restTemplate.postForLocation("http://USERSERVICE/user",user);
  ```

  

  | 函数                                                         | 参数                                                         | 例子 |
  | ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
  | postForLocation（String url,Object request,Object...uriVariables） | 由于postForLocation函数会返回新资源的URI，该URI就相当于指定了返回类<br/>型，所以此方法实现的POST请求不需要像postForEntity和postForObject那样指<br/>定responseType。 其他的参数用法相同 |      |
  | postForLocation（String url,Object request,Map uriVariables） |                                                              |      |
  | postForLocation（URI url,Object request）                    |                                                              |      |

#### Put请求：

在RestTemplate中，对PUT请求可以通过put方法进行调用实现，比如：

```java
RestTemplate restTemplate=new RestTemplate();
Long id=10001L;
User user=new User("didi",40)
restTemplate.put（"http://USER-SERVICE/user/{1}",user,id）;
```

三种不同的重载方法：

- put（String url,Object request,Object...urlVariables）

- put（String url,Object request,Map urlVariables）

- put（URI url,Object request）

  put函数为void类型，所以没有返回内容，也就没有其他函数定义的responseType参数，除此之外的其他传入参数定义与用法与postForObject基本一致。 

#### Delete请求：

在RestTemplate中，对DELETE请求可以通过delete方法进行调用实现，比如：

```java
RestTemplate restTemplate=new RestTemplate();
Long id=10001L;
restTemplate.delete("http://USER-SERVICE/user/{1}",id);
```

三种不同的重载方法：

- delete（String url,Object...urlVariables）

- delete（String url,Map urlVariables）

- delete（URI url）

  由于我们在进行REST请求时，通常都将DELETE请求的唯一标识拼接在url中，所以DELETE请求也不需要request的body信息，就如上面的三个函数实现一样，非常简单。 url指定DELETE请求的位置，urlVariables绑定url中的参数即可.

@LoadBalanced 注解标记RestTemplate，接口LoadBalanceClient是客户端负载均衡的核心。

Ribbon 六大核心配置：

- IClientConfig:Ribbon 的客户端配置，默认采用com.netflix.client.config.DefaultClientConfigImpl实现。
- IRule:Ribbon 的负载均衡策略，默认采用com.netflix.loadbalancer.ZoneAvoidanceRule实现，该策略能够在多区域环境下选出最佳区域的实例进行访问。
- IPing:Ribbon的实例检查策略，默认采用com.netflix.loadbalancer.NoOpPing实现，该检查策略是一个特殊的实现，实际上它并不会检查实例是否可用，而是始终返回true，默认认为所有服务实例都是可用的。
- ServerList<Server>：服务实例清单的维护机制，默认采用com.netflix.loadbalancer.ConfigurationBasedServerList实现。
- ServerListFilter<Server>：服务实例清单过滤机制，默认采用org.springframework.cloud.netflix.ribbon.ZonePreferenceServerListFilter实现，该策略能够优先过滤出与请求调用方处于同区域的服务实例。
- ILoadBalancer：负载均衡器，默认采用com.netflix.loadbalancer.ZoneAwareLoadBalancer实现，它具备了区域感知的能力。





- NFLoadBalancerClassName：配置ILoadBalancer接口的实现。
- NFLoadBalancerPingClassName：配置IPing接口的实现。
- NFLoadBalancerRuleClassName：配置IRule接口的实现。
- NIWSServerListClassName：配置ServerList接口的实现。
- NIWSServerListFilterClassName：配置ServerListFilter接口的实现



