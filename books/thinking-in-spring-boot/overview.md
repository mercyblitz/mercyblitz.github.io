# 《Spring Boot 编程思想》 - 内容总览

由于本书所讨论的内容跨度广泛，功能特性鲜明，由《核心篇》、《运维篇》以及《Web 篇》三册分别讨论之。《核心篇》开篇总览 Spring Boot 核心特性，逐一讨论 [Spring Boot 官网](https://spring.io/projects/spring-boot) 所罗列之六大特性，然而其中两点并非 Spring Boot 专属，故点到为止，而将其他特性归纳为五大议题，分别为 “自动装配（Auto-Configuration）”、`SpringApplication`、“外部化配置”、“Spring Boot Actuator” 以及 “嵌入式 Web 容器”。其中，前两者是《核心篇》讨论的议题，后两者则是 Spring Boot 官方定义的 "Production-Ready" 特性，均偏向 Spring Boot 应用运维，因此纳入《运维篇》的讨论范畴。至于“嵌入式 Web 容器”，它将结合传统 Java EE Servlet、Spring Web MVC 以及 Spring 5 WebFlux 内容放至《Web 篇》探讨，具体议题安排如下：

- 《核心卷》
    - 总览 Spring Boot
    - 走向自动装配
    - 理解 `SpringApplication`

- 《运维卷》
    - 超越外部化配置
    - 简化 Spring 应用运维体系

- 《Web 卷》
    - 渐行渐远的 Servlet
    - 从 Servlet 到 Web MVC
    - 从 Reactive 到 WebFlux
    - 嵌入式 Web 容器

在《核心卷》中，“走向自动装配”是以 Spring Boot 自动装配特性为核心议题，从 Spring Framework 时代展开讨论。从 Spring Framework 1.x 伊始，Spring 注解驱动编程模型逐渐发展成熟，包括 Spring ”元注解(Meta-Annotations) “、”模式注解(Stereotype Annotations) “，”组合注解（Composed Annotations）“以及”注解属性别名和覆盖（Attribute Aliases and Overrides）“。与此同时，Spring 注解驱动也形成了自成一派的设计模式，可总结为 “`@Enable`  模块驱动“，”Web 自动装配“和”条件装配“。拥有强大的 Spring 注解驱动的加持，Spring Boot 自动装配的出现不过是顺水推舟的事情。因此，它的形成过程自然是讨论的焦点。或许，自动装配是 Spring Boot 最新显著的特性，然而这并非核心特性的全部。事实上，Spring Framework 在 Spring 应用上下文生命周期的设计上存在一些的”瑕疵“，如 Spring 事件/监听时机较晚、`PropertySource` 扩展不便等。Spring Boot 作为Spring Framework 延伸，引入了全新 `SpringAppplication` API。从此，Spring Boot 具备了管理 Spring 应用上下文生命周期的能力，进而衍生出 Spring Boot 事件（Spring 事件扩展）以及外部化配置（`PropertySource` 扩展）等全新特性。所以，”理解 `SpringAppplication`“ 部分的技术细节成为了《核心卷》结尾深入探讨的议题。

目前，《核心卷》和《运维卷》已编写完毕，《Web 卷》正在同步更新中，内容安排可能发生变更，请读者以最终发行为准。

在内容结构上，本书采用“总分总”的方式，首先总体介绍讨论范围，随后深入展开细节的讨论，最后予以总结。同时，为了避免先入为主的影响，本书将会针对官方文档的描述内容提出疑问或假设，大胆地猜测其可能实现的方式，再结合实现源码加以验证，随后将通过示例代码巩固理解。在写作手法上，本书效仿传统中国历史书籍的编著手法，将纪传体和编年体予以综合。如果从功能特性来看，它属于纪传体，如自动装配、`SpringApplication`，以及外部化配置等。如此表述的方式更容易系统地掌握 Spring Boot 以及 Spring Framework 的核心特性。如果从特性的发展历程来观察，它则属于编年体，如 Spring Framework 注解驱动编程模型从 1.x 到 5.0 中的发展与 Spring Boot 自动装配之间的关联，以及 Spring Boot 1.0 到 1.4 的外部化配置源是怎样利用 Spring Environment 抽象逐步完善等。更为重要的是，在论述方式上，增加了论点、论证以及论据，从而知其然知其所以然。在特性的讨论过程中，补充说明的内容可能会穿插其中（“小马哥提示您”将在出版物中删除）。在特性讨论的结尾处，“小马哥有话说”将总结所论议题，并且发表感想，类似于《史记》中的“太史公曰”。

所谓 ”兼听则明，偏听则暗”，本书的讨论范围并不会局限在 Spring Boot 或 Spring Framework，也将 Spring Cloud 甚至是 Spring Cloud Data Flow 纳入参考，探讨 Spring Boot 在两者中的运用。站在更为宏观的角度，在整个 Java EE 的生态中，Spring 技术栈并非独此一家，也不完全是“开山之作”，不少相关的特性可在 JSR 规范以及其他 Java EE 实现中找到原型。换言之，Spring 技术栈可认为是一种非常成功的“重复发明轮子”，不仅适配了 JSR 实现，而且“借鉴”了他山之石，逐步实现了自身的生态系统。

总而言之，全书的讨论将以 Spring Boot 为中心，议题发散至 Spring 技术栈、JSR 以及 Java。希望透过全局的视角，了解变迁的历程；经过多方的比较，理解特性的原理；综合标准的规范，掌握设计的哲学。当您纵览全书之后，或许将会明白为什么说 “Spring Boot 易学难精“，因为它的核心是 Spring Framework，而后者的理解程度又取决于 JSR 规范以及 Java 的熟悉度。或许 Spring 技术栈不算一流的技术，却是一等的成功。



## [返回](/books/thinking-in-spring-boot/)