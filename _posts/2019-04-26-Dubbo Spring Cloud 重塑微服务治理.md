> 原文链接：[Dubbo Spring Cloud 重塑微服务治理](https://mp.weixin.qq.com/s/K60e1VkAWjwQXftmHw74NQ)，来自于微信公众号：`次灵均阁`

## 摘要

在 Java 微服务生态中，Spring Cloud[^1] 成为了开发人员的首选技术栈，然而随着实践的深入和运用规模的扩大，大家逐渐意识到 Spring Cloud 的局限性。在服务治理方面，相较于 Dubbo[^2] 而言，Spring Cloud 并不成熟。遗憾的是，Dubbo 往往被部分开发者片面地视作服务治理的 RPC 框架，而非微服务基础设施。即使是那些有意将 Spring Cloud 迁移至 Dubbo 的小伙伴，当面对其中迁移和改造的成本时，难免望而却步。庆幸的是，Dubbo 生态体系已发生巨大变化，Dubbo Spring Cloud 作为 Spring Cloud Alibaba[^3] 的最核心组件，完全地拥抱 Spring Cloud 技术栈，不但无缝地整合 Spring Cloud 注册中心，包括 Nacos[^4]、Eureka[^5]、Zookeeper[^6] 以及 Consul[^7]，而且完全地兼容 Spring Cloud Open Feign[^8] 以及 @LoadBalanced RestTemplate，本文将讨论 Dubbo Spring Cloud 对 Spring Cloud 技术栈所带来的革命性变化。

> 注：由于 Spring Cloud 技术栈涵盖的特性众多，因此本文讨论的范围仅限于服务治理部分。



## 简介

Dubbo Spring Cloud 基于 Dubbo Spring Boot 2.7.1[^9] 和 Spring Cloud 2.x 开发，无论开发人员是 Dubbo 用户还是 Spring Cloud 用户，都能轻松地驾驭，并以接近“零”成本的代价使应用向上迁移。Dubbo Spring Cloud 致力于简化 Cloud Native 开发成本，提高研发效能以及提升应用性能等目的。

2019年4月19日，Dubbo Spring Cloud 首个 Preview Release，随同 Spring Cloud Alibaba `0.2.2.RELEASE` 和  `0.9.0.RELEASE` 一同发布[^10]，分别对应 Spring Cloud Finchley[^11] 与 Greenwich[^12] (下文分别简称为 “F” 版 和 “G” 版) 。



## 版本支持

由于 Spring 官方宣布 Spring Cloud Edgware(下文简称为 “E” 版) 将在 2019 年 8 月 1 号后停止维护13，因此，目前 Dubbo Spring Cloud 发布版本并未对 “E” 版提供支持，仅为 “F” 版 和 “G” 版开发，同时也建议和鼓励 Spring Cloud 用户更新至 “F” 版 或 “G” 版。

同时，Dubbo Spring Cloud 基于 Apache Dubbo Spring Boot 2.7.1 开发（最低 Java 版本为 1.8），提供完整的 Dubbo 注解驱动、外部化配置以及 Production-Ready 的特性，详情请参考：https://github.com/apache/incubator-dubbo-spring-boot-project

以下表格将说明 Dubbo Spring Cloud 版本关系映射关系：

| Spring Cloud | Spring Cloud Alibaba | Spring Boot | Dubbo Spring Boot                  |
| ------------ | -------------------- | ----------- | ---------------------------------- |
| Finchley     | `0.2.2.RELEASE`      | `2.0.x`     | `2.7.1`                            |
| Greenwich    | `0.9.0.RELEASE`      | `2.1.x`     | `2.7.1`                            |
| Edgware      | `0.1.2.RELEASE`      | `1.5.x`     | :x: Dubbo Spring Cloud 不支持该版本 |



## 功能特性

由于 Dubbo Spring Cloud 构建在原生的 Spring Cloud 之上，其服务治理方面的能力可认为是 Spring Cloud Plus，不仅完全覆盖 Spring Cloud 原生特性[^13]，而且提供更为稳定和成熟的实现，特性比对如下表所示：

| 功能组件                                             | Spring Cloud                           | Dubbo Spring Cloud                                     |
| ---------------------------------------------------- | -------------------------------------- | ------------------------------------------------------ |
| 分布式配置（Distributed configuration）              | Git、Zookeeper、Consul、JDBC           | Spring Cloud 分布式配置 + Dubbo 配置中心[^14]          |
| 服务注册与发现（Service registration and discovery） | Eureka、Zookeeper、Consul              | Spring Cloud 原生注册中心[^15] + Dubbo 原生注册中心[^16] |
| 负载均衡（Load balancing）                           | Ribbon（随机、轮询等算法）             | Dubbo 内建实现（随机、轮询等算法 + 权重等特性）        |
| 服务熔断（Circuit Breakers）                         | Spring Cloud Hystrix                   | Spring Cloud Hystrix + Alibaba Sentinel[^17] 等        |
| 服务调用（Service-to-service calls）                 | Open Feign、`RestTemplate`             | Spring Cloud 服务调用 + Dubbo `@Reference`             |
| 链路跟踪（Tracing）                                  | Spring Cloud Sleuth[^18] + Zipkin[^19] | Zipkin、opentracing 等                                 |



### 高亮特性

#### 1. Dubbo 使用 Spring Cloud 服务注册与发现

Dubbo Spring Cloud 基于 Spring Cloud Commons 抽象实现 Dubbo 服务注册与发现，应用只需增添外部化配置属性 “`dubbo.registry.address = spring-cloud://localhost`”，就能轻松地桥接到所有原生 Spring Cloud 注册中心，包括：
- Nacos
- Eureka
- Zookeeper
- Consul

> Dubbo Spring Cloud 将在下个版本支持 Spring Cloud 注册中心与 Dubbo 注册中心并存，提供双注册机制，实现无缝迁移。



#### 2. Dubbo 作为 Spring Cloud 服务调用

默认情况，Spring Cloud Open Feign 以及 `@LoadBalanced` `RestTemplate` 作为 Spring Cloud 的两种服务调用方式。Dubbo Spring Cloud 为其提供了第三种选择，即 Dubbo 服务将作为 Spring Cloud 服务调用的同等公民出现，应用可通过 Apache Dubbo 注解 `@Service`和 `@Reference` 暴露和引用 Dubbo 服务，实现服务间多种协议的通讯。同时，也可以利用 Dubbo 泛化接口轻松实现服务网关。



#### 3. Dubbo 服务自省

Dubbo Spring Cloud 引入了全新的服务治理特性 - 服务自省（Service Introspection），其设计目的在于最大化减轻注册中心负载，去 Dubbo 注册元信息中心化。假设一个 Spring Cloud 应用引入 Dubbo Spring Boot Starter，并暴露 N 个 Dubbo 服务，以 [Dubbo Nacos 注册中心](https://github.com/apache/incubator-dubbo/tree/master/dubbo-registry/dubbo-registry-nacos) 为例，当前应用将注册 N+1 个 Nacos 应用，除 Spring Cloud 应用本身之前，其余 N 个应用均来自于 Dubbo 服务，当 N 越大时，注册中心负载越重。因此，Dubbo Spring Cloud 应用对注册中心的负载相当于传统 Dubbo 的 N 分之一，在不增加基础设施投入的前提下，理论上，使其集群规模扩大 N 倍。当然，未来的 Dubbo 也将提供服务自省的能力。



#### 4. Dubbo 迁移 Spring Cloud 服务调用

尽管 Dubbo Spring Cloud 完全地保留了原生 Spring Cloud 服务调用特性，不过 Dubbo 服务治理的能力是 Spring Cloud Open Feign 所不及的，如高性能、高可用以及负载均衡稳定性等方面。因此，建议开发人员将 Spring Cloud Open Feign 或者 `@LoadBalanced` `RestTemplate` 迁移为 Dubbo 服务。考虑到迁移过程并非一蹴而就，因此，Dubbo Spring Cloud 提供了方案，即 `@DubboTransported` 注解。该注解能够帮助服务消费端的 Spring Cloud Open Feign 接口以及 `@LoadBalanced` `RestTemplate` Bean 底层走 Dubbo 调用（可切换 Dubbo 支持的协议），而服务提供方则只需在原有 `@RestController` 类上追加 Dubbo `@Servce` 注解（需要抽取接口）即可，换言之，在不调整 Feign 接口以及 `RestTemplate` URL 的前提下，实现无缝迁移。如果迁移时间充分的话，建议使用 Dubbo 服务重构系统中的原生 Spring Cloud 服务的定义。



## 简单示例

开发 Dubbo Spring Cloud 应用的方法与传统 Dubbo 或 Spring Cloud 应用类似，按照以下步骤就能完整地实现Dubbo 服务提供方和消费方的应用，完整的示例代码请访问一下资源：

- Dubbo 服务提供方应用 - <https://github.com/spring-cloud-incubator/spring-cloud-alibaba/tree/master/spring-cloud-alibaba-examples/spring-cloud-alibaba-dubbo-examples/spring-cloud-dubbo-server-sample>
- Dubbo 服务消费方应用 - <https://github.com/spring-cloud-incubator/spring-cloud-alibaba/tree/master/spring-cloud-alibaba-examples/spring-cloud-alibaba-dubbo-examples/spring-cloud-dubbo-client-sample>



### 定义 Dubbo 服务接口

Dubbo 服务接口是服务提供方与消费方的远程通讯契约，通常由普通的 Java 接口（interface）来声明，如 `EchoService` 接口：

```java
public interface EchoService {

    String echo(String message);
}
```

为了确保契约的一致性，推荐的做法是将 Dubbo 服务接口打包在第二方或者第三方的 artifact（jar）中，如以上接口就存放在 artifact [spring-cloud-dubbo-sample-api](https://github.com/spring-cloud-incubator/spring-cloud-alibaba/tree/master/spring-cloud-alibaba-examples/spring-cloud-alibaba-dubbo-examples/spring-cloud-dubbo-sample-api) 之中。

对于服务提供方而言，不仅通过依赖 artifact 的形式引入 Dubbo 服务接口，而且需要将其实现。对应的服务消费端，同样地需要依赖该 artifact，并以接口调用的方式执行远程方法。接下来进一步讨论怎样实现 Dubbo 服务提供方和消费方。



### 实现 Dubbo 服务提供方

#### 初始化 `spring-cloud-dubbo-server-sample` Maven 工程

首先，创建 `artifactId` 名为 `spring-cloud-dubbo-server-sample` 的 Maven 工程，并在其  `pom.xml` 文件中增添 
Dubbo Spring Cloud 必要的依赖：

```xml
<dependencies>
    <!-- Sample API -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-dubbo-sample-api</artifactId>
        <version>${project.version}</version>
    </dependency>

    <!-- Spring Boot dependencies -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-actuator</artifactId>
    </dependency>

    <!-- Dubbo Spring Cloud Starter -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-dubbo</artifactId>
    </dependency>

    <!-- Spring Cloud Nacos Service Discovery -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
</dependencies>
```

以上依赖 artifact 说明如下：

- `spring-cloud-dubbo-sample-api` : 提供 `EchoService` 接口的 artifact
- `spring-boot-actuator` : Spring Boot Production-Ready artifact，间接引入 `spring-boot` artifact
- `spring-cloud-starter-dubbo` : Dubbo Spring Cloud Starter `artifact`，间接引入 `dubbo-spring-boot-starter` 等 artifact
- `spring-cloud-starter-alibaba-nacos-discovery` : Nacos Spring Cloud 服务注册与发现 `artifact`


值得注意的是，以上 artifact 未指定版本(version)，因此，还需显示地声明 `<dependencyManagement>` :

```xml
<dependencyManagement>
    <dependencies>
        <!-- Spring Cloud Alibaba dependencies -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>0.9.0.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

> 以上完整的 Maven 依赖配置，请参考 `spring-cloud-dubbo-server-sample` [`pom.xml`](spring-cloud-dubbo-server-sample/pom.xml) 文件

完成以上步骤之后，下一步则是实现 Dubbo 服务


#### 实现 Dubbo 服务

`EchoService` 作为暴露的 Dubbo 服务接口，服务提供方 `spring-cloud-dubbo-server-sample` 需要将其实现：

```java
@org.apache.dubbo.config.annotation.Service
class EchoServiceImpl implements EchoService {

    @Override
    public String echo(String message) {
        return "[echo] Hello, " + message;
    }
}
```

其中，`@org.apache.dubbo.config.annotation.Service` 是 Dubbo 服务注解，仅声明该 Java 服务（本地）实现为 Dubbo 服务。
因此，下一步需要将其配置 Dubbo 服务（远程）。



#### 配置 Dubbo 服务提供方

在暴露 Dubbo 服务方面，推荐开发人员外部化配置的方式，即指定 Java 服务实现类的扫描基准包。
> Dubbo Spring Cloud 继承了 Dubbo Spring Boot 的外部化配置特性，也可以通过标注 `@DubboComponentScan` 来实现基准包扫描。

同时，Dubbo 远程服务需要暴露网络端口，并设定通讯协议，完整的 YAML 配置如下所示：

```yaml
dubbo:
  scan:
    # dubbo 服务扫描基准包
    base-packages: org.springframework.cloud.alibaba.dubbo.bootstrap
  protocol:
    # dubbo 协议
    name: dubbo
    # dubbo 协议端口（ -1 表示自增端口，从 20880 开始）
    port: -1
  registry:
    # 挂载到 Spring Cloud 注册中心
    address: spring-cloud://localhost
    
spring:
  application:
    # Dubbo 应用名称
    name: spring-cloud-alibaba-dubbo-server
  main:
    # Spring Boot 2.1 需要设定
    allow-bean-definition-overriding: true
  cloud:
    nacos:
      # Nacos 服务发现与注册配置
      discovery:
        server-addr: 127.0.0.1:8848
```

以上 YAML 内容，上半部分为 Dubbo 的配置：

- `dubbo.scan.base-packages` : 指定 Dubbo 服务实现类的扫描基准包
- `dubbo.protocol` : Dubbo 服务暴露的协议配置，其中子属性 `name` 为协议名称，`port` 为协议端口（ -1 表示自增端口，从 20880 开始）
- `dubbo.registry` : Dubbo 服务注册中心配置，其中子属性 `address` 的值 "spring-cloud://localhost"，说明挂载到 Spring Cloud 注册中心
> 当前 Dubbo Spring Cloud 实现必须配置 `dubbo.registry.address = spring-cloud://localhost`，下一个版本将其配置变为可选
> （参考 [issue #592](https://github.com/spring-cloud-incubator/spring-cloud-alibaba/issues/592)），
> 并且支持传统 Dubbo 协议的支持（参考 [issue #588](https://github.com/spring-cloud-incubator/spring-cloud-alibaba/issues/588)）

下半部分则是 Spring Cloud 相关配置：

- `spring.application.name` : Spring 应用名称，用于 Spring Cloud 服务注册和发现。
> 该值在 Dubbo Spring Cloud 加持下被视作 `dubbo.application.name`，因此，无需再显示地配置 `dubbo.application.name`
- `spring.main.allow-bean-definition-overriding` : 在 Spring Boot 2.1 以及更高的版本增加该设定，
因为 Spring Boot 默认调整了 Bean 定义覆盖行为。（推荐一个好的 Dubbo 讨论 [issue #3193](https://github.com/apache/incubator-dubbo/issues/3193#issuecomment-474340165)）
- `spring.cloud.nacos.discovery` : Nacos 服务发现与注册配置，其中子属性 server-addr 指定 Nacos 服务器主机和端口

> 以上完整的 YAML 配置文件，请参考 `spring-cloud-dubbo-server-sample` [`bootstrap.yaml`](spring-cloud-dubbo-server-sample/src/main/resources/bootstrap.yaml) 文件


完成以上步骤后，还需编写一个 Dubbo Spring Cloud 引导类。


#### 引导 Dubbo Spring Cloud 服务提供方应用

Dubbo Spring Cloud 引导类与普通 Spring Cloud 应用并无差别，如下所示：
```java
@EnableDiscoveryClient
@EnableAutoConfiguration
public class DubboSpringCloudServerBootstrap {

    public static void main(String[] args) {
        SpringApplication.run(DubboSpringCloudServerBootstrap.class);
    }
}
```

在引导 `DubboSpringCloudServerBootstrap` 之前，请提前启动 Nacos 服务器。
当 `DubboSpringCloudServerBootstrap` 启动后，将应用 `spring-cloud-dubbo-server-sample` 将出现在 Nacos 控制台界面。


当 Dubbo 服务提供方启动后，下一步实现一个 Dubbo 服务消费方。



### 实现 Dubbo 服务消费方

由于 Java 服务就 `EchoService`、服务提供方应用 `spring-cloud-dubbo-server-sample` 以及 Nacos 服务器均已准备完毕。Dubbo 服务消费方
只需初始化服务消费方 Maven 工程 `spring-cloud-dubbo-client-sample` 以及消费 Dubbo 服务。



#### 初始化 `spring-cloud-dubbo-client-sample` Maven 工程

与服务提供方 Maven 工程类，需添加相关 Maven 依赖：

```xml
<dependencyManagement>
    <dependencies>
        <!-- Spring Cloud Alibaba dependencies -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>0.9.0.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <!-- Sample API -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-dubbo-sample-api</artifactId>
        <version>${project.version}</version>
    </dependency>

    <!-- Spring Boot dependencies -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-actuator</artifactId>
    </dependency>

    <!-- Dubbo Spring Cloud Starter -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-dubbo</artifactId>
    </dependency>

    <!-- Spring Cloud Nacos Service Discovery -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
</dependencies>
```

与应用 `spring-cloud-dubbo-server-sample` 不同的是，当前应用依赖 `spring-boot-starter-web`，表明它属于 Web Servlet 应用。

> 以上完整的 Maven 依赖配置，请参考 `spring-cloud-dubbo-client-sample` [`pom.xml`](spring-cloud-dubbo-client-sample/pom.xml) 文件


#### 配置 Dubbo 服务消费方

Dubbo 服务消费方配置与服务提供方类似，当前应用 `spring-cloud-dubbo-client-sample` 属于纯服务消费方，因此，所需的外部化配置更精简：

```yaml
dubbo:
  registry:
    # 挂载到 Spring Cloud 注册中心
    address: spring-cloud://localhost
  cloud:
    subscribed-services: spring-cloud-alibaba-dubbo-server
    
spring:
  application:
    # Dubbo 应用名称
    name: spring-cloud-alibaba-dubbo-client
  main:
    # Spring Boot 2.1 需要设定
    allow-bean-definition-overriding: true
  cloud:
    nacos:
      # Nacos 服务发现与注册配置
      discovery:
        server-addr: 127.0.0.1:8848
```

对比应用 `spring-cloud-dubbo-server-sample`，除应用名称 `spring.application.name` 存在差异外，`spring-cloud-dubbo-client-sample`
新增了属性 `dubbo.cloud.subscribed-services` 的设置。并且该值为服务提供方应用 "spring-cloud-dubbo-server-sample"。

- `dubbo.cloud.subscribed-services` : 用于服务消费方订阅服务提供方的应用名称的列表，若需订阅多应用，使用 "," 分割。
不推荐使用默认值为 "*"，它将订阅所有应用。
> 当应用使用属性 `dubbo.cloud.subscribed-services` 默认值时，日志中将会输出一行警告：
> > Current application will subscribe all services(size:x) in registry, a lot of memory and CPU cycles may be used,
> > thus it's strongly recommend you using the externalized property 'dubbo.cloud.subscribed-services' to specify the services

由于当前应用属于 Web 应用，它会默认地使用 8080 作为 Web 服务端口，如果需要自定义，可通过属性 `server.port` 调整。

> 以上完整的 YAML 配置文件，请参考 `spring-cloud-dubbo-client-sample` [`bootstrap.yaml`](spring-cloud-dubbo-client-sample/src/main/resources/bootstrap.yaml) 文件



#### 引导 Dubbo Spring Cloud 服务消费方应用

为了减少实现步骤，以下引导类将 Dubbo 服务消费以及引导功能合二为一：

```java
@EnableDiscoveryClient
@EnableAutoConfiguration
@RestController
public class DubboSpringCloudClientBootstrap {

    @Reference
    private EchoService echoService;

    @GetMapping("/echo")
    public String echo(String message) {
        return echoService.echo(message);
    }

    public static void main(String[] args) {
        SpringApplication.run(DubboSpringCloudClientBootstrap.class);
    }
}
```

不仅如此，`DubboSpringCloudClientBootstrap` 也作为 REST Endpoint，通过暴露 `/echo` Web 服务，消费 Dubbo `EchoService` 服务。因此，
可通过 `curl` 命令执行 HTTP GET 方法：

```
$ curl http://127.0.0.1:8080/echo?message=%E5%B0%8F%E9%A9%AC%E5%93%A5%EF%BC%88mercyblitz%EF%BC%89
```

HTTP 响应为：

```
[echo] Hello, 小马哥（mercyblitz）
```

以上结果说明应用 `spring-cloud-dubbo-client-sample` 通过消费 Dubbo 服务，返回服务提供方 `spring-cloud-dubbo-server-sample` 运算后的内容。



## 高阶示例

如果您需要进一步了解 Dubbo Spring Cloud 使用细节，可参考官方 Samples：https://github.com/spring-cloud-incubator/spring-cloud-alibaba/tree/master/spring-cloud-alibaba-examples/spring-cloud-alibaba-dubbo-examples

其子模块说明如下：
- spring-cloud-dubbo-sample-api：API 模块，存放 Dubbo 服务接口和模型定义
- spring-cloud-dubbo-provider-web-sample：Dubbo Spring Cloud 服务提供方示例（Web 应用）
- spring-cloud-dubbo-provider-sample：Dubbo Spring Cloud 服务提供方示例（非 Web 应用）
- spring-cloud-dubbo-consumer-sample：Dubbo Spring Cloud 服务消费方示例
- spring-cloud-dubbo-servlet-gateway-sample：Dubbo Spring Cloud Servlet 网关简易实现示例



## 问题反馈
如果您在使用 Dubbo Spring Cloud 的过程中遇到任何问题，请内容反馈至 https://github.com/spring-cloud-incubator/spring-cloud-alibaba/issues



## 进阶阅读
关于更多的 Dubbo Spring Cloud 特性以及设计细节，请关注 

- Spring Cloud Alibaba wiki - https://github.com/spring-cloud-incubator/spring-cloud-alibaba/wiki

- Dubbo 的博客：http://dubbo.apache.org/zh-cn/blog/index.html

  

## 下篇预告

接下的文章将会介绍 Dubbo Spring Cloud 高阶示例的运用和实现，敬请关注小马哥微信公众号：`次灵均阁`

[![img](https://camo.githubusercontent.com/36671f72e171f0c95f33f781f294b3e6bdceef07/68747470733a2f2f6d65726379626c69747a2e6769746875622e696f2f626f6f6b732f7468696e6b696e672d696e2d737072696e672d626f6f742f6173736574732f6d795f6d705f7172636f64652e6a7067)](https://camo.githubusercontent.com/36671f72e171f0c95f33f781f294b3e6bdceef07/68747470733a2f2f6d65726379626c69747a2e6769746875622e696f2f626f6f6b732f7468696e6b696e672d696e2d737072696e672d626f6f742f6173736574732f6d795f6d705f7172636f64652e6a7067)

获得最新 Dubbo Spring Cloud 相关资讯。


## [新书推荐](https://item.jd.com/12570242.html)


本书全名为[《Spring Boot 编程思想》](https://mercyblitz.github.io/books/thinking-in-spring-boot/)，是以 Spring Boot 2.0 为讨论的主线，讨论的范围将涵盖 Spring Boot 1.x 的所有版本，以及所关联的 Spring Framework 版本，致力于：
- 场景分析：掌握技术选型
- 系统学习：拒绝浅尝辄止
- 重视规范：了解发展趋势
- 源码解读：理解设计思想
- 实战演练：巩固学习成果

![clipboard.png](/img/bVbqfh2)

> 欢迎小伙伴在京东或当当订购
京东存有少量现货（随机发送签名版），可先睹为快：http://t.cn/ExjBU2M
当当价格优惠，需要五月初发货：http://t.cn/EX0QteF

## 分享推荐

### 免费分享

- [「小马哥技术周报」](https://www.douyu.com/mercyblitz)
	- [斗鱼直播](https://www.douyu.com/mercyblitz)
	- [B 站录播](http://space.bilibili.com/327910845/channel/detail?cid=52311)
- [「慕课网」](https://www.imooc.com/t/5387391)
	- [Spring Boot 2.0深度实践-初遇Spring Boot](https://www.imooc.com/learn/933) 
  - [Spring Boot 2.0深度实践之系列总览](https://www.imooc.com/learn/1058)
- [「SegmentFault」](https://segmentfault.com/u/mercyblitz)
	- [「小马哥 2019 跨年直播」一入 Java 深似海，从此“劝退”成必然](https://mp.weixin.qq.com/s?__biz=MzIxNDU4NjE1OQ==&mid=2247484085&idx=1&sn=5905f53e69bae9d48b3783a83bde40b3)	



### 收费分享

- [「小马哥 Java 知识星球」](http://t.cn/RnxUYzd)
> 深入探讨 Java 相关技术，包括行业动态，架构设计，设计模式，框架使用，源码分析等。

- SegmentFault 直播
  - [《Java 微服务实践 - Spring Boot / Spring Cloud》](http://t.cn/RoC0nNi)
  - [《一入 Java 深似海》](http://t.cn/E6bTa9O)

- 慕课视频
  - [《Spring Boot 2.0深度实践之核心技术篇》](http://t.cn/ReChCU9)

- 慕课网
    - [Spring Boot 2.0深度实践之核心技术篇](https://coding.imooc.com/class/252.html)
    - Spring Boot 2.0深度实践之生态整合篇（即将上线...）


## 关于作者

小马哥，Java 劝退师，Apache 和 Spring Cloud 等知名开源架构成员，[点击查看详情](https://mercyblitz.github.io/about/)。

---
[^1]: Spring Cloud provides tools for developers to quickly build some of the common patterns in distributed systems (e.g. configuration management, service discovery, circuit breakers, intelligent routing, micro-proxy, control bus, one-time tokens, global locks, leadership election, distributed sessions, cluster state). - https://spring.io/projects/spring-cloud
[^2]: Apache Dubbo (incubating) |ˈdʌbəʊ| is a high-performance, light weight, java based RPC framework. - https://dubbo.apache.org

[^3]: Spring Cloud Alibaba provides a one-stop solution for distributed application development. - https://github.com/spring-cloud-incubator/spring-cloud-alibaba

[^4]: Nacos is an easy-to-use dynamic service discovery, configuration and service management platform for building cloud native applications - https://nacos.io/

[^5]: Eureka is a REST (Representational State Transfer) based service that is primarily used in the AWS cloud for locating services for the purpose of load balancing and failover of middle-tier servers. - https://github.com/Netflix/eureka

[^6]: ZooKeeper is a centralized service for maintaining configuration information, naming, providing distributed synchronization, and providing group services. - https://zookeeper.apache.org

[^7]: Consul is a distributed service mesh to connect, secure, and configure services across any runtime platform and public or private cloud - https://www.consul.io/

[^8]: Spring Cloud Open Feign - https://github.com/spring-cloud/spring-cloud-openfeign

[^9]: 从 2.7.0 开始，Dubbo Spring Boot 与 Dubbo 在版本上保持一致

[^10]: Preview releases of Spring Cloud Alibaba are available: 0.9.0, 0.2.2, and 0.1.2 - <https://spring.io/blog/2019/04/19/preview-releases-of-spring-cloud-alibaba-are-available-0-9-0-0-2-2-and-0-1-2>

[^11]: 目前最新的 Spring Cloud “F” 版的版本为：`Finchley.SR2` - <https://cloud.spring.io/spring-cloud-static/Finchley.SR2/single/spring-cloud.html>

[^12]: 当前Spring Cloud “G” 版为 `Greenwich.RELEASE`

[^13]:  Spring Cloud 特性列表 - <https://cloud.spring.io/spring-cloud-static/Greenwich.RELEASE/single/spring-cloud.html#_features>

[^14]:  Dubbo 2.7 开始支持配置中心，可自定义适配 - <http://dubbo.apache.org/zh-cn/docs/user/configuration/config-center.html>

[^15]: Spring Cloud 原生注册中心，除 Eureka、Zookeeper、Consul 之外，还包括 Spring Cloud Alibaba 中的 Nacos

[^16]: Dubbo 原生注册中心 - <http://dubbo.apache.org/zh-cn/docs/user/references/registry/introduction.html>

[^17]: Alibaba Sentinel：Sentinel 以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性 - <https://github.com/alibaba/Sentinel/wiki/%E4%BB%8B%E7%BB%8D>，目前 Sentinel 已被 Spring Cloud 项目纳为 Circuit Breaker  的候选实现 - <https://spring.io/blog/2019/04/16/introducing-spring-cloud-circuit-breaker>

[^18]: Spring Cloud Sleuth - <https://spring.io/projects/spring-cloud-sleuth>

[^19]: Zipkin - <https://github.com/apache/incubator-zipkin>

