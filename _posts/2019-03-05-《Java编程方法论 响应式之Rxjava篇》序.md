# 《Java编程方法论 响应式之Rxjava篇》序

在《2019 一月的InfoQ 架构和设计趋势报告》[^1]中，响应式编程（Reactive Programming）和函数式（Functional Programing）仍旧编列在第一季度（Q1）的 Early Adopters（早期采纳者） 中。尽管这仅是一家之言，然而不少的开发人员逐渐意识到 Reactive 之风俨然吹起。也许您的生产系统尚未出现 Reactive 的身影，不过您可能听说过 Spring WebFlux 或 Netflix Hystrix 等开源框架。笔者曾请教过 Pivotal（Spring 母公司）布道师 Josh Long[^2]：”Spring 技术栈未来的重心是否要布局在 Reactive 之上？“。对方的答复是：”没错，Reactive 是未来趋势。“。同时，越来越多的开源项目开始签署 Reactive 宣言（The Reactive Manifesto）[^3]，并喊出 ”Web Are Reactive“ 的口号。

或许开源界的种种举动无法说服您向 Reactive 的”港湾“中停靠，不过 Java 9  Flow API[^4] 的引入，又给业界注入了一剂强心针。不难看出，无论是 Java API，还是 Java 框架均走向了 Reactive 编程模型的道路，这并非是一种巧合。

通常，人们谈到的 Reactive 可与 Reactive 编程划上等号，以”非阻塞（Non-Blocking）“和”异步（Asynchronous）“的特性并述，数据结构与操作相辅相成。Reactive 涉及函数式和并发两种编程模型，前者关注语法特性，后者强调执行效率。简言之，Reactive 编程的意图在于 ”Less Code，More Efficient“。除此之外，个人认为 Reactive 更大的价值在于统一 Java 并发编程模型，使得同步和异步的实现代码无异，同时做到 Java 编程风格与其他编程语言更好地融合，或许您也发现 Java  与 JS 在 Reactive 方面并不存在本质区别。纵观 Java 在 Reactive 编程上的发展而看，其特性更新可谓是步步为营，如履薄冰。尽管 Java 线程 API `Thread` 与 `Runnable` 就已具备异步以及非阻塞的能力，然而同步和异步编程的模式并不统一，并且理解 `Thread` API 的细节和管理线程生命周期的成本均由开发人员概括承受。虽然 Java 5 引入 J.U.C 框架（Java 并发框架）之后， `ExecutorService` 实现减轻了以上负担。不过开发人员仍需关注 `ExecutorService` 实现细节，比如怎样合理地设置线程池空间以及阻塞队列又成为新的挑战。为此，Java 7 又引入 `ForkJoinPool` API，不过此时的J.U.C 框架与 Reactive 理念仍存在距离，即使是线程安全的数据结构，也并不具备并行计算的能力，如：集合并行排序，同时操作集合的手段也相当的贫瘠，缺少类似 Map/Reduce 等操作。不过这些困难只是暂时的，终究被 Java 8 ”救赎“。`Stream` API 的出现不但具备数据操作在串行和并行间自由切换的能力，如 `sequential()` 以及 `parallel()` 方法，而且淡化了并发的特性，如 `sorted()` 方法即可能是传统排序，亦或是并行排序。相同的设计哲学也体现在 Java Reactive 实现框架中，如同书中提及的 RxJava[^5] API `io.reactivex.Observable` 。统一编程模型只是 `Stream` 其中设计目标之一，它结合 Lambda 语法特性，虽然提供了数量可观的操作方法，如 `flatMap()` 等，然而无论对比 RxJava，还是  Reactor[^6] ，`Stream` 操作方法却又相形见绌。值得一提的是，这些操作方法在 Reactive 的术语中称之为操作符（Operators）。当然框架内建的操作符的多与寡，并非判断其是否为 Reactive 实现的依据。其中决定性因素在于数据必须来源于发布方（生产者）的”推送（Push）“，而非消费端的”拉取（Pull）“。显然，`Stream`  属于消费端已就绪（Ready）的数据集合，并不存在其他数据推送源。不过 JVM 语言早期的 Reactive 定义处于模糊地带，如 RxJava  API 属于观察者模式（Observer Pattern）[^7]的扩展，而非迭代器（Iterator Pattern）模式[^8]的实现。而 Reactor 的实现则拥抱 Reactive Streams 规范[^9] ，该规范消费端对于数据的操作是被动的处理，而非主动的索。换言之，数据的到达存在着不确定性[^10]。当推送的数据无法得到消费端及时效应时，Reactive 框架必须提供背压（Backpressure）[^11]实现，确保消费端拥有”拒绝的权利（cancel）”。在此理论基础上，Reactive Streams 规范定义了一套抽象的 API，作为 Java 9 `java.util.concurrent.Flow` API 的顶层设计。不过关于操作符的部分，该规范似乎不太关心，这也是为什么 RxJava 和 Reactor 均称自身为 Reactive 扩展框架的原因，同时两者在 API 级别提供多种调度器（Schedulers）[^12]实现，适配不同并发场景提供。尽管 Reactive 定义在不同的阵营之间存在差异，援引本人在《Reactive-Programming-一种技术-各自表述》[^13]文中的总结：

