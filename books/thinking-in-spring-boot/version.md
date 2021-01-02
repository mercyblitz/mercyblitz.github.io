# 《Spring Boot 编程思想》 - 版本范围

为了系统性地讨论 Spring Boot 的发展脉络，会将 Spring Boot 2.0 与 1.x 的版本加以对比，探索从 1.0 到 2.0 版本之间的重要变化，便于读者后续架构、整合以及迁移等工作，由于截止到当前编写时间，恰逢 Spring Boot `2.0.2.RELEASE` 版本发布，为统一源码分析，将讨论的 Spring Boot 2.0 版本固定在 `2.0.2.RELEASE`，相同时间点的 Spring Boot 1.5 的版本则是 `1.5.10.RELEASE`。同时由于更低的版本已停止维护，所以可选择其最后发行的小版本作为参考，故所有涉及的 Spring Boot 版本如下所示：

- Spring Boot 2.0 : `2.0.2.RELEASE`
- Spring Boot 1.5 : `1.5.10.RELEASE`
- Spring Boot 1.4 : `1.4.7.RELEASE`
- Spring Boot 1.3 : `1.3.8.RELEASE`
- Spring Boot 1.2 : `1.2.8.RELEASE`
- Spring Boot 1.1 : `1.1.9.RELEASE`
- Spring Boot 1.0 : `1.0.2.RELEASE`

由于 Spring Boot 2.0 最低依赖的 Java 版本为 8，而 Spring Boot 1.x 最低兼容 Java 1.6，因此 Java 1.8 是 Spring Boot 各个版本兼容的交集，也是后续的实例工程 `thinking-in-spring-boot-samples` 的运行时环境。除此之外，讨论将更多地关注 Spring Boot 与其核心依赖 Spring Framework 之间的版本映射关系。

| Spring Boot 版本 | Spring Framework 版本 | JDK 版本 |
| ---------------- | --------------------- | -------- |
| `2.0.2.RELEASE`  | `5.0.6.RELEASE`       | 1.8+     |
| `1.5.10.RELEASE` | `4.3.14.RELEASE`      | 1.6+     |
| `1.4.7.RELEASE`  | `4.3.9.RELEASE`       | 1.6+     |
| `1.3.8.RELEASE`  | `4.2.8.RELEASE`       | 1.6+     |
| `1.2.8.RELEASE`  | `4.1.9.RELEASE`       | 1.6+     |
| `1.1.9.RELEASE`  | `4.0.8.RELEASE`       | 1.6+     |
| `1.0.2.RELEASE`  | `4.0.3.RELEASE`       | 1.6+     |

不难看出，Spring Boot 2.0 对应的 Spring Framework 版本是 5.0，而 Spring Boot 1.x 则依赖于 Spring Framework 4.x。之所以要具体到 Spring Framework 的某个版本号，除了避免版本差异所导致源码分析失准的情况，更多的是由于 Spring 技术栈特殊的版本号管理。按照传统 Java 版本的约定，第一位数字表示主版本号，控制大版本更新，第二位则代表次版本号，可小范围地引入新的特性和相关 API，而第三位则用于问题修正或安全补丁等。而 Spring 技术栈不时地利用第三位版本号引入新的 API，典型的代表有 Spring Framework 3.0.1 引入的 API `BeanDefinitionRegistryPostProcessor`，Spring Boot 1.3.2 引入的 API `ExitCodeEvent`。同时，在框架 API 的兼容性，从 Spring Framework 到 Spring Cloud 逐渐降低，Spring Boot 处于比上不足比下有余的情况。因此，本书在深入讨论的过程中反复地强调 API 兼容的重要性，也希望读者在自研的过程中尤为关注。

为了理解 Spring Boot 特性发展的过程，将 Spring Framework 版本讨论的范围从 1.x 到 5.0。换言之，本书将几乎涵盖所有的 Spring Framework 以及 Spring Boot 版本，包括两者所涉及的 [JSR](https://jcp.org/en/jsr/overview)（Java Specification Requests），如 Servlet 、Bean Validation 和 JAX-RS 等规范。

> 小马哥提示您：更多的 JSR 资讯，请参考官方网页 [https://jcp.org/en/jsr/overview](https://jcp.org/en/jsr/overview)，或访问小马哥 JSR 收藏页面 [https://github.com/mercyblitz/jsr](https://github.com/mercyblitz/jsr)，下载归档的 JSR PDF 文件。




## [返回](/books/thinking-in-spring-boot/)