# Dubbo Cloud Native 实践与思考

<!-- TOC -->

- [Dubbo Cloud Native 实践与思考](#dubbo-cloud-native-%E5%AE%9E%E8%B7%B5%E4%B8%8E%E6%80%9D%E8%80%83)
    - [分享简介](#%E5%88%86%E4%BA%AB%E7%AE%80%E4%BB%8B)
    - [自我介绍](#%E8%87%AA%E6%88%91%E4%BB%8B%E7%BB%8D)
    - [主要议程](#%E4%B8%BB%E8%A6%81%E8%AE%AE%E7%A8%8B)
        - [Cloud Native 基础设施](#cloud-native-%E5%9F%BA%E7%A1%80%E8%AE%BE%E6%96%BD)
            - [服务发现（Service Discovery ）](#%E6%9C%8D%E5%8A%A1%E5%8F%91%E7%8E%B0service-discovery)
                - [如何选择](#%E5%A6%82%E4%BD%95%E9%80%89%E6%8B%A9)
                    - [Eureka](#eureka)
                    - [Consul](#consul)
                    - [Zookeeper](#zookeeper)
            - [负载均衡](#%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1)
            - [服务网关](#%E6%9C%8D%E5%8A%A1%E7%BD%91%E5%85%B3)
            - [分布式配置](#%E5%88%86%E5%B8%83%E5%BC%8F%E9%85%8D%E7%BD%AE)
            - [服务熔断](#%E6%9C%8D%E5%8A%A1%E7%86%94%E6%96%AD)
            - [链路跟踪](#%E9%93%BE%E8%B7%AF%E8%B7%9F%E8%B8%AA)
            - [服务监控](#%E6%9C%8D%E5%8A%A1%E7%9B%91%E6%8E%A7)
        - [Cloud Native 架构选型](#cloud-native-%E6%9E%B6%E6%9E%84%E9%80%89%E5%9E%8B)
            - [CNCF 架构体系](#cncf-%E6%9E%B6%E6%9E%84%E4%BD%93%E7%B3%BB)
            - [Spring Cloud 架构体系](#spring-cloud-%E6%9E%B6%E6%9E%84%E4%BD%93%E7%B3%BB)
            - [Dubbo 架构体系](#dubbo-%E6%9E%B6%E6%9E%84%E4%BD%93%E7%B3%BB)
        - [Dubbo Cloud Native 准备](#dubbo-cloud-native-%E5%87%86%E5%A4%87)
            - [Dubbo 注解驱动（Annotation-Driven）](#dubbo-%E6%B3%A8%E8%A7%A3%E9%A9%B1%E5%8A%A8annotation-driven)
                - [`@DubboComponentScan` 服务端示例](#dubbocomponentscan-%E6%9C%8D%E5%8A%A1%E7%AB%AF%E7%A4%BA%E4%BE%8B)
                - [`@DubboComponentScan` 客户端示例](#dubbocomponentscan-%E5%AE%A2%E6%88%B7%E7%AB%AF%E7%A4%BA%E4%BE%8B)
            - [Dubbo 外部化配置（Externalized Configuration）](#dubbo-%E5%A4%96%E9%83%A8%E5%8C%96%E9%85%8D%E7%BD%AEexternalized-configuration)
        - [现场演示环节](#%E7%8E%B0%E5%9C%BA%E6%BC%94%E7%A4%BA%E7%8E%AF%E8%8A%82)
            - [Dubbo 整合 Hystrix 示例](#dubbo-%E6%95%B4%E5%90%88-hystrix-%E7%A4%BA%E4%BE%8B)
                - [Dubbo 客户端实现](#dubbo-%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%AE%9E%E7%8E%B0)
                    - [Dubbo 服务端实现](#dubbo-%E6%9C%8D%E5%8A%A1%E7%AB%AF%E5%AE%9E%E7%8E%B0)
                - [测试客户端 REST 服务](#%E6%B5%8B%E8%AF%95%E5%AE%A2%E6%88%B7%E7%AB%AF-rest-%E6%9C%8D%E5%8A%A1)
    - [参考资源](#%E5%8F%82%E8%80%83%E8%B5%84%E6%BA%90)

<!-- /TOC -->

## 分享简介

Cloud Native 应用架构随着云技术的发展受到业界特别重视和关注，尤其是 CNCF（Cloud Native Computing Foundation）项目蓬勃发展之际。Dubbo 作为服务治理的标志性项目，自然紧跟业界的潮流，拥抱技术的变化。本次分享的议题包括介绍 Apache 孵化项目Dubbo Spring Boot Project 以及汇报 Dubbo 与 Cloud Native 整合过程中的一些实践与思考，如适配  Spring Cloud 、服务发现、服务网关、服务跟踪以及监控等。

> 注：为了读者的阅读方便和习惯，本文字稿将在演讲内容的基础上做出适当的调整。



## 自我介绍

马昕曦（小马哥），阿里巴巴中间件技术专家，十余年 Java EE 从业经验，Dubbo 维护者、架构师以及微服务布道师。目前主要负责阿里巴巴集团微服务技术实施、架构衍进、基础设施构建等。重点关注云计算、微服务以及软件架构等领域。通过 SUN Java（SCJP、SCWCD、SCBCD）以及 Oracle OCA 等的认证。



## 主要议程

今天我非常荣幸地与大家一起讨论关于 Dubbo Cloud Native 相关议题，本次议题紧扣“实践与思考“两个关键字，主要的议程包括：
* **Cloud Native 基础设施**
* **Cloud Native 架构选型** 
* **Dubbo Cloud Native 准备**





### Cloud Native 基础设施

关于 Cloud Native 的定义，不同的云平台可能给出的内容存在差异。此处，我向大家介绍目前最热门的 CNCF 的定义：

>  ”[CNCF Cloud Native Definition v1.0](https://github.com/cncf/toc/blob/master/DEFINITION.md)“ 中的描述：
>
> > Cloud native technologies empower organizations to build and run scalable applications in modern, dynamic environments such as public, private, and hybrid clouds. Containers, service meshes, microservices, immutable infrastructure, and declarative APIs exemplify this approach.

相对于其他学术流派，CNCF 的 Cloud Native 定义更为具体，偏向于软件技术。这一点我们从文中的一些关键字能够明显地体会到，如关键字 "Containers（容器）"、"service meshes"、”microservices（微服务）“等。通常，开发人员较为关注的 Cloud Native 基础设施为：“服务发现”、“负载均衡”、“服务网关”、“分布式配置”、“服务熔断”以及“跟踪监控”，如图所示：

![幻灯片05](/img/assets/幻灯片05.jpg)

由于 PPT 格式的限制，此处我将“链路跟踪”与“服务监控” 并陈为“跟踪监控”。接下来，我们进入“服务发现”的讨论。





#### 服务发现（Service Discovery ）

随着微服务架构（MSA）受到不同规模企业的青睐，服务治理的实施逐渐被提上基础设施改造的议程。尽管这些概念在 SOA 时代已经提出，然而引起业界广泛关注应归功于微服务。服务发现（Service Discovery ）作为服务治理的核心特性，通常也将服务注册（Service Registration）一并讨论。无论是服务发现，还是服务注册，在具体落地实施时，它们必须面对技术选型的问题。在座的各位，包括我，大多数是 Java 程序员，自然关心 Java 的技术方案。目前，Java 社区最为津津乐道的方案莫过于 Spring Cloud，搭配 Netflix OSS 组件 Eureka，帮助 Spring Boot 应用快速搭建服务发现体系。其中，Eureka Server 作为注册中心服务器，Spring Boot 应用整合 Eureka Client 向 Eureka Server 注册。实际上，Spring Cloud 除了整合 Netflix Eureka 作为服务发现之外，还提供了 Apache Zookeeper 和 HachiCorp Consul 的实现，所以这三种方案出现在当前页面：

![幻灯片06](/img/assets/幻灯片06.jpg)



其中还包括 Redis 和 Apache Curator，前者是 Dubbo 的服务发现实现方案之一，然而小马哥并不建议使用 Redis 作为注册中心，还是保持它缓存中间件的单纯性较好。而 Curator 作为 Zookeeper Java 客户端类库，它不但可用在 Dubbo，而且其扩展项目 Curator Service Discovery 也是 Spring Cloud 整合 Zookeeper 作为服务发现的关键基础设施。或许大家思考以上方案应该如何选型的问题。



##### 如何选择



###### Eureka

当服务发现选型时，Netflix Eureka 或许是在开发人员脑海中复现的首选方案。然而 Eureka 在阿里大规模实践时，它的表现并不理想，当 Eureka 客户端服务实例数量达到一定时，Eureka Server 时常会出现服务不可用的情况，主要的问题集中在更新（Update）机制、复制（Replication）机制以及内存型存储。由于时间的关系，此处我不加详细说明，部分答案在 Eureka Wiki [Eureka 2.0 Motivations](https://github.com/Netflix/eureka/wiki/Eureka-2.0-Motivations) 中也有描述：

> [Why Eureka 2.0?](https://github.com/Netflix/eureka/wiki/Eureka-2.0-Motivations#why-eureka-20)
>
> * Only support homogenous client views
> * Only supports scheduled updates
> * Replication algorithm limits scalability
>
> > 注：以上具体内容在分享现场并没有具体提及，此处特意为读者补充。



以上问题 Netflix 早在 2015 年已意识到，然而 Eureka 2.0 的发布遥遥无期。后来，我托朋友联系上了 Netflix 的工程师，咨询他们关于 Eureka 1 在自身生产环境的使用情况。他们的回复是部分场景在使用。这样的答复值得玩味，再细问其覆盖比重，对方三缄其口。这不得不让我对 Eureka 的成熟度产生了质疑，所以我不建议大家在数以千计的应用实例场景中使用。





###### Consul

Consul 同样作为 Spring Cloud 服务中心，基于 GO 语言开发，其数据一致性采用 Raft 算法，低内存，集群支持。曾一度成为我理想的替换 Eureka 的方案，不过本人并不具备 Consul 的大规模运用，为此还特意请教永辉云创的架构师翟永超（《Spring Cloud 微服务实战》的作者）。他告知 Consul 表现不错，并在跨 DC（数据中心）方面也比较稳定：![image-20180627210416358](/img/assets/image-20180627210416358.png)

他的答复让我增强了 Consul 的信心，稍显遗憾的是其 Consul 应用节点略少。后来，我听说 B 站的哥们自研服务发现中间件 **[discovery](https://github.com/Bilibili/discovery/)**，他们应该也对 Consul 做过调研和评估，他们的看法是：

![image-20180627210529749](/img/assets/image-20180627210529749.png)

> Github 开源地址：https://github.com/Bilibili/discovery/
>
> **[discovery](https://github.com/Bilibili/discovery/)** 在 B 站 K8S 上的使用情况：
>
> ![image-20180627210509857](/img/assets/image-20180627210509857.png)

综合两家公司的评估，尽管没有经过本人实际操作，并且两者没有提供具体的数据指标，然而在一定程度上说明 Consul 作为注册中心的实例节点规模大概在 2k 以内。换言之，它比较适合中小型企业。



###### Zookeeper

Zookeeper 即可是 Spring Cloud 注册中心，又能作为 Dubbo 注册中心，与 Eureka 不同，它属于 CP 分布式策略，而后者属于 AP。两者的共同点在于均属于内存型注册中心，在大规模集群场景，也会遇到 Eureka 类似的问题。不过从运维的角度，相较于 Eureka 而言，熟悉 Zookeeper 运维朋友更多。在生态性方面，Zookeeper 周边的生态更丰富，如 Zookeeper C API，尽管 Eureka 提供了语言无关性的 REST 接口。同时，Zookeeper 还从当配置服务器的角色，降低了学习的成本。综上结论，我推荐使用 Zookeeper 作为服务发现基础设施，无论您选择 Dubbo 方案，还是使用 Spring Cloud。尽管它在大规模集群时也出现 Zookeeper 间歇性卡顿等问题。





#### 负载均衡

![幻灯片07](/img/assets/幻灯片07.jpg)

负载均衡是第二个重要 Cloud Native 基础设施，熟悉 Spring Cloud 的朋友一定对右侧的蝴蝶结有印象，它就是 Netflix OSS 负载均衡组件 Ribbon，框架层面提供了多种负载均衡规则，如：

* **随机** - `RandomRule`
* **轮循** - `RoundRobinRule`
* **权重响应时间** - `WeightedResponseTimeRule`

`WeightedResponseTimeRule` 之外，其他的 Ribbon 负载均衡实现均没有提供权重因子，而权重因子对于蓝绿发布、服务预热等方面的帮助是至关重要的。因此，权重因子在 Dubbo “**随机**“、”**轮询**“ 以及 ”**最少活跃调用数**“ 负载均衡算法中均体现。

以上讨论的两种框架均属于 Java 实现，而中间的 Kong 则是更为通用的实现，通常它作为 API 服务网关，后面我们将继续讨论。可简单地认为它是 Nginx + Lua 的扩展，负载均衡自然成为不可或缺的特性。其默认的负载均衡算法为具备权重的轮询（weighted-round-robin），同时一致性 Hash 算法作为可选方案。



#### 服务网关

![幻灯片08](/img/assets/幻灯片08.jpg)

谈及服务网关，Java 工程师最容易想到的是 Spring Cloud Zuul。Zuul 是 Netflix 基于 Servlet API 开发的 Web 服务代理组件，在 Spring Cloud 使用场景中，它与 Eureka  和 Ribbon 整合，打造具备服务动态更新和负载均衡能力的服务网关。

最近，随着 Spring Cloud Finchley 的发布，Spring Cloud Zuul 的替代方案 Spring Cloud Gateway 孕育而生，不过官方的描述还是比较谦虚谨慎，并没有一刀切地引导开发人员从 Zuul 迁移到 Gateway 上来：

> API Gateway built on top of the Spring Ecosystem, including: Spring 5, Spring Boot 2 and Project Reactor. Spring Cloud Gateway aims to provide a simple, yet effective way to route to APIs and provide cross cutting concerns to them such as: security, monitoring/metrics, and resiliency.

两者不同点在于，Zuul 运行在 Servlet 容器中，而 Gateway 并不像 Spring WebFlux 能够兼容 Servlet 3.1 运行时，而是必须依赖 Netty 的运行时，以及整合 Reactive 框架 Reactor，实现异步非阻塞网关。由于近期对于 Spring 5  WebFlux 能够大幅提升应用性能的观点甚嚣尘上，实际上，没有任何直接性能基准测试证明 WebFlux 能够加快程序执行速度，或许大家认为我的观点与主流格格不入，可是我要告诉大家的是，这个问题我在同事间验证过很多次，大多数情况，Reactive 并不没有提升性能。就连 Spring 官方也承认这个观点：

> [1.1.7. Performance vs scale](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web-reactive.html#webflux-performance)
>
> Performance has many characteristics and meanings. **Reactive and non-blocking generally do not make applications run faster**. They can, in some cases, for example if using the `WebClient` to execute remote calls in parallel. **On the whole it requires more work to do things the non-blocking way and that can increase slightly the required processing time**.
>
> > 资源地址：https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web-reactive.html#webflux-performance

同时，这里提供一篇 [Spring 5 WebFlux: Performance tests](https://blog.ippon.tech/spring-5-webflux-performance-tests/) 的文章，在结尾部分给出了结论，作者坦言在速度上没有明显的提升，甚至从结果来看，速度稍微更糟糕：

- *No improvement in speed was observed with our reactive apps (the Gatling results are even slightly worse).*

> 以上测试工程和结论是由开源项目 JHipster 的工程师给出，具备一定的客观性和可信度。
>
> > 资源地址：https://blog.ippon.tech/spring-5-webflux-performance-tests/

换言之，基于 Reactor 开发的 Gateway 在性能可能并没有明显的提升。因此，Zuul 和 Gateway 的性能对比则演变为 Servlet 容器和 Netty Web 容器的比较，感兴趣的朋友可以去网上寻找一些比较数据，两者的性能在伯仲间。

当然，我和在座的各位一样，对 Java 的实现方案自然是情有独钟。然而我想说的是，身为 Java 工程师，**眼中难免有 Java，但是眼中不要只有 Java**。Nginx 作为当年著名 “C10K” 问题的解决方案，无论从连接数量，还是资源消耗方面均优于 Java 实现。作为技术人，应该具有更为宽广的胸怀，接纳非我族类的气魄，该放手的时候就放手。Nginx 作为服务网关不失为一种好的方案，然而它的动态性略为不足，需要结合 Lua 脚本辅助完成，因此，OpenResty 和 Kong 这类方案脱颖而出。如果就 HTTP API 网关而言，个人认为 Kong 的方案更佳，因为它提供完整的解决方案，包括前面讨论的负载均衡（权重）、服务熔断以及服务发现等特性。类似的特性在 CNCF 项目 Envoy 也有体现，它是另一种高性能代理的方案，提供服务发现、健康和负载均衡。在协议上，天然支持 HTTP 和 HTTP/2，而通讯协议支持 gRPC，建议大家予以高度关注。

值得一提的是，HTTP API 网关通常需要支持 sidecar，换言之，支撑网关服务的基础设施必须提供服务发现的能力，就功能性而言，Zuul 和 Gateway 自身并不具备这样的特性，需要搭配 Eureka 这样组件，它们更像服务路由器的角色。





#### 分布式配置

![幻灯片09](/img/assets/幻灯片09.jpg)

左边和中间的四种技术均为 Spring Cloud 分布式配置的底层存储，其中 Git 为版本式配置，而 JDBC 是从 Spring Cloud Edgware 版本开始支持，提供更为通用和动态的配置源。这里我们又见到 Zookeeper 的声影，从简化运维的角度，可以利用 Zookeeper 即承担服务发现，也作为分布式配置的基础设施。而最右边的 etcd 是最近非常火的 Kubernetes 分布式配置的 key-value 存储，提供快速、简单、安全和可高的解决方案。





#### 服务熔断

![幻灯片10](/img/assets/幻灯片10.jpg)

服务熔断也非常让开发人员联想到 Spring Cloud Hystrix 技术，不过 Hystrix 并非与 Spring Cloud 强耦合，当然 Dubbo 也能结合 Netflix Hystrix 框架提供服务熔断的能力，后面部分将介绍 Dubbo 与 Hystrix 整合，提升 Dubbo 服务熔断的能力。确切地说，Dubbo 所提供的能力是集群容错，包括 Failover 等模式。 Kong 也天然地支持服务熔断的能力，所以它作为 API 网关的特性是全面的。





#### 链路跟踪

![幻灯片11](/img/assets/幻灯片11.jpg)

以上链路跟踪的基础设施从左至右，分别为 Zipkin、OpenTracing 以及 Jaeger，三者的灵感均来自于 [Google 论文 Dapper](https://ai.google/research/pubs/pub36356)。相对而言，Java 程序员可能更为熟悉 Zipkin，因为它是 Spring Cloud Sleuth 首选方案，提供客户端上报以及服务端聚合和 Dashboard 等功能。而 OpenTracing 和 Jaeger 是 CNCF 孵化项目，前者属于开放的标准，提供多语言的适配实现，后者则由 Uber（优步）公司开发并开源的链路跟踪项目，功能上与 Zipkin 类似，不过它基于 GO 语言开发，同时也提供 Java 客户端。
> OpenTracing 官网：http://opentracing.io/
> jaeger 官网：https://www.jaegertracing.io/





#### 服务监控 

![幻灯片12](/img/assets/幻灯片12.jpg)

服务监控与链路跟踪有所区别，主要用于监控应用系统或业务的指标数据，可能是健康阈值，如 CPU 或 内存使用率，也可以是业务指标，如最近一小时的用户登录量。通常采用 Metrics 方式暴露，可使用客户端推送或服务端拉取的方式传输 Metrics 信息到数据中心。通常 Metrics 数据与时间是存在对应关系，因此，基本上采用时序型数据库来存储，如图中的 OpenTSDB。通常，Java 微服务应用会选择 Spring Boot 框架作为基础设施，如我之前设计的监控架构就采用了 Spring Boot + OpenTSDB ，后端存储基于 HBase。当时 Spring Boot Actuator Metrics 仅为简单的 Key Value 形式，自然 OpenTSDB 是理想的选择。随着 Spring Boot 2.0 开始支持 [Micrometer](https://micrometer.io/) 之后，使得 Spring Boot 的应用能够整合更多的 Micrometer 适配方案，其中名气较大的就是图中间的 [Prometheus](https://prometheus.io/)，它同样也是 CNCF 的孵化项目。

当然服务监控不只是 Metrics 方式，我所知道国内不少的公司采用了日志收集的方案，并搭配 ELK（Elasticsearch, Logstash, Kibana） 架构，减少运维成本。假设您没有使用该方案，或者仅使用了 Elasticsearch 的话，无论哪种方案，图形化界面的监控是必不可少的，因此我推荐 Grafana，该项目能够支持多种数据源，包括前文提到的 OpenTSDB、Prometheus 以及 ElasticSearch 等。由此，从数据采集、上报、聚合以及展示的特性上，这些基础设施帮助 Cloud Native 应用构建服务监控的闭环。

本议程介绍了一些 Cloud Native 技术设施，接下里我们继续讨论 Cloud Native 架构选型。





### Cloud Native 架构选型


#### CNCF 架构体系

![CNCF](/img/assets/CNCF.png)

CNCF 体系作为目前最热门的架构选型之一，基本上围绕着 Kubernetes 为中心而构建。个人认为，Java 业界和 CNCF 体系并没有达成共识，如服务网关，CNCF 主打 Envoy，而 Java 主要的方案为 Zuul 和 Spring Cloud Gateway。因此，个人建议是密切的关注 CNCF 的发展，不过个别孵化项目可以先行，如 Prometheus 和 Jaeger 等。 至于 CNCF 与 Java 生态的整合和落地，还得有待时日。





#### Spring Cloud 架构体系

![Spring Cloud](/img/assets/Spring-Cloud.png)

实际上，这个图片并非 Spring Cloud 组件架构，而是将其整合在 Pivotal Cloud Foundry (PCF) 架构中。基本上，Spring Cloud 功能组件均有所体现，包括 Eureka、Hystrix、Ribbon 等。不过值得注意的是，Spring Cloud Stream 是一套较为完整和抽象的流式编程框架，屏蔽了底层传输介质（不仅是消息服务），如 Kafka、RabbitMQ 等。除此之外，其他的组件可圈可点，如 Eureka 在大规模运用中的卡顿问题、Ribbon 缺少权重、Zuul 连接数限制和资源消耗、服务调用受限于 Feign REST 协议限制等。如果在小规模场景使用，以上限制或问题不明显，可以说 Spring Cloud 完全能够适任。

不过，差不多两年前，我曾在不同的公开场合讲过：”Spring Boot 易学难精，Spring Cloud 能用但不成熟“。当时很多人觉得我“离经叛道”，然而这句话并非空穴来风，是我这几年来 Java 微服务架构实施的心得。这两年来，深受 Spring Cloud “折磨”的小伙伴逐渐觉醒，慢慢地开始回到 Dubbo 等技术方案。如 Martin Fowler 在为“微服务”下定义时，提到通讯协议要用轻量级的 REST。假设微服务要做到服务无关的话，那么 Web Services 协议也是可以，尽管它看起来比较重，不过 Web Services 的结构化和强类型，可以省去不少的运行时校验逻辑。在我看来，微服务更大程度应该体现在服务粒度上，诚如 Netflix 前架构师 Adrian Cockcroft 说言：“Fine grain SOA”（微服务就是细粒度的 SOA），就这一点而言，比较容易地和业界达成共识。当我们把 Martin 的话视如圭臬时，我们是否要思考它是否经得起工程检验。这里，我没有兴趣贬低他人，来抬高自己（Dubbo），从而引导让大家放弃 Spring Cloud，而是我们需要给 Spring Cloud 时间，包括未来 Dubbo 也会向 Spring Cloud 靠拢并整合。在阿里的内部，基于 Nacos（马上开源的项目）和 Apache RocketMQ，实现了 Spring Cloud Service Discovery、Config 以及 Stream 等整合和适配，一旦时机成熟，可能会开源与大家共建。

既然谈到了 Dubbo，下面我们再来讨论 Dubbo 的架构体系。




#### Dubbo 架构体系

![Dubbo.png](/img/assets/Dubbo.png)

编程模型方面，不但支持传统的 Spring XML 配合方式，已经实现注解驱动以及外部化配置，并且全面支持最新的 Spring Boot 2.0，在不久的未来，大家会看到 Dubbo 与 Spring Cloud 的整合，使开发人员无缝地衔接已有的 Spring Cloud 应用。
> Dubbo Spring Boot 项目地址：https://github.com/apache/incubator-dubbo-spring-boot-project

注册中心方面，Dubbo 将整合 Eureka、etcd 以及 Consul 基础设施，深度与业界热门方案整合。

熔断机制方面，Dubbo 会在近期发布 Hystrix 整合实现，将编程友好性做得最大化。

通讯协议方面，Dubbo 将会支持 gRPC、Thrift 等热门通讯协议。

至于序列化协议，自然首先考虑的是 Protobuf，因其高层 gRPC 搭配 HTTP/2 快成或已经成为下一代通讯协议的事实标准，使得任何人无法忽视它们的存在。当然其他协议也会陆续支持。

其他方面，我这里就不一一介绍，总之，现在 Dubbo 已不再只是一个单一的 PRC 框架，而是要拥抱业界，形成完整的生态体系，与业界形成最大公约数。





### Dubbo Cloud Native 准备

![幻灯片16](/img/assets/幻灯片16.jpg)

在 Dubbo 架构体系时，我们曾提到编程模型的变化。从 Dubbo `2.5.8` 开始，注解驱动和外部化配置均已得到支持。同时，Dubbo 已经合并 Dubbox 代码，Java JAX-RS 标准得到了支持，目前业界事实的 REST 标准 Spring Web MVC 正在同步开发。Reactive 的支持也在同步进行，小马哥还得友好地提醒一下各位，对于 Reactive 的期望不应该过分的关注性能的提升。




#### Dubbo 注解驱动（Annotation-Driven）


在 Dubbo  `2.5.7` 之前的版本 ，Dubbo 提供了两个核心注解 `@Service` 以及 `@Reference`，分别用于Dubbo 服务提供和 Dubbo 服务引用。

其中，`@Service` 作为 XML 元素 `<dubbo:service>` 的替代注解，与 Spring Framework `@org.springframework.stereotype.Service` 类似，用于服务提供方 Dubbo 服务暴露。与之相对应的 `@Reference`，则是替代`<dubbo:reference>` 元素，类似于 Spring 中的 `@Autowired`。

 `2.5.7` 之前的Dubbo，与早期的 Spring Framework 2.5 存在类似的不足，即注解支持不够充分。注解需要和 XML 配置文件配合使用，如下所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
	http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

    <dubbo:application name="annotation-provider"/>
    <dubbo:registry address="127.0.0.1:4548"/>
    <dubbo:annotation package="com.alibaba.dubbo.config.spring.annotation.provider"/>

</beans>
```

不仅如此，当时的版本存在“ `@Service` Bean 不支持 Spring AOP” 以及 “`@Reference` 不支持字段继承性” 等问题。

从 `2.5.7` 开始，Dubbo 开始引入组件扫描 Annotation `@DubboComponentScan`，借鉴了 Spring Boot 1.3 引入的 `@ServletComponentScan`。

在职责上，`@DubboComponentScan` 相对于 Spring Boot `@ServletComponentScan` 更为繁重，原因在于处理 Dubbo  `@Service` 类暴露 Dubbo 服务外，还有帮助 Spring  Bean `@Reference`字段或者方法注入 Dubbo 服务代理。 

在场景上，Spring Framework `@ComponentScan` 组件扫描逻辑更为复杂。而在 `@DubboComponentScan`  只需关注 `@Service` 和 `@Reference` 处理。


> 注：更多 Dubbo 注解驱动的详情，请参考[《Dubbo 注解驱动（Annotation-Driven）》](https://github.com/mercyblitz/blogs/blob/master/java/dubbo/Dubbo-Annotation-Driven.md)


##### `@DubboComponentScan` 服务端示例

假设，服务提供方和服务消费分均依赖服务接口`DemoService`:

```java
package com.alibaba.dubbo.demo;

public interface DemoService {

    String sayHello(String name);

}
```

* 服务提供方实现`DemoService`  - `AnnotationDemoService` 

同时标注 Dubbo `@Service` ：

```java
package com.alibaba.dubbo.demo.provider;

import com.alibaba.dubbo.config.annotation.Service;
import com.alibaba.dubbo.demo.DemoService;

/**
 * Annotation {@link DemoService} 实现
 *
 * @author <a href="mailto:mercyblitz@gmail.com">Mercy</a>
 */
@Service
public class AnnotationDemoService implements DemoService {

    @Override
    public String sayHello(String name) {
        return "Hello , " + name;
    }

}
```

* 服务端 `@Configuration` Class

```java
package com.alibaba.dubbo.demo.config;

import com.alibaba.dubbo.config.ApplicationConfig;
import com.alibaba.dubbo.config.ProtocolConfig;
import com.alibaba.dubbo.config.RegistryConfig;
import com.alibaba.dubbo.config.spring.context.annotation.DubboComponentScan;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * 服务提供方配置
 *
 * @author <a href="mailto:mercyblitz@gmail.com">Mercy</a>
 */
@Configuration
@DubboComponentScan("com.alibaba.dubbo.demo.provider") // 扫描 Dubbo 组件
public class ProviderConfiguration {

    /**
     * 当前应用配置
     */
    @Bean("dubbo-annotation-provider")
    public ApplicationConfig applicationConfig() {
        ApplicationConfig applicationConfig = new ApplicationConfig();
        applicationConfig.setName("dubbo-annotation-provider");
        return applicationConfig;
    }

    /**
     * 当前连接注册中心配置
     */
    @Bean("my-registry")
    public RegistryConfig registryConfig() {
        RegistryConfig registryConfig = new RegistryConfig();
        registryConfig.setAddress("N/A");
        return registryConfig;
    }

    /**
     * 当前连接注册中心配置
     */
    @Bean("dubbo")
    public ProtocolConfig protocolConfig() {
        ProtocolConfig protocolConfig = new ProtocolConfig();
        protocolConfig.setName("dubbo");
        protocolConfig.setPort(12345);
        return protocolConfig;
    }
}
```

* 服务提供方引导类

```java
package com.alibaba.dubbo.demo.bootstrap;

import com.alibaba.dubbo.demo.DemoService;
import com.alibaba.dubbo.demo.config.ProviderConfiguration;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

/**
 * 服务提供方引导类
 *
 * @author <a href="mailto:mercyblitz@gmail.com">Mercy</a>
 */
public class ProviderBootstrap {

    public static void main(String[] args) {
        // 创建 Annotation 配置上下文
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
        // 注册配置 Bean
        context.register(ProviderConfiguration.class);
        // 启动上下文
        context.refresh();
        // 获取 DemoService Bean
        DemoService demoService = context.getBean(DemoService.class);
        // 执行 sayHello 方法
        String message = demoService.sayHello("World");
        // 控制台输出信息
        System.out.println(message);
    }
    
}
```




##### `@DubboComponentScan` 客户端示例


* 消费服务 `DemoService`

```java
package com.alibaba.dubbo.demo.consumer;

import com.alibaba.dubbo.config.annotation.Reference;
import com.alibaba.dubbo.demo.DemoService;

/**
 * Annotation 驱动 {@link DemoService} 消费方
 *
 * @author <a href="mailto:mercyblitz@gmail.com">Mercy</a>
 */
public class AnnotationDemoServiceConsumer {

    @Reference(url = "dubbo://127.0.0.1:12345")
    private DemoService demoService;

    public String doSayHell(String name) {
        return demoService.sayHello(name);
    }
}
```

* 消费端 `@Configuration` Class

与服务提供方配置类似，服务消费方也许 Dubbo 相关配置 Bean - `ConsumerConfiguration`


```java
package com.alibaba.dubbo.demo.config;

import com.alibaba.dubbo.config.ApplicationConfig;
import com.alibaba.dubbo.config.RegistryConfig;
import com.alibaba.dubbo.config.spring.context.annotation.DubboComponentScan;
import com.alibaba.dubbo.demo.consumer.AnnotationDemoServiceConsumer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * 服务消费方配置
 *
 * @author <a href="mailto:mercyblitz@gmail.com">Mercy</a>
 */
@Configuration
@DubboComponentScan
public class ConsumerConfiguration {

    /**
     * 当前应用配置
     */
    @Bean
    public ApplicationConfig applicationConfig() {
        ApplicationConfig applicationConfig = new ApplicationConfig();
        applicationConfig.setName("dubbo-annotation-consumer");
        return applicationConfig;
    }

    /**
     * 当前连接注册中心配置
     */
    @Bean
    public RegistryConfig registryConfig() {
        RegistryConfig registryConfig = new RegistryConfig();
        registryConfig.setAddress("N/A");
        return registryConfig;
    }

    /**
     * 注册 AnnotationDemoServiceConsumer，@DubboComponentScan 将处理其中 @Reference 字段。
     * 如果 AnnotationDemoServiceConsumer 非 Spring Bean 的话，
     * 即使 @DubboComponentScan 指定 package 也不会进行处理，与 Spring @Autowired 同理
     */
    @Bean
    public AnnotationDemoServiceConsumer annotationDemoServiceConsumer() {
        return new AnnotationDemoServiceConsumer();
    }

}
```

* 服务消费方引导类

```java
package com.alibaba.dubbo.demo.bootstrap;

import com.alibaba.dubbo.demo.config.ConsumerConfiguration;
import com.alibaba.dubbo.demo.config.ProviderConfiguration;
import com.alibaba.dubbo.demo.consumer.AnnotationDemoServiceConsumer;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

/**
 * 服务消费端引导类
 *
 * @author <a href="mailto:mercyblitz@gmail.com">Mercy</a>
 */
public class ConsumerBootstrap {

    public static void main(String[] args) {
        // 启动服务提供方上下文
        startProviderContext();
        // 启动并且返回服务消费方上下文
        ApplicationContext consumerContext = startConsumerContext();
        // 获取 AnnotationDemoServiceConsumer Bean
        AnnotationDemoServiceConsumer consumer = consumerContext.getBean(AnnotationDemoServiceConsumer.class);
        // 执行 doSayHello 方法
        String message = consumer.doSayHello("World");
        // 输出执行结果
        System.out.println(message);
    }

    /**
     * 启动并且返回服务消费方上下文
     *
     * @return AnnotationConfigApplicationContext
     */
    private static ApplicationContext startConsumerContext() {
        // 创建服务消费方 Annotation 配置上下文
        AnnotationConfigApplicationContext consumerContext = new AnnotationConfigApplicationContext();
        // 注册服务消费方配置 Bean
        consumerContext.register(ConsumerConfiguration.class);
        // 启动服务消费方上下文
        consumerContext.refresh();
        // 返回服务消费方 Annotation 配置上下文
        return consumerContext;
    }

    /**
     * 启动服务提供方上下文
     */
    private static void startProviderContext() {
        // 创建 Annotation 配置上下文
        AnnotationConfigApplicationContext providerContext = new AnnotationConfigApplicationContext();
        // 注册配置 Bean
        providerContext.register(ProviderConfiguration.class);
        // 启动服务提供方上下文
        providerContext.refresh();
    }

}
```


#### Dubbo 外部化配置（Externalized Configuration）

在Dubbo 注解驱动例子中，无论是服务提供方，还是服务消费方，均需要转配相关配置Bean：

```java
    @Bean
    public ApplicationConfig applicationConfig() {
        ApplicationConfig applicationConfig = new ApplicationConfig();
        applicationConfig.setName("dubbo-annotation-consumer");
        return applicationConfig;
    }
```

虽然实现类似于`ProviderConfiguration` 和 `ConsumerConfiguration` 这样的 Spring  `@Configuration` Bean 成本并不高，不过通过 Java Code 的方式定义配置 Bean，或多或少是一种 Hard Code（硬编码）的行为，缺少弹性。

尽管在 Spring 应用中，可以通过 `@Value` 或者 `Environment` 的方式获取外部配置，其代码简洁性以及类型转换灵活性存在明显的不足。因此，Spring Boot  提出了外部化配置（External Configuration）的感念，即通过程序以外的配置源，动态地绑定指定类型。

随着 Spring Boot / Spring Cloud 应用的流行，开发人员逐渐地接受并且使用 Spring Boot 外部化配置（External Configuration），即通过 `application.properties` 或者 `bootstrap.properties` 装配配置 Bean。

下列表格记录了 Dubbo 内置配置类：

| 配置类              | 标签                   | 用途         | 解释                                                                    |
| ------------------- | ---------------------- | ------------ | ----------------------------------------------------------------------- |
| `ProtocolConfig`    | `<dubbo:protocol/>`    | 协议配置     | 用于配置提供服务的协议信息，协议由提供方指定，消费方被动接受            |
| `ApplicationConfig` | `<dubbo:application/>` | 应用配置     | 用于配置当前应用信息，不管该应用是提供者还是消费者                      |
| `ModuleConfig`      | `<dubbo:module/>`      | 模块配置     | 用于配置当前模块信息，可选                                              |
| `RegistryConfig`    | `<dubbo:registry/>`    | 注册中心配置 | 用于配置连接注册中心相关信息                                            |
| `MonitorConfig`     | `<dubbo:monitor/>`     | 监控中心配置 | 用于配置连接监控中心相关信息，可选                                      |
| `ProviderConfig`    | `<dubbo:provider/>`    | 提供方配置   | 当 ProtocolConfig 和 ServiceConfig 某属性没有配置时，采用此缺省值，可选 |
| `ConsumerConfig`    | `<dubbo:consumer/>`    | 消费方配置   | 当 ReferenceConfig 某属性没有配置时，采用此缺省值，可选                 |
| `MethodConfig`      | `<dubbo:method/>`      | 方法配置     | 用于 ServiceConfig 和 ReferenceConfig 指定方法级的配置信息              |
| `ArgumentConfig`    | `<dubbo:argument/>`    | 参数配置     | 用于指定方法参数配置                                                    |

通过申明对应的 Spring 扩展标签，在 Spring 应用上下文中将自动生成相应的配置 Bean。

在 Dubbo 官方用户手册的[“属性配置”](http://dubbo.io/books/dubbo-user-book/configuration/properties.html)章节中，`dubbo.properties` 配置属性能够映射到  `ApplicationConfig` 、`ProtocolConfig` 以及 `RegistryConfig` 的字段。从某种意义上来说，`dubbo.properties`  也是 Dubbo 的外部化配置。

> 注：更多外部化配置的详情，请参考[《Dubbo 外部化配置（Externalized Configuration）》](https://github.com/mercyblitz/blogs/blob/master/java/dubbo/Dubbo-Externalized-Configuration.md)





### 现场演示环节

> 本环境为分享后部分，现在编码 + 演示环境，当前文字仅提供代码实现。


#### Dubbo 整合 Hystrix 示例

> 本示例出现在分享议程的代码演示，将其放置此处，方便阅读理解



##### Dubbo 客户端实现



* 实现 `HystrixCommand`

```java
public class ResultHystrixCommand extends HystrixCommand<Result> {

    private final Invoker<?> invoker;

    private final Invocation invocation;

    public ResultHystrixCommand(Invoker<?> invoker, Invocation invocation) {
        super(HystrixCommandGroupKey.Factory.asKey(
                "ResultHystrixCommand"),
                100); // 设置超时时间
        // 关联 Dubbo Invoker 和 Invocation
        this.invoker = invoker;
        this.invocation = invocation;
    }

    @Override
    protected Result run() throws Exception {
        // 远程方法调用执行
        return invoker.invoke(invocation);
    }
}
```



当目标方法执行时间超过 100 ms 时，触发熔断，并抛出 `new UnsupportedOperationException("No fallback available.")`。



* Dubbo  `Filter` 整合 `ResultHystrixCommand`

```java
@Activate(group = Constants.CONSUMER, value = "hystrix") // 命名当前 Filter 为 "hystrix"
public class HystrixFilter implements Filter {

    @Override
    public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
        return new ResultHystrixCommand(invoker, invocation).execute();
    }
}
```



* 创建并配置 `Filter` SPI 配置文件

在相对于 ClassPath 资源 `META-INF/dubbo/` 下创建 `com.alibaba.dubbo.rpc.Filter`，并配置如下：

```properties
hystrix=com.alibaba.boot.dubbo.demo.consumer.filter.HystrixFilter
```



* 配置 `@Reference` `filter` 属性

```java
@RestController
public class DemoConsumerController {

    @Reference(
            version = "${demo.service.version}",
            application = "${dubbo.application.id}",
            url = "dubbo://localhost:12345",
            filter = "hystrix" // 指向 HystrixFilter 实现
    )
    private DemoService demoService;

    @RequestMapping("/sayHello")
    public String sayHello(@RequestParam String name) {
        return demoService.sayHello(name);
    }

}
```



###### Dubbo 服务端实现

* 服务提供者实现

```java
@Service(
        version = "${demo.service.version}",
        application = "${dubbo.application.id}",
        protocol = "dubbo",
        registry = "${dubbo.registry.id}"
)
public class DefaultDemoService implements DemoService {

    private final Random random = new Random();

    public String sayHello(String name) {
        hold();
        return "Say : Hello, " + name + " (from Spring Boot)";
    }

    private void hold() { // 随机等待 < 200 ms，当时间超过 100 ms 时，触发客户端熔断
        long time = random.nextInt(200);
        System.out.println("To hold " + time + " ms!");
        try {
            Thread.sleep(time);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }

}
```



##### 测试客户端 REST 服务

依次启动服务端 和客户端 Spring Boot 应用，

* 执行 `curl` 命令

```
mercyblitz$ curl http://localhost:8080/sayHello?name=Hello
```

* 测试结果

```
{"timestamp":"2018-06-23T01:33:58.682+0000","status":500,"error":"Internal Server Error","message":"ResultHystrixCommand timed-out and no fallback available.","path":"/sayHello"}
```

运行结果说明服务端方法执行超过 100 ms，引起客户端熔断。

（EOF）

## 参考资源

* [Dubbo 官网](https://dubbo.apache.org/)：https://dubbo.apache.org/
* [Dubbo 工程](https://github.com/apache/incubator-dubbo)：https://github.com/apache/incubator-dubbo
* [Dubbo Spring Boot 工程](https://github.com/apache/incubator-dubbo-spring-boot-project)：https://github.com/apache/incubator-dubbo-spring-boot-project
* [CNCF Landscape](https://landscape.cncf.io/)：https://landscape.cncf.io/
* [Spring Cloud 官网](https://projects.spring.io/spring-cloud/)：https://projects.spring.io/spring-cloud/
* [Kong 社区官网](https://konghq.com/kong-community-edition/)：https://konghq.com/kong-community-edition/
* [Opentracing 官网](http://opentracing.io/)：http://opentracing.io/
* [Jaeger 官网](https://www.jaegertracing.io/)：https://www.jaegertracing.io/
* [Prometheus 官网](https://prometheus.io/)：https://prometheus.io/
* [OpenTsdb 官网](http://opentsdb.net/)：http://opentsdb.net/
* [Grafana 官网](https://grafana.com/)：https://grafana.com/
* [小马哥 Github](https://github.com/mercyblitz)：https://github.com/mercyblitz 