> Reactive Programming 作为观察者模式（[Observer](https://en.wikipedia.org/wiki/Observer_pattern)） 的延伸，不同于传统的命令编程方式（ [Imperative programming](https://en.wikipedia.org/wiki/Imperative_programming)）同步拉取数据的方式，如迭代器模式（[Iterator](https://en.wikipedia.org/wiki/Iterator_pattern)） 。而是采用数据发布者同步或异步地推送到数据流（Data Streams）的方案。当该数据流（Data Steams）订阅者监听到传播变化时，立即作出响应动作。在实现层面上，Reactive Programming 可结合函数式编程简化面向对象语言语法的臃肿性，屏蔽并发实现的复杂细节，提供数据流的有序操作，从而达到提升代码的可读性，以及减少 Bugs 出现的目的。同时，Reactive Programming 结合背压（Backpressure）的技术解决发布端生成数据的速率高于订阅端消费的问题。

不难看出，Reactive 是一门综合的编程艺术，在实现框架的加持下，相同的代码逻辑实现同步和异步非阻塞功能，从而达到提升应用整体性能的目的。不过现实的情况或许没有那么理想，Spring 官方文档在《Web on Reactive Stack》章节中提到，"Reactive 和非阻塞通常并不是让应用运行的更快"[^14]：

> *Reactive and non-blocking generally do not make applications run faster.*

为此，JHipster[^15] 给出了一份《 Spring 5 WebFlux 性能测试报告》[^16]，其中一条结论是，”Reactive 应用并没有表现出速度提升（甚至是变得更差）“：

> *No improvement in speed was observed with our reactive apps (the Gatling results are even slightly worse).*

数月后，看似相反的结论却在DZone[^17]一篇名为《Raw Performance Numbers - Spring Boot 2 Webflux vs. Spring Boot 1》[^18]的文中出现，测试结果是 Spring Boot 2 WebFlux在高并发下响应时间更为平稳。实际上，这个测试结论有些”关公战秦琼“的味道，毕竟 Spring Boot 2.0 下的 WebFlux 和 Spring Boot 1.0 中的 Servlet  容器所使用线程模型是不同的，并且 Servlet 3.0 异步以及非阻塞特性缺省是关闭的。不过以上两篇的结论并不矛盾，前者关注于响应速度，后者则强调吞吐量，都是性能的核心指标。遗憾的是，两篇文章均未对各自的测试用例做出调优，因此以上结论都存在一定的局限性，这也是本人对 Reactive 技术能否提升性能提出质疑的地方。

如果本人是国内提出 Reactive 问题的第一人的话，那么知秋[^19]就是国内第一个解决问题的人。作为国内为数不多的 Reactive 以及 NIO 方面的专家，在技术研究上，他追求格物致知，不轻忽技术细节。在知识分享上，他可谓是知无不言，言无不尽，不仅在社交群中答疑解惑，而且录制免费视频，发布在 B 站[^20]以及 YouTube 频道[^21]，并得到 Josh Long 等大佬的推文(Twitter)。或许以上方式还不足以完整地讨论 Java Reactive 技术，知秋选择了漫长而又艰苦的著书之路，尽管他是本人的朋友，然而 ”内举不避亲“，笔者推荐给读者朋友，首先是因为这是大陆地区第一本全面解读 Java Reactive 技术的书籍，除作者的雄厚技术积累背书之外，书中的知识脉络是循序渐进的。同时，这也是一本引人深思的书，本书在导读源码的同时，也引导读者对于代码设计上的思考。再者，这又是一本知识苦旅的书，因为它涉及面较广，读者不仅需要具备一定的 Java 并发以及面向对象设计，而且需要读者付出较多的时间去反复推敲。正所谓”夫夷以近，则游者众；险以远，则至者少“[^22]，笔者希望读者在购买此书后，不轻言放弃，当您面临挑战时，那才是成长的开始。同时，也期盼读者将 Reactive 技术付之于实践，提早触碰未来。



小马哥（mercyblitz)[^23]

