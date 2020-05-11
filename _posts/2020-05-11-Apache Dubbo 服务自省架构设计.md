# 背景

随着微服务架构的推广和普及，服务之间的耦合度在逐步降低。在演化的过程中，伴随着应用组织架构的变化以及基础设施的衍进，服务和应用之间的边界变得更为模糊。Java 作为一门面向对象的编程语言，Java 接口（interface）作为服务之间通讯的一等公民，配合文档（JavaDoc）便于开发人员理解和维护。基于相同的编程哲学，Apache Dubbo 作为传统的 RPC 服务治理框架，通过接口实现分布式服务。然而对于微服务治理而言，应用（或“服务”）才是基础设施的核心要素。面对云原生（Cloud Native）技术的兴起，传统的 Dubbo 架构不断地面临着新的的挑战。下面内容将以 Apache Dubbo 2.7.5 为基础，介绍全新架构 - Apache Dubbo 服务自省（后文简称“服务自省”），了解 Dubbo 传统架构所面临的现实挑战，以及服务自省架构的设计和解决之道。





# 术语约定

- Service：SOA 或微服务中的“服务”，或称之为“应用”，具有全局唯一的名称
- Service Name: 服务名称，或应用名称
- Servce Instance：服务实例，或称为应用实例（Application Instance），表示单个 Dubbo 应用进程
- Registry：注册中心
- Dubbo 服务：又称之为“Dubbo 业务服务”，包含 Java 接口、通讯协议，版本（version）和分组（group）等元信息
- Dubbo 服务 ID：唯一鉴定 Dubbo 服务的元数据，用于 Dubbo 服务暴露（发布）和订阅
- Provider：Dubbo 服务提供方
- Consumer：Dubbo 服务消费方
- Dubbo 服务暴露：也称之为 Dubbo 服务发布，或英文中的“export”、"exported"
- Dubbo 应用服务：也称之为 Dubbo 业务服务，或业务 Dubbo 服务





# 使用场景

服务自省是 Dubbo 应用在运行时处理和分析 Dubbo 服务元信息（Metadata）的过程，如当前应用暴露 的Dubbo 服务以及各自的通讯协议等。期间会伴随着事件的广播和处理，如服务暴露事件。Dubbo 服务自省架构是其传统架的一种补充，更是未来 Dubbo 架构，它更适合以下使用场景：

- 超大规模 Dubbo 服务治理场景
- 微服务架构和元原生应用
- Dubbo 元数据架构的基石



## 超大规模 Dubbo 服务治理场景

如果 Dubbo 集群规模超过一千以上，或者集群扩缩容已无法自如地执行，如 Zookeeper 管理数万 Dubbo 服务，服务自省可极大化减轻注册中心的压力，尤其在内存足迹、网络传输以及变更通知上体现。



## 微服务架构和元原生应用

如果想要 Dubbo 应用更好地微服务化，或者更接近于云原生应用，那么服务自省是一种不错的选择，它能够提供已应用为粒度的服务注册与发现模型，全面地支持最流行的 Spring Cloud 和 Kubernetes 注册中心，并且能与 Spring Cloud 或 Spring Boot 应用交互。 



## Dubbo 元数据架构的基石

Dubbo 元数据架构是围绕 Dubbo DevOps 而引入，包括 Dubbo 配置元数据（如：属性配置、路由规则等）和结构元数据（如：Java 注解、接口和文档等）。服务自省作为 Dubbo 元数据的基础设施，不仅支持所有元数据的存储和平滑升级，而且不会对注册中心、配置中心和元数据中心产生额外的负担。





# 传统架构

Apache Dubbo 是一款面向接口代理的高性能 RPC 框架，提供服务注册与发现的特性，其基础架构如下图所示：

