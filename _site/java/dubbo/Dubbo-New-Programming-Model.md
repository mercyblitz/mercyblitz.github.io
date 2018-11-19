# Dubbo 新编程模型



## 整体愿景



随着微服务架构的广泛地推广和实施。在 Java 生态系统中，以 Spring Boot 和 Spring Cloud 为代表的微服务框架，引入了全新的编程模型，包括注解驱动（Annotation-Driven）、外部化配置（External Configuration）以及自动装配（Auto-Configure）等。新的编程模型无需 XML 配置、简化部署、提升开发效率。



为了更好地实践微服务架构，Dubbo 从 `2.5.7` 版本开始， 针对 Spring 应用场景（包括 Spring Boot 和 Spring Cloud），新引入注解驱动（Annotation-Driven）、外部化配置（External Configuration）等编程模型。同时，新的编程模型也是即将发布的 Spring Boot Starter（[`dubbo-spring-boot-starter`](https://github.com/dubbo/dubbo-spring-boot-project)） 的基础设施。更为重要的是，从 Dubbo `2.5.8` 开始，无论传统 Spring 应用，还是 Spring Boot 应用，两者之间可以实现无缝迁移（无需任何调整）。





## [注解驱动（Annotation-Driven）](Dubbo-Annotation-Driven.md)





## [外部化配置（Externalized Configuration）](Dubbo-Externalized-Configuration.md)




