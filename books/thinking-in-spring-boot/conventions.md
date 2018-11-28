# 《Spring Boot 编程思想》 - 相关约定


本书在议题的讨论中，将在文档引用、示例代码、日志输出、源码版本、源码省略等方面做出约定。




## 文档引用约定

在文档引用方面，Spring Boot 官方文档的默认版本为 `2.0.2.RELEASE`，即网址：
[https://docs.spring.io/spring-boot/docs/2.0.2.RELEASE/reference/htmlsingle/](https://docs.spring.io/spring-boot/docs/2.0.2.RELEASE/reference/htmlsingle/)

Spring Framework 官方文档则选择 `5.0.6.RELEASE`，地址为：[https://docs.spring.io/spring/docs/5.0.6.RELEASE/spring-framework-reference/](https://docs.spring.io/spring/docs/5.0.6.RELEASE/spring-framework-reference/)

为了遵照原文，讨论的过程中将援引原文，如 Spring Boot 官方文档在 ["11.5 Creating an Executable Jar"](https://docs.spring.io/spring-boot/docs/2.0.2.RELEASE/reference/htmlsingle/#getting-started-first-application-executable-jar) 章节中介绍此等构建方式，即构建可执行 JAR，又称之为 "fat jars"

> We finish our example by creating a completely self-contained executable jar file that we could run in production. Executable jars (sometimes called “fat jars”) are archives containing your compiled classes along with all of the jar dependencies that your code needs to run.

此处的 Spring Boot 官方文档版本为 `2.0.2.RELEASE`，否则在讨论的内容中会特别说明其他版本信息。




## 示例代码约定

为了减少代码冗余和内容篇幅，因此 Java 示例代码通常会省略其 `package` 和 `import` 部分的代码，如下所示：

```java
@SpringBootApplication
public class FirstAppByGuiApplication {

    public static void main(String[] args) {
        SpringApplication.run(FirstAppByGuiApplication.class, args);
    }

    @Bean
    public RouterFunction<ServerResponse> helloWorld() {
        return route(GET("/hello-world"),
                request -> ok().body(Mono.just("Hello,World"), String.class)
        );
    }
}
```

如果该示例被重构或调整多次，其不变的代码将会被省略，仅关注变更或核心代码：

```java
@SpringBootApplication
public class FirstAppByGuiApplication {
    ...

    /**
     * {@link ApplicationRunner#run(ApplicationArguments)} 方法在
     * Spring Boot 应用启动后回调
     *
     * @param context WebServerApplicationContext
     * @return ApplicationRunner Bean
     */
    @Bean
    public ApplicationRunner runner(WebServerApplicationContext context) {
        return args -> {
            System.out.println("当前 WebServer 实现类为："
                    + context.getWebServer().getClass().getName());
        };
    }
}
```

不难发现，为了方便理解，在示例代码中会添加必要的注释加以说明。同时，如果在调整的过程中，若出现颠覆性变化的话，通常将注释无需的功能，方便后续回顾：

```java
//@Configuration
//@ComponentScan
@EnableAutoConfiguration
//@SpringBootApplication(scanBasePackages = "thinking.in.spring.boot.config")
public class FirstAppByGuiApplication {

	public static void main(String[] args) {
		SpringApplication.run(FirstAppByGuiApplication.class, args);
	}

//    /**
//     * {@link ApplicationRunner#run(ApplicationArguments)} 方法在
//     * Spring Boot 应用启动后回调
//     *
//     * @param context WebServerApplicationContext
//     * @return ApplicationRunner Bean
//     */
//    @Bean
//    public ApplicationRunner runner(WebServerApplicationContext context) {
//        return args -> {
//            System.out.println("当前 WebServer 实现类为："
//                    + context.getWebServer().getClass().getName());
//        };
//    }
}
```

同样的原则适用于 XML 文件或其他配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>thinking-in-spring-boot</groupId>
    <artifactId>first-app-by-gui</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>war</packaging>
    ...
        <!--&lt;!&ndash; Use Jetty instead &ndash;&gt;-->
        <!--<dependency>-->
            <!--<groupId>org.springframework.boot</groupId>-->
            <!--<artifactId>spring-boot-starter-jetty</artifactId>-->
        <!--</dependency>-->

        <!--&lt;!&ndash; Use Undertow instead &ndash;&gt;-->
        <!--<dependency>-->
            <!--<groupId>org.springframework.boot</groupId>-->
            <!--<artifactId>spring-boot-starter-undertow</artifactId>-->
        <!--</dependency>-->

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </dependency>
    ...
</project>
```



## 日志输出约定

为了精简示例运行时的日志输出，书中的内容并无完全与实际情况相同，主要的差异在于其中移除了重复以及相关时间等非重要信息，如

```
$ mvn spring-boot:run
(...部分内容被省略...)
[           main] o.s.w.r.f.s.s.RouterFunctionMapping      : Mapped (GET && /hello-world) -> thinkinginspringboot.firstappbygui.FirstAppByGuiApplication$$Lambda$276/708609190@7af17431
(...部分内容被省略...)
当前 WebServer 实现类为：org.springframework.boot.web.embedded.tomcat.TomcatWebServer
[           main] t.f.FirstAppByGuiApplication             : Started FirstAppByGuiApplication in 2.119 seconds (JVM running for 5.071)
```

当内容中出现 "(...部分内容被省略...)" 时，说明其中省略数行的日志内容，并且几乎所有的日志输出到标准输出（`System.out`）。




## 源码版本约定

在源码分析过程中，为了厘清不同版本中的实现细节和变化，通常在目标代码下方带有 Spring Boot 版本信息，以及 Maven 依赖的 GAV 坐标信息（GAV = `groupId`、`artifactId` 以及 `version`)，如下所示：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Configuration
@EnableAutoConfiguration
@ComponentScan
public @interface SpringBootApplication {

	/**
	 * Exclude specific auto-configuration classes such that they will never be applied.
	 * @return the classes to exclude
	 */
	Class<?>[] exclude() default {};

}
```
> 以上实现源码源于 Spring Boot `1.2.8.RELEASE`
> > Maven GAV 坐标为： `org.springframework.boot:spring-boot-autoconfigure:1.2.8.RELEASE`

如果以上版本信息没有出现时，Spring Boot 默认采用 `2.0.2.RELEASE`，Spring Framework 则选择 `5.0.6.RELEASE`，JDK 源码的版本则是 `1.8.0_172`。




## 源码省略约定

在源码分析的过程中，考虑到实现代码可能相对繁杂，为观其大意，便于记忆，期间将会注释或移除部分无关痛痒的内容，如：

```java
public class AutoConfigurationImportSelector
		implements DeferredImportSelector, BeanClassLoaderAware, ResourceLoaderAware,
		BeanFactoryAware, EnvironmentAware, Ordered {
    ...
    protected List<String> getCandidateConfigurations(AnnotationMetadata metadata,
			AnnotationAttributes attributes) {
		List<String> configurations = SpringFactoriesLoader.loadFactoryNames(
				getSpringFactoriesLoaderFactoryClass(), getBeanClassLoader());
		...
		return configurations;
	}
    ...
	protected Class<?> getSpringFactoriesLoaderFactoryClass() {
		return EnableAutoConfiguration.class;
	}
    ...
}
```

当然，其省略的部分并未一无是处，而是根据小马哥个人经验来筛选，必然存在个人主观的影响，建议读者先结合对应的版本源码，整体把握，逐步形成选读的意识。




## 表达约定

本书的讨论内容可能对相同事务出现不同的表述方式，如：

- 注解：Annotation

- 配置 Class：`@Configuration` 类、`@Configuration` Class、 Configuration Class

- 包：`package`

- 类路径：Class Path、class-path、类路径