![image.png](https://cdn.nlark.com/yuque/0/2020/png/222258/1586917947915-c2d638b0-b87b-4fe0-883d-63b251055ddf.png)

**(图 1）**



- **Provider** 为服务提供方，提供 Java 服务接口的实现，并将其元信息注册到 Dubbo 注册中心（过程 1.register 所示）



- **Consumer** 为服务消费端，从 Dubbo 注册中心检索订阅的 Java 服务接口的元信息（过程 2.subscribe 所示），通过框架处理后，生成代理程序执行远程方法调用（过程 4.invoke 所示）



- **Registry** 为注册中心，属于注册元信息中心化基础设施（如 Apache Zookeeper 或 Alibaba Nacos），为 Provider 提供注册通道，为 Cosumer 提供订阅渠道。同时，注册中心支持注册元信息变更通知，通知 Consumer 上游 Provider 节点的变化（如扩容或缩容）。而注册元信息均以 Dubbo URL 的形式存储



- **Monitor** 为服务治理平台，提供开发和运维人员服务查询、路由规则、服务 Mock 和测试等治理能力



综上所述，**Dubbo 注册与发现的对象是 Dubbo 服务（Java 接口****），而其载体为****注册元信息，即** **Dubbo URL**，如：`dubbo://192.168.1.2:20880/com.foo.BarService?version=1.0.0&group=default`，通常包含必须信息，如服务提供方 IP 和端口、 Java 接口，可选包含版本（version）和分组（group）等。服务 URL 所包含的信息能够唯一界别服务提供方的进程。





# 现实挑战

为了更好地符合 Java 开发人员的编程习惯，Dubbo 以 Java 服务接口作为注册对象，所面临的现实挑战主要有：

- 如何解决或缓解注册中心压力过载
- 如何支持以应用为粒度的服务注册与发现
- 如何精简 Dubbo URL 元数据



## 如何解决或缓解注册中心压力过载



### 注册中心内存压力

Dubbo 注册中心是中心化的基础设施，大多数注册中心的实现为内存型存储，比如 Zookeeper、Nacos 或 Consul、Eureka。注册中心的内存消耗与 Dubbo 服务注册的数量成正比，任一 Dubbo Provider 允许注册 N 个 Dubbo 服务接口，当 N 越大，注册中心的负载越重。根据不完全统计，Dubbo 核心 Provider 用通常会暴露 20 ~ 50 个服务接口。注册中心是中心化的基础设施，其稳定性面临严峻考验。尽管微服务架构不断地深化，然而现实情况是，更多开发者仍旧愿意在单一 Provider 上不断地增加 Dubbo 服务接口，而非更细粒度的 Dubbo Provider 组织。



### 注册中心网络压力

为了避免单点故障，主流的注册中心均提供高可用方案。为解决集群环境数据同步的难题，内建一致性协议，如 Zookeeper 使用的 Zab 协议，Consul 采用的 Raft 协议。无论哪种方式，当 Dubbo URL 数量变化频繁时，网络和 CPU 压力也会面临考验。如果注册中心与客户端之间维持长连接状态的话，如 Zookeeper，注册中心的网络负担会更大。



### 注册中心通知压力

假设某个 Dubbo Provider 注册了 N 个 Dubbo 服务接口，当它扩容或缩容 M 个实例（节点）时，N 数量越大，注册中心至少有 M * N 个 Dubbo URL 注册或移除。同时，大多数注册中心实现支持注册变化通知，如 Zookeeper 节点变化通知。当 Dubbo Consumer 订阅该 Provider 的 Dubbo 服务接口数为 X 时，X 数值越大，通知的次数也就越多。实际上，对于来自同一 Provider 的服务接口集合而言，X-1 次通知是重复和无价值的。



如果 Dubbo 注册实体不再是服务 URL，而是 Dubbo Provider 节点的话，那么上述情况所描述的注册中心压力将得到很大程度的缓解。（负载只有过去的 1/N 甚至更少），然而 Dubbo 如何以应用为粒度来注册又是一个新的挑战。



## 如何支持以应用为粒度的服务注册与发现

尽管 Dubbo 也存在应用（Application）的概念，不过传统的使用场景并非核心要素，仅在 Dubbo Monitor 或 Dubbo Admin 场景下做辨识之用。随着微服务架构和云原生技术的兴起，以应用为粒度的注册模型已是大势所趋，如 Spring Cloud 和 Kubernetes 服务注册与发现模型。注册中心所管理的对象通常与业务无关，甚至不具备 RPC 的语义。在术语上，微服务架构中的“服务”（Services）与云原生中“应用”（Applications）是相同的概念，属于逻辑名称，而它们的成员则以服务实例（Service Instances）体现，服务和服务实例的数量关系为 1:N。



单个服务实例代表一个服务进程，而多个 Dubbo 服务 URL 可隶属一个 Dubbo Provider 进程，因此，Dubbo URL 与服务实例的数量关系是 N : 1。假设一个 Dubbo Provider 进程仅提供一个 Dubbo 服务（接口）的话，即 N = 1 的情况，虽然以应用为粒度的服务注册与发现能够基于 Dubbo 传统的 Registry SPI 实现，不过对于现有 Dubbo 应用而言，将存在巨大的应用微服务化工作。



### 支持 Spring Cloud 服务注册与发现模型

Spring Cloud 是 VMware 公司（前为 Pivotal）推出的，一套以 Spring 为技术栈的云原生（Cloud-Native）解决方案，在 Java 微服务领域具备得天独厚的优势，拥有超大规模的全球用户。Spring Cloud 官方支持三种注册中心实现，包括：Eureka、Zookeeper 和 Consul，Spring Cloud Alibaba 扩展了 Nacos 注册中心实现。 尽管 Zookeeper、Consul 和 Nacos 也被 Apache Dubbo 官方支持，然而两者的服务注册与发现的机制不尽相同。

若要 Dubbo 支持 Spring Cloud 服务注册与发现模型，Dubbo 则需基于 Dubbo Registry SPI 实现，否则底层的变化和兼容性存在风险。



### 支持 Kubernetes 服务注册与发现模型

Kubernetes 源自 Google 15 年生产环境的运维经验，是一个可移植的、可扩展的开源平台，用于管理容器化的工作负载和服务。Kubernetes 原生服务发现手段主要包括：DNS 和 API Server。DNS 服务发现是一种服务地址的通用方案，不过对于相对复杂 Dubbo 元数据而言，这种服务发现机制或许无法直接被 Dubbo Registry SPI 适配。相反，API Server 所支持相对更便利，毕竟 [Spring Cloud Kubernetes](https://github.com/spring-cloud/spring-cloud-kubernetes) 同样基于此机制实现，并已在生产环境得到验证。换言之，只要 Dubbo 支持 Spring Cloud 服务注册与发现模型，那么基于 Kubernetes API Server 的支持也能实现。



### 兼容 Dubbo 传统服务注册与发现模型

所谓兼容 Dubbo 传统服务注册与发现模型，包含两层含义：

- 基于 Dubbo Registry SPI 同时支持 Spring Cloud 和 Kubernetes 服务注册与发现模型
- 传统和新的 Dubbo 服务注册与发现模型之间能够相互发现



## 如何精简 Dubbo URL 元数据

Dubbo 从 2.7.0 开始增加了简化 URL 元数据的特性，被“简化”的数据存放至[元数据中心](http://dubbo.apache.org/en-us/docs/user/references/metadata/introduction.html)。由于 Dubbo 传统服务注册与发现模型并未减少 Dubbo 服务 URL 注册数量。因此，精简后的 URL 并未明显地减少注册中心所承受的压力。同时，Dubbo URL 元数据精简模式存在一定的限制，即所有的 Dubbo Provider 节点必须是无状态的，每个节点中的 URL 元信息均是一致的，现实中，这个要求非常难以保证，尤其在同一 Provider 节点存在不同的版本或配置的情况下。综上所述，Dubbo URL 元数据需要进一步精简，至少压力应该避免聚集在注册中心之上。





# 架构设计

架构上，Dubbo 服务自省不仅要解决上述挑战，而且实际场景则更为复杂，因此，架构细节也将循序渐进地展开讨论，整体架构可由以下子架构组成：

- 服务注册与发现架构
- 元数据服务架构
- 事件驱动架构



## 服务注册与发现架构

Dubbo 服务自省首要需求是减轻注册中心的承载的压力，同时，以应用为粒度的服务注册与发现模型不但能够最大化的减少 Dubbo 服务元信息注册数量，而且还能支持 Spring Cloud 和 Kubernetes 环境，可谓是一举两得，架构图如下所示：



![image.png](https://cdn.nlark.com/yuque/0/2020/png/222258/1586917948035-b9bc9e3e-7bd9-43ee-afb0-ecfb404b2d52.png)

**(图 2）**



### 注册实体

图中所示，从 Provider 和 Consumer 向注册中心注册的实体不再是 Dubbo URL，而是服务实例（Service Instance），一个服务实例代表一个 Provider 或 Consumer Dubbo 应用进程。服务实例属性包括：

- 服务名（Service Name）：该名称必须在注册中心全局唯一

> 注：名称规则架构上不做约束，不过不同注册中心的规则存在差异

- 主机地址（Host/IP）：能够被解析的主机名或者 TCP IP 地址
- 服务端口（Port）：应用进程所暴露的 Dubbo 协议端口，如 Dubbo 默认端口 20880

> 注：如果应用进程暴露多个 Dubbo 协议端口，如 dubbo 和 rest，那么，服务端口随机挑选其一，架构上不强制检验端口是否可用

- 元数据（Metadata）：服务实例的附加信息，用于存储 Dubbo 元信息，类似于通讯协议头或附件
- 激活状态（Enabled）：用于标记当前实例是否对外提供服务



上述**服务实例模型的支持依赖于注册中心的实现**。换言之，并非所有注册中心实现满足服务自省架构的要求。



### 注册中心

除了满足服务实例模型的要求之外，注册中心还得具备以下能力：

- 服务实例变化通知（Notification）：如上图步骤 4 所示，当 Consumer 订阅的 Provider 的服务实例发生变化时，注册中心能够实时地通知 Consumer
- 心跳检测（Heartbeats）：注册中心能够检测失效的服务实例，并且合理地移除它们



业界主流的注册中心中满足上述要求的有：

- Apache [Zookeeper](https://zookeeper.apache.org/)
- HashiCorp [Consul](https://www.consul.io/)
- Netflix [Eureka](https://github.com/netflix/Eureka)
- Alibaba [Nacos](https://nacos.io/)
- Kubernetes API Server



总之，**Spring Cloud 与** **Kubernetes** **注册中心均符合服务自省对注册中心的要求**。不过，在 Dubbo 传统 RPC 使用场景中，Provider 和 Consumer 关注的是 Dubbo 服务接口，而非 Service 或服务实例。假设需要将现有的 Dubbo 应用迁移至服务自省架构，Provider 和 Consumer 做大量的代码调整是不现实的。理想的情况下，两端实现代码均无变化，仅修改少量配置，就能达到迁移的效果。那么，Dubbo 服务接口是如何与 Service 进行映射的呢？



### Dubbo 服务与 Service 映射

前文曾讨论，单个 Dubbo Service 能够发布多个 Dubbo 服务，所以，Dubbo 服务与 Service 的数量关系是 N 对 1。不过，Dubbo 服务与 Dubbo Service 之间并不存在强绑定关系，换言之，某个 Dubbo 服务也能部署在多个 Dubbo Services 中，因此，Dubbo 服务与 Service 数量关系是 N 对 M（N, M >= 1），如下图所示：



![image.png](https://cdn.nlark.com/yuque/0/2020/png/222258/1586917947948-fddd2712-f32b-4d62-9404-58aae11ac92f.png)

**(图 3）**



上图中 P1 Service 到 P3 Service 为 Dubbo Service，com.acme.Interface1 到 com.acme.InterfaceN 则为 Dubbo 服务接口全称限定名（QFN）。值得注意的是，Dubbo 服务的 Java 接口（interface）允许不同的版本（version）或分组（group），所以仅凭 Java 接口无法唯一标识某个 Dubbo 服务，还需要增加通讯协议（protocol）方可，映射关系更新如下：

![image.png](https://cdn.nlark.com/yuque/0/2020/png/222258/1586917947946-75fd41ab-83a2-4175-a1a6-522a493e959e.png)**(图 4）**



Dubbo 服务 ID 字符表达模式为： `${protocol}:${interface}:${version}:${group}` , 其中，版本（version）或分组（group）是可选的。当 Dubbo Consumer 订阅 Dubbo 服务时，构建对应 ID，通过这个 ID 来查询 Dubbo Provider 的 Service 名称列表。



由于 Dubbo 服务与 Service 的映射关系取决于业务场景，架构层面无从预判。因此，这种映射关系只能在 Dubbo 服务暴露时（运行时）才能确定，否则，Dubbo 服务能被多个 Consumer 应用订阅时，Consumer 无法定位 Provider Service 名称，进而无法完成服务发现。同时，映射关系的数据通常采用配置的方式来存储，服务自省提供两种配置实现，即 “中心化映射配置” 和 “本地化映射配置”。





#### 中心化映射配置 

明显地，注册中心来扮演动态映射配置的角色并不适合，不然，Dubbo Service 与映射关系在注册中心是平级的，无论在理解上，还是设计上是混乱的。结合 Dubbo 现有基础设施分析，这个存储设施可由 Dubbo 配置中心承担。

其中 Dubbo 2.7.5 动态配置 API（`DynamicConfiguration` ）支持二级结构，即：group 和 key，其中，group 存储 Dubbo 服务 ID，而 key 则关联对应的 Dubbo Service 名称，对应的 "图 4” 的数据结构则是：

![image.png](https://cdn.nlark.com/yuque/0/2020/png/222258/1586927530780-d36670f9-6d0a-4263-a3e5-a64dbbf11cc6.png)

**(图 5）**



如此设计的原因如下：

1. 获取 Dubbo 服务对应 Services

利用 `DynamicConfiguration#getConfigKeys(String group)` 方法，能够轻松地通过 Dubbo 服务 ID 获取其发布的所有 Dubbo Services，结合服务发现接口获取服务所部署的 Service 实例集合，最终转化为  Dubbo `URL` 列表。



1. 避免 Dubbo Services 配置相互覆盖

以 Dubbo 服务 ID `dubbo:com.acme.Interface1:default` 为例，它的提供者 Dubbo Services 分别：P1 Service 和 P2 Service。假设配置 Group 为 "default"（任意名字均可）， Key 为 "dubbo:com.acme.Interface1:default"，而内容则是 Dubbo Service 名称的话。当 P1 Service 和 P2 Service 同时启动时，无论哪个 Services 最后完成 Dubbo 服务暴露，那么，该配置内容必然是二选其一，无论配置中心是否支持原子操作。即使配置中心支持内容追加的特性，由于两个 Service 服务实例过程不确定，配置内容可能会出现重复，如：“P1 Service,P2 Service,P1 Service”。



1. 获取 Dubbo 服务发布的 timestamp







##### 配置中心潜在的压力 

假设当 P1 Service 存在 5 个服务实例，当 Dubbo 服务 `dubbo:com.acme.Interface1:default`（ID）发布时，配置所关联的 key 就是当前 Dubbo Service 名称，即（P1 Service），而内容则是最后发布该 Dubbo 服务的时间戳（timestamp）。当服务实例越多时，配置中心和网络传输所承受的写入压力也就越大。当然架构设计上，服务自省也希望避免重复推送配置，比如在 `DynamicConfiguration` API 增加类似于 `publishConfigIfAbsent` 这样的方法，不过目前大多数配置中心产品（如：Nacos、Consul）不支持这样的操作，所以未来服务自省架构会有针对性的提供支持（如：Zookeeper）。



##### 注册中心作为配置中心

由于服务自省架构必须依赖注册中心，同时动态映射配置又依赖配置中心的话，应用的架构复杂度和维护成本均有所提升，不过 Apache Dubbo 所支持的部分注册中心也可作为配置中心使用，情况如下所示：

| 基础软件                                            | 注册中心 | 配置中心 |
| --------------------------------------------------- | -------- | -------- |
| Apache [Zookeeper](https://zookeeper.apache.org/)   | ✅        | ✅        |
| HashiCorp [Consul](https://www.consul.io/)          | ✅        | ✅        |
| Alibaba [Nacos](https://nacos.io/)                  | ✅        | ✅        |
| Netflix [Eureka](https://github.com/netflix/Eureka) | ✅        | ❌        |
| Kubernetes API Server                               | ✅        | ❌        |

其中，[Zookeeper](https://zookeeper.apache.org/)、[Consul](https://www.consul.io/) 和 [Nacos](https://nacos.io/) 是目前业界流行的注册中心，这对于大多数选择开源产品的应用无疑是一个福音。







#### 本地化映射配置

如果开发人员认为配置中心的引入增加了架构的复杂性，那么，静态映射配置或许是一种解决方案。

>  该特性并未在最新 Dubbo 2.7.6 全面发布，部分特性已在 [Dubbo Spring Cloud](https://github.com/alibaba/spring-cloud-alibaba/wiki/Dubbo-Spring-Cloud) 中发布



##### 接口映射配置

在 Dubbo 传统的编程模型中， 常以 Java 注解 `@Reference` 或 XML 元素 `` 订阅目标 Dubbo 服务。服务自省架构在此基础上增加 `service` 属性的映射一个或多个 Dubbo Service 名称，如：

```
<reference services="P1 Service,P2 Service" interface="com.acme.Interface1" />
```

或

```
@Reference(services="P1 Service,P2 Service") 
private com.acme.Interface1 interface1;
```

如此配置后，Dubbo 服务 `com.acme.Interface1` 将向 `p1-service` 和 `p2-service` 订阅服务。如果开发人员认为这种方式会侵入到代码，服务自省还提供外部化配置方式配置映射。



##### 外部化映射配置

服务自省架构支持外部化配置的方式声明“Dubbo 服务与 Service 映射”，配置格式为 `Properties` ，以图 4 为例，内容如下：

```
dubbo\:com.acme.Interface1\:default = P1 Service,P2 Service
thirft\:com.acme.InterfaceX = P1 Service,P3 Service
rest\:com.acme.interfaceN = P1 Service
```



##### 应用级别映射配置

除此之外，[Dubbo Spring Cloud](https://github.com/alibaba/spring-cloud-alibaba/wiki/Dubbo-Spring-Cloud) 提供应用级别的 Dubbo 服务映射配置，即 `dubbo.cloud.subscribed-services` ，例如：

```
dubbo:
    cloud:
    subscribed-services: P1 Service,P3 Service
```



总之，无论是映射配置的方式是中心化还是本地化，服务 Consumer 依赖这些数据来定位 Dubbo Provider Services，再通过服务发现 API 结合 Service 名称（列表）获取服务实例集合，为合成 Dubbo URL 做准备：

![image.png](https://cdn.nlark.com/yuque/0/2020/png/222258/1587022366127-0b590a0f-e457-4f62-a9c0-334864419481.png)

**(图 6）**



不过，映射关系并非是一种强约束，Dubbo Provider 的服务是否可用的检验方法是探测目标 Dubbo Service 是否存在，并需确认订阅的 Dubbo 服务在目标 Services 是否真实暴露，因此，服务自省引入了 Dubbo 元数据服务架构，来完成 Dubbo 服务 URL 的存储。





## 元数据服务架构

Dubbo 元数据服务是一个常规的 Dubbo 服务，为服务订阅端提供 Dubbo 元数据的服务目录，类似于 WebServices 中的 [WDSL](https://en.wikipedia.org/wiki/Web_Services_Description_Language) 或 REST 中的 [HATEOAS](https://en.wikipedia.org/wiki/HATEOAS)，帮助 Dubbo Consumer 获取订阅的 Dubbo 服务的 URL 列表。元数据服务架构无法独立于服务注册与发现架构而存在，下面通过“整体架构”的讨论，了解两者之间的关系。





### 整体架构

架构上，无论 Dubbo Service 属于 Provider 还是 Consumer，甚至是两者的混合，每个 Dubbo （Service）服务实例有且仅有一个 Dubbo 元数据服务。换言之，Dubbo Service 不存在纯粹的 Consumer，即使它不暴露任何业务服务，那么它也可能是 Dubbo 运维平台（如 Dubbo Admin）的 Provider。不过出于行文的习惯，Consumer 仍旧被定义为 Dubbo 服务消费者（应用）。由于每个 Dubbo Service 均发布自身的 Dubbo 元数据服务，那么，架构不会为不同的 Dubbo Service 设计独立的元数据服务接口（Java）。换言之，所有的 Dubbo Service 元数据服务接口是统一的，命名为 `MetadataService` 。



#### 微观架构

从 Dubbo 服务（URL）注册与发现的视角， `MetadataService` 扮演着传统 Dubbo 注册中心的角色。综合服务注册与发现架构（Dubbo Service 级别），微观架构如下图所示：



![image.png](https://cdn.nlark.com/yuque/0/2020/png/222258/1587024494773-541903b0-e760-46b0-9c08-516e973c5e18.png)

**(图 7）**

**
**

对于 Provider（服务提供者）而言，Dubbo 应用服务暴露与传统方式无异，而 `MetadataService` 的暴露时机必须在它们完成后，同时， `MetadataService` 需要收集这些 Dubbo 服务的 URL（存储细节将在“元数据服务存储模式“ 小节讨论）。假设某个 Provider 的 Dubbo 应用服务暴露数量为 N，那么，它所有的 Dubbo 服务暴露数量为 N + 1。



对于 Consumer（服务消费者）而言，获 Dubbo 应用服务订阅 URL 列表后，Dubbo 服务调用的方式与传统方式是相同的。不过在此之前，Consumer 需要通过 `MetadataService` 合成订阅 Dubbo 服务的 URL。该过程之所以称之为“合成”，而非“获取，是因为一次 `MetadataService` 服务调用仅在其 Provider 中的一台服务实例上执行，而该 Provider 可能部署了 N 个服务实例。具体“合成”的细节需要结合“宏观架构”来说明。



#### 宏观架构

元数据服务的宏观架构依赖于服务注册与发现架构，如下图所示：

![image.png](https://cdn.nlark.com/yuque/0/2020/png/222258/1587032163717-adb59fdc-893e-4f98-a6b8-dabbd3767e40.png)

**(图 8）**



图 8 中 p 和 c 分别代表 Provider 和 Consumer 的执行动作，后面紧跟的数字表示动作的次序，从 0 开始计数。执行动作是串行的，并属于 Fast-Fail 设计，如果前阶段执行失败，后续动作将不会发生。之所以如此安排是为了确保 `MetadataService` 能够暴露和消费。首先从 Provider 执行流程开始说明。



##### Provider 执行流程

- p0：发布所有的 Dubbo 应用服务，声明和定义方式与传统方式完全相同。
- p1：暴露 `MetadataService` ，该步骤完全由框架自行处理，无论是否 p0 是否暴露 Dubbo 服务
- p2：在服务实例注册之前， 框架将触发并处理事件（Event），将 `MetadataService` 的元数据先同步到服务实例（Service Instance）的元数据。随后，执行服务实例注册
- p3：建立所有的 Dubbo 应用服务与当前 Dubbo Service 名称的映射，并同步到配置源（抽象）



##### Consumer 执行流程

- c0：注册当前 Dubbo Service 的服务实例，可选步骤，架构允许 Consumer 不进行服务注册
- c1：通过订阅 Dubbo 服务元信息查找配置源，获取对应 Dubbo Services 名称（列表）
- c2：利用已有 Dubbo Service 名称（可能存在多个），通过服务发现 API 获取 Provider 服务实例集合。假设 Service 名称 P，服务实例数量为 N
- c3：

1. 1. 随机选择 Provider 一台服务实例 Px，从中获取 `MetadataService` 的元数据
   2. 将元数据组装 `MetadataService` Dubbo 调用客户端（代理）
   3. 发起 `MetadataService` Dubbo 调用，获取该服务实例 Px 所暴露的 Dubbo 应用服务 URL 列表
   4. 从 Dubbo 应用服务 URL 列表过滤出当前订阅 Dubbo 应用服务的 URL
   5. 理论上，步骤 c 和 d 还需要执行 N-1 次才能获取 P 所有服务实例的 Dubbo URL 列表。为了减少调用次数，步骤 d 的结果作为模板，克隆其他 N-1 台服务实例 URL 列表 
   6. 将所有订阅 Dubbo 应用服务的 URL 同步到 Dubbo 客户端（与传统方式是相同的）

- c4：发起 Dubbo 应用服务调用（与传统方式是相同的）



不难看出，上述架构以及流程结合了“服务注册与发现”与“元数据服务”双架构，步骤之间会触发相关 Dubbo 事件，如“服务实例注册前事件”等。换言之，三种架构综合体也就是服务自省架构。



至此，关于 Dubbo 服务自省架构设计方面，还存在一些细节亟待说明，比如：

1. 不同的 Dubbo Service 的 `MetadataService` 怎样体现差异呢？
2. `MetadataService` 作为一个常规的 Dubbo 服务，它的注册元信息存放在何处？
3. `MetadataService` 作为服务目录，它管理的 Dubbo 应用服务 URL 是如何存储的？
4. 在 Consumer 执行流程的 c3.e 中，克隆 N - 1 条 URL 的前提是该 Provider 的所有服务实例均部署了相同 Dubbo 应用服务。如果 Provider 处于升级的部署过程，同一 Dubbo 应用服务接口在不同的服务实例上存在差异，那么该服务的 URL 应该如何获取？
5. 除了 Dubbo 服务 URL 发现之外，元数据服务还支持哪些元数据类型呢？





### 元数据服务 Metadata

元数据服务 Metadata，称之为“元数据服务的元数据”，主要包括：

- inteface：Dubbo 元数据服务所暴露的接口，即 `MetadataService` 
- serviceName : 当前 `MetadataService` 所部署的 Dubbo Service 名称，作为 `MetadataService` 分组信息
- group：当前 `MetadataService` 分组，数据使用 serviceName
- version：当前 `MetadataService` 的版本，版本号通常在接口层面声明，不同的 Dubbo 发行版本 version 可能相同，比如 Dubbo 2.7.5 和 2.7.6 中的 version 均为 1.0.0。理论上，version 版本越高，支持元信息类型更丰富
- protocol： `MetadataService` 所暴露协议，为了确保 Provider 和 Consumer 通讯兼容性，默认协议为：“dubbo”，也可以支持其他协议。
- port：协议所使用的网络端口
- host：当前 `MetadataService` 所在的服务实例主机或 IP
- params：当前 `MetadataService` 暴露后 URL 中的参数信息



不难得出，凭借以上元数据服务的 Metadata，可将元数据服务的 Dubbo 服务 ID 确定，辅助 Provider 服务暴露和 Consumer 服务订阅 `MetadataService` 。不过对于 Provider，这些元信息都是已知的，而对 Consumer 而言，它们直接能获取的元信息仅有：

- serviceName：通过“Dubbo 接口与 Service 映射”关系，可得到 Provider Service 名称
- interface：即 `MetadataService` ，因为 Provider 和 Consumer 公用 `MetadataService` 接口
- group：即 serviceName



不过 Consumer 合成 `MetadataService` Dubbo URL 还需获取 version、host、port、protocol 以及 params：

- version：尽管 `MetadataService` 接口是统一接口，然而 Provider 和 Consumer 可能引入的 Dubbo 版本不同，从而它们使用的 `MetadataService` version 也会不同，所以这个信息需要 Provider 在暴露`MetadataService` 时，同步到服务实例的 Metadata 中，方便 Consumer 从 Metadata 中获取
- host：由于 Consumer 已得到 serviceName，可通过服务发现 API 获取服务实例对象，该对象包含 host 属性，直接被 Consumer 获取即可。
- port：与 version 类似，从 Provider 服务实例中的 Metadata 中获取
- params：同上



通过元数据服务 Metadata 的描述，解释了不同 Dubbo Services 是怎样体现差异性的，并且说明了 `MetadataService` 元信息的存储介质，这也就是服务自省架构为什么强依赖支持 Metadata 的注册中心的原因。下个小节将讨论 `MetadataService` 所存储 Dubbo 应用服务 URL 存放在何处。





### 元数据服务存储模式

Dubbo 2.7.5 在引入 `MetadataService` 的同时，也为其设计了两种存储方式，适用于不同的场景，即“本地存储模式”和“远程存储模式”。其中，本地存储模式是默认选项。



#### 元数据服务本地存储模式

本地存储模式又称之为内存存储模式（In-Memory），如 Dubbo 应用服务发现和注册场景中，暴露和订阅的 URL 直接存储在内存中。架构上，本地存储模式的 `MetadataService` 相当于去中心化的 Dubbo 应用服务的注册中心。

 

#### 元数据服务远程存储模式

远程存储模式，与去中心化的本地存储模式相反，采用 Dubbo 元数据中心来管理 Dubbo 元信息，又称之为元中心化存储模式（Metadata Center)。





#### 选择存储模式

为了减少负载压力和维护成本，服务自省中的元数据服务推荐使用**“****本****地存储模式”**。



回顾前文“Consumer 执行流程”中的步骤 c3.e，为了减少 `MetadataService` 调用次数，服务自省将第一次的调用结果作为模板，再结合其他 N-1 服务实例的元信息，合成完整的 N 台服务实例的 Dubbo 元信息。假设，Dubbo Service 服务实例中部署的 Dubbo 服务数量和内容不同，那么，c3.e 的执行步骤是存在问题的。因此，服务自省引入“**Dubbo 服务修订版本**”的机制来解决不对等部署的问题。



尽管“Dubbo 服务修订版本”机制能够介绍 `MetadataService` 整体消费次数，然而当新修订版本的服务实例过少，并且 Consumer 过多时，如新的版本 Provider 应用分批部署，每批的服务实例为 1 台，而其 Consumer 服务实例成千上万。为了确保这类场景的稳定性，Provider 和 Consumer 的 `MetadataService` 可选择“**远程存储模式**”，避免消费热点的发生。





### Dubbo 服务修订版本 

当业务出现变化时，Dubbo Service 的 Dubbo 服务也会随之升级。通常，Provider 先行升级，Consumer 随后跟进。



考虑以下场景，Provider “P1” 线上已发布 interface 为 `com.acme.Interface1`，group 为 `group` , version 为 `v1` ，即 Dubbo 服务 ID 为：`dubbo:com.acme.Interface1:v1:default` 。P1 可能出现升级的情况有：



1. Dubbo 服务 interface 升级

由于 Dubbo 基于 Java 接口来暴露服务，同时 Java 接口通常在 Dubbo 微服务中又是唯一的。如果 interface 的全类名调整的话，那么，相当于 `com.acme.Interface1` 做下线处理，Consumer 将无法消费到该 Dubbo 服务，这种情况不予考虑。如果是 Provider 新增服务接口的话，那么 `com.acme.Interface1` 则并没有变化，也无需考虑。所以，有且仅有一种情况考虑，即“**Dubbo interface 方法声明升级**”，包括：

- 增加服务方法
- 删除服务方法
- 修改方法签名



1. Dubbo 服务 group、version 和 protocol 升级

假设 P1 在升级过程中，新的服务实例部署仅存在调整 group 后的 Dubbo 服务，如 `dubbo:com.acme.Interface1:v1:test` ，那么这种升级就是不兼容升级，在新老交替过程中，Consumer 仅能消费到老版本的 Dubbo 服务。当新版本完全部署完成后，Consumer 将无法正常服务调用。如果，新版本中 P1 同时部署了 `dubbo:com.acme.Interface1:v1:default`

 和 `dubbo:com.acme.Interface1:v1:test` 的话，相当于 group 并无变化。同理，version 和 protocol 变化，相当于 Dubbo 服务 ID 变化，**这类情况无需处理**。



1. Dubbo 服务元数据升级

这是一种比较特殊的升级方法，即 Provider 所有服务实例 Dubbo 服务 ID 相同，然而 Dubbo 服务的参数在不同版本服务实例存在差异，假设 Dubbo Service P1 部署 5 台服务，其中 3 台服务实例设置 timeout 为 1000 ms，其余 2 台 timeout 为 3000 ms。换言之，P1 拥有两个版本（状态）的 `MetadataService` 。



综上所述，无论是 Dubbo interface 方法声明升级，还是 Dubbo 服务元数据升级，均可认为是 Dubbo 服务升级的因子，这些因子所计算出来的数值称之为“Dubbo 服务修订版本”，服务自省架构将其命名为“**revision**”。架构设设计上，当 Dubbo Service 增加或删除服务方法、修改方法签名以及调整 Dubbo 服务元数据，revision 也会随之变化，revision 数据将存放在其 Dubbo 服务实例的 metadata 中。当 Consumer 订阅 Provider Dubbo 服务元信息时，`MetadataService` 远程调用的次数取决于服务实例列表中出现 revision 的个数，整体执行流程如下图所示：

![image.png](https://cdn.nlark.com/yuque/0/2020/png/222258/1587353997685-489ac731-460d-4103-9750-0b999b8a5442.png)

**(图 9）**



1. Consumer 通过服务发现 API 向注册中心获取 Provider 服务实例列表
2. 注册中心返回 6 台服务实例，其中 revision 为 1 的服务实例为 Instance 1 到 3, revision 为 2 的服务实例是 Instance 4 和 Instance 5，revision 为 3 的服务实例仅有 Instance 6
3. Consumer 在这 6 台服务实例中随机选择一台，如图中 Instance 3
4. Consumer 向 Instance 3 发起 `MetadataService` 的远程调用，获得 Dubbo URL 列表，并建立 revision 为 1 的 URL 列表缓存，用 cache = { 1:urls(r1) } 表示
5. （重复步骤 4）Consumer 再从剩余的 5 台服务实例中随机选择一台，如图中的 Instance 5，由于 Instance 5 与 Instance 3 的 revision 分为为 2 和 1，此时缓存 cache = { 1:urls(r1) } 未命中，所以 Consumer 将再次发起远程调用，获取新的 Dubbo URL 列表，并更新缓存，即 cache = { 1:urls(r1) , 2:urls(r2) }
6. （重复步骤 4）Consumer 再从剩余的 4 台服务实例中随机选择一台，假设服务实例是 Instance 6，由于此时 revision 为3，所以缓存 cache = { 1:urls(r1) , 2:urls(r2) } 再次未命中，再次发起远程调用，并更新缓存 cache = { 1:urls(r1) , 2:urls(r2) , 3:urls(r3) }
7. （重复步骤 4）由于缓存 cache = { 1:urls(r1) , 2:urls(r2) , 3:urls(r3) } 已覆盖三个 revision 场景，如果该步骤选择服务实例落在 revision 为 1 的子集中，只需克隆 urls(r1)，并根据具体服务实例替换部分 host 和 port 等少量元信息即可，组成成新的 Dubbo URL 列表，依次类推，计算直到剩余服务实例为 0。



大多数情况，revision 的数量不会超过 2，换言之，Consumer 发起 `MetadataService` 的远程调用不会超过 2次。无论 revision 数量的大小，架构能够保证获取 Dubbo 元信息的正确性。

 

当然  `MetadataService` 并非仅支持 Dubbo URL 元数据，还有其他类型的支持。





### 元数据类型

架构上，元数据服务（`MetadataService`）未来将逐步替代 Dubbo 2.7.0 [元数据中心](http://dubbo.apache.org/zh-cn/docs/user/references/metadata/introduction.html)，并随着 Dubbo 版本的更迭，所支持的元数据类型也将有所变化，比如 Dubbo 2.7.5 元数据服务支持的类型包括：

- Dubbo 暴露的服务 URL 列表
- Dubbo 订阅的服务 URL 列表
- Dubbo 服务定义



#### Dubbo 暴露的服务 URL 列表

当前 Dubbo Service 暴露或发布 Dubbo 服务 URL 集合，如：[ `dubbo://192.168.1.2:20880/com.acme.Interface1?group=default&version=v1` , `thirft://192.168.1.2:20881/com.acme.InterfaceX` , `rest://192.168.1.2:20882/com.acme.interfaceN` ]



#### Dubbo 订阅的服务 URL 列表

当前 Dubbo Service 所有订阅的 Dubbo 服务 URL 集合，该元数据主要被 Dubbo 运维平台来收集。



#### Dubbo 服务定义

Dubbo 服务提供方（Provider）在服务暴露的过程中，将元信息以 JSON 的格式同步到注册中心，包括服务配置的全部参数，以及服务的方法信息（方法名，入参出参的格式）。在服务自省引入之前，该元数据被 Dubbo 2.7.0 [元数据中心](http://dubbo.apache.org/zh-cn/docs/user/references/metadata/introduction.html) 存储，如：

```
{
 "parameters": {
  "side": "provider",
  "methods": "sayHello",
  "dubbo": "2.0.2",
  "threads": "100",
  "interface": "org.apache.dubbo.samples.metadatareport.configcenter.api.AnnotationService",
  "threadpool": "fixed",
  "version": "1.1.1",
  "generic": "false",
  "revision": "1.1.1",
  "valid": "true",
  "application": "metadatareport-configcenter-provider",
  "default.timeout": "5000",
  "group": "d-test",
  "anyhost": "true"
 },
 "canonicalName": "org.apache.dubbo.samples.metadatareport.configcenter.api.AnnotationService",
 "codeSource": "file:/../dubbo-samples/dubbo-samples-metadata-report/dubbo-samples-metadata-report-configcenter/target/classes/",
 "methods": [{
  "name": "sayHello",
  "parameterTypes": ["java.lang.String"],
  "returnType": "java.lang.String"
 }],
 "types": [{
  "type": "java.lang.String",
  "properties": {
   "value": {
    "type": "char[]"
   },
   "hash": {
    "type": "int"
   }
  }
 }, {
  "type": "int"
 }, {
  "type": "char"
 }]
}
```



#### 更多元数据类型支持

在架构上，元数据服务（`MetadataService`）所支持元数据类型是不限制的，如下图所示：

![image.png](https://cdn.nlark.com/yuque/0/2020/png/222258/1588215019092-0921fe6f-1095-4b69-82f7-4fc05b62a0e4.png)

**(图 10）**



除上文曾讨论的三种元数据类型，还包括“Dubbo 服务 REST 元信息” 和 “其他元信息”。其中，Dubbo 服务 REST 元信息包含 Dubbo 服务 与 REST 映射信息，可用于 Dubbo 服务网关，而其他元信息可能包括 Dubbo 服务 JavaDoc 元信息，可用于 Dubbo API 文档。





### 元数据服务升级

考虑到 Dubbo Provider 和 Consumer 可能依赖不同发行版本的 `MetadataService` ，因此，Provider 提供的和 Consumer 所需要的元数据类型并不对等，如 Provider 使用 Dubbo 版本为 2.7.5，该发行版本仅支持“Dubbo 暴露的服务 URL 列表”，“Dubbo 订阅的服务 URL 列表”和“Dubbo 服务定义”，这三种元数据分别来源于接口的三个方法。当 Consumer 使用了更高的 Dubbo 版本，并需要获取“Dubbo 服务 REST 元信息”时，自然无法从 Provider 端获取。假设 `MetadataService` 为其新增一个方法，那么，当 Consumer 发起调用时，那么这个调用自然会失败。即使两端使用的版本相同，那么 Provider 仍有可能选择性支持特定的元数据类型。为了确保元数据接口的兼容性，`MetadataService` 应具备元数据类型支持的判断。如此设计，`MetadataService` 在元数据类型上支持更具有弹性。





## 事件驱动架构

相较于传统的 Dubbo 架构，服务自省架构的执行流程更为复杂，执行动作之间的关联非常紧密，如 Dubbo Service 服务实例注册前需要完成 Dubbo 服务 revision 的计算，并将其添加至服务实例的 metadata 中。又如当 Dubbo Service 服务实例出现变化时，Consumer 元数据需要重新计算。这些动作被 “事件”（`Event`）驱动，驱动者被定义为“事件分发器”（ `EventDispatcher` ），而动作的处理则由“事件监听器”（`EventListener`）执行，三者均为 “**Dubbo 事件**"的核心组件，同样由 Dubbo 2.7.5 引入。不过，Dubbo 事件是相对独立的架构，不过被服务自省中的“服务注册与发现架构”和“元数据服务架构”依赖。





### Dubbo 内建事件

Dubbo 内建事件可归纳为以下类型：

- Dubbo 服务类型事件
- Dubbo Service 类型事件
- Dubbo 服务实例类型事件
- Dubbo 服务注册和发现类型事件





#### Dubbo 服务类型事件

| 事件类型                          | 事件触发时机              |
| --------------------------------- | ------------------------- |
| `ServiceConfigExportedEvent`      | 当 Dubbo 服务暴露完成时   |
| `ServiceConfigUnexportedEvent`    | 当 Dubbo 服务下线后       |
| `ReferenceConfigInitializedEvent` | 当 Dubbo 服务引用初始化后 |
| `ReferenceConfigDestroyedEvent`   | 当 Dubbo 服务引用销毁后   |





#### Dubbo Service 类型事件

| 事件类型                             | 事件触发时机                 |
| ------------------------------------ | ---------------------------- |
| `DubboShutdownHookRegisteredEvent`   | 当 Dubbo ShutdownHook 注册后 |
| `DubboShutdownHookUnregisteredEvent` | 当 Dubbo ShutdownHook 注销后 |
| `DubboServiceDestroyedEvent`         | 当 Dubbo 进程销毁后          |





#### Dubbo 服务实例类型事件

| 事件类型                              | 事件触发时机                                 |
| ------------------------------------- | -------------------------------------------- |
| `ServiceInstancePreRegisteredEvent`   | 当 Dubbo 服务实例注册前                      |
| `ServiceInstanceRegisteredEvent`      | 当 Dubbo 服务实例注册后                      |
| `ServiceInstancePreUnregisteredEvent` | 当 Dubbo 服务实例注销前                      |
| `ServiceInstanceUnregisteredEvent`    | 当 Dubbo 服务实例注销后                      |
| `ServiceInstancesChangedEvent`        | 当 某个 Dubbo Service 下的服务实例列表变更时 |





#### Dubbo 服务注册和发现类型事件

| 事件类型                            | 事件触发时机                          |
| ----------------------------------- | ------------------------------------- |
| `ServiceDiscoveryInitializingEvent` | 当 Dubbo 服务注册与发现组件初始化中   |
| `ServiceDiscoveryInitializedEvent`  | 当 Dubbo 服务注册与发现组件初始化后   |
| `ServiceDiscoveryExceptionEvent`    | 当 Dubbo 服务注册与发现组件异常发生时 |
| `ServiceDiscoveryDestroyingEvent`   | 当 Dubbo 服务注册与发现组件销毁中     |
| `ServiceDiscoveryDestroyedEvent`    | 当 Dubbo 服务注册与发现组件销毁后     |





# 结束语





# What's the next?