2019 年 3 月 5 日

[^1]: Architecture and Design InfoQ Trends Report - January 2019 - https://www.infoq.com/articles/architecture-trends-2019
[^2]: Josh (@starbuxman) is the Spring Developer Advocate at Pivotal and a Java Champion - https://spring.io/team/jlong
[^3]: The Reactive Manifesto -  https://www.reactivemanifesto.org
[^4]:  https://docs.oracle.com/javase/9/docs/api/java/util/concurrent/Flow.html
[^5]: RxJava: Reactive Extensions for the JVM - https://github.com/ReactiveX/RxJava6
[^6]:  Reactor is a fully non-blocking reactive programming foundation for the JVM, with efficient demand management (in the form of managing "backpressure"). - https://github.com/reactor/reactor-core
[^7]: Observer Pattern - http://en.wikipedia.org/wiki/observer_pattern
[^8]: Iterator Pattern - https://en.wikipedia.org/wiki/Iterator_pattern
[^9]: Reactive Streams is an initiative to provide a standard for asynchronous stream processing with non-blocking back pressure. - https://www.reactive-streams.org/
[^10]:  Handling streams of data—especially “live” data whose volume is not predetermined - https://github.com/reactive-streams/reactive-streams-jvm#goals-design-and-scope
[^11]: Backpressure is an integral part of this model in order to allow the queues which mediate between threads to be bounded. The benefits of asynchronous processing would be negated if the backpressure signals were synchronous (see also the [Reactive Manifesto](http://reactivemanifesto.org/)), therefore care has been taken to mandate fully non-blocking and asynchronous behavior of all aspects of a Reactive Streams implementation. - https://github.com/reactive-streams/reactive-streams-jvm#goals-design-and-scope
[^12]: RxJava Scheduler - http://reactivex.io/documentation/scheduler.html, Reactor Schedulers : https://projectreactor.io/docs/core/release/reference/#schedulers
[^13]: 《Reactive Programming 一种技术，各自表述》：https://mercyblitz.github.io/2018/07/25/Reactive-Programming-一种技术-各自表述
[^14]:  1.1.6. Performance - https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-performance
[^15]:JHipster -https://www.jhipster.tech/
[^16]: 《Spring 5 WebFlux: Performance tests》- https://blog.ippon.tech/spring-5-webflux-performance-tests
[^17]:  DZone : https://dzone.com
[^18]:《Raw Performance Numbers - Spring Boot 2 Webflux vs. Spring Boot 1》 - https://dzone.com/articles/raw-performance-numbers-spring-boot-2-webflux-vs-s
[^19]:  知秋，本书的作者笔名
[^20]: https://space.bilibili.com/2494318
[^21]: https://www.youtube.com/channel/UCmxVT2rkFwDX1tPpVDPJQaQ
[^22]:《游褒禅山记》，王安石
[^23]:小马哥，[Java 劝退师](https://www.douyu.com/mercyblitz)，[Apache Dubbo](https://dubbo.apache.org/) PPMC、[Spring Cloud Alibaba](https://github.com/spring-cloud-incubator/spring-cloud-alibaba) 项目架构师 - https://mercyblitz.github.io/about/