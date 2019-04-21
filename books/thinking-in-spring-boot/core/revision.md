# 《Spring Boot 编程思想 - 核心篇》- 勘误汇总

如果读者在阅读《Spring Boot 编程思想 - 核心篇》或示例练习的过程中发现了其中错误，请将错误内容提交至 https://github.com/mercyblitz/thinking-in-spring-boot-samples/issues，小马哥将勘误内容汇总到此，修正后的内容将在后续的书籍发行中体现。



书名：《Spring Boot 编程思想 - 核心篇》
ISBN：978-7-121-36039-8



## 版次：2019 年 3 月第 1 版

最新更新时间：**2019-04-21** 



| 位置           | 错误描述                                                     | 修正描述                                       | 类型 | 贡献者                                              | 来源                                                         | 修正版次 |
| -------------- | ------------------------------------------------------------ | ---------------------------------------------- | ---- | --------------------------------------------------- | ------------------------------------------------------------ | -------- |
| 21页第1行      | “8080/HTTP”，缺少空格                                        | “8080/ HTTP”                                   | 排版 | [Cyric-Cao](https://github.com/Cyric-Cao)           | [#7](https://github.com/mercyblitz/thinking-in-spring-boot-samples/issues/7) |          |
| 39页第2行      | “org.springframe work.boot” 多出空格                         | 移除空格                                       | 排版 | [stackfing](https://github.com/stackfing)           | [#3](https://github.com/mercyblitz/thinking-in-spring-boot-samples/issues/3) |          |
| 40页第1行      | “org.springframework.boot.Spring Application”                | org.springframework.boot.SpringApplication     | 排版 | [Yuhuiyang-Dev](https://github.com/Yuhuiyang-Dev)   | [#3](https://github.com/mercyblitz/thinking-in-spring-boot-samples/issues/3) |          |
| 43页第7行      | “故符合xxxx.startWith(BOOT_INF_LIB)的判断”                   | “故符合xxxx.startWith(BOOT_INF_CLASSES)的判断” | 描述 | [nosqlcoco](https://github.com/nosqlcoco)           | [#3](https://github.com/mercyblitz/thinking-in-spring-boot-samples/issues/3) |          |
| 42页第4行      | “在IDEA中同时按下`command+O`”                                | `command+O` 调整为 `command+n`                 | 描述 | [Yuhuiyang-Dev](https://github.com/Yuhuiyang-Dev)   | [#3](https://github.com/mercyblitz/thinking-in-spring-boot-samples/issues/3) |          |
| 73页第6行      | “类似于Spring Boot FAR JAR”                                  | 应调整为“修正Spring Boot FAT JAR”              | 拼写 | [xkcoding](https://github.com/xkcoding)             | [#3](https://github.com/mercyblitz/thinking-in-spring-boot-samples/issues/3#issuecomment-485229241) |          |
| 91页第1行      | “当前 WebServer 实现类为 org.springframework.boot.web.embedded.undertow.UndertowWebServer” | 应纳入控制台输出部分，而非正文                 | 排版 | [xkcoding](https://github.com/xkcoding)             | [#8](https://github.com/mercyblitz/thinking-in-spring-boot-samples/issues/8#issue-435356782) |          |
| 92页倒数第2段  | “当前 WebServer 实现类为 org.springframework.boot.web.embedded.undertow.UndertowWebServer”不应出现在正文部分 | 应纳入控制台输出部分，而非正文                 | 排版 | [xkcoding](https://github.com/xkcoding)             | [#8](https://github.com/mercyblitz/thinking-in-spring-boot-samples/issues/8#issuecomment-485058837) |          |
| 93页第2行      | “替换默认的 Jetty Web Server”                                | 此处应为“默认的 Netty Web Server”              | 描述 | [liaozan](https://github.com/liaozan)               | [#3](https://github.com/mercyblitz/thinking-in-spring-boot-samples/issues/3#issuecomment-485140736) |          |
| 93页引文       | 出现“spring-boot-star ter-tomcat”的单词starter分开和“spring-boot-starter-undertow” 中的unde rtow单词分开 | 移除空格                                       | 排版 | [zhengjiangming](https://github.com/zhengjiangming) | [#3](https://github.com/mercyblitz/thinking-in-spring-boot-samples/issues/3#issuecomment-483940970) |          |
| 94页正文第1段  | “当前 WebServer 实现类为 org.springframework.boot.web.embedded.jetty.JettyWebServer”不应出现在正文部分 | 应纳入控制台输出部分，而非正文                 | 排版 | [xkcoding](https://github.com/xkcoding)             | [#8](https://github.com/mercyblitz/thinking-in-spring-boot-samples/issues/8#issuecomment-485058940) |          |
| 95页正文第2段  | “当前 WebServer 实现类为 org.springframework.boot.web.embedded.tomcat.TomcatWebServer”不应出现在正文部分 | 应纳入控制台输出部分，而非正文                 | 排版 | [xkcoding](https://github.com/xkcoding)             | [#8](https://github.com/mercyblitz/thinking-in-spring-boot-samples/issues/8#issuecomment-485058976) |          |
| 98页第2行      | “A single @Springboot Application” 一个注解被分开2个词了     | 移除中间的空格                                 | 排版 | [porscheYong](https://github.com/porscheYong)       | [#3](https://github.com/mercyblitz/thinking-in-spring-boot-samples/issues/3#issuecomment-485218187) |          |
| 103页正文第2段 | “尽管 @CompoentScan 仅关注于 @Component”,@CompoentScan 拼写错误 | “@CompoentScan” 修改为“@ComponentScan”         | 拼写 | [xkcoding](https://github.com/xkcoding)             | [#9](https://github.com/mercyblitz/thinking-in-spring-boot-samples/issues/9) |          |
| 106页正文第2段 | “当前 WebServer 实现类为 org.springframework.boot.web.embedded.tomcat.TomcatWebServer”不应出现在正文部分 | 应纳入控制台输出部分，而非正文                 | 排版 | [xkcoding](https://github.com/xkcoding)             | [#8](https://github.com/mercyblitz/thinking-in-spring-boot-samples/issues/8#issuecomment-485059764) |          |
| 110页倒数第3行 | “其Bean对象的行文”描述错误                                   | 调整为“其Bean对象的行为”                       | 描述 | [alonecong](https://github.com/alonecong)           | [#3](https://github.com/mercyblitz/thinking-in-spring-boot-samples/issues/3#issuecomment-484747560) |          |
| 112页正文11行  | “所谓GLIB提升并非是@bean对象提供的”                          | “GLIB”调整为“CGLIB”                            | 拼写 | [bilaisheng](https://github.com/bilaisheng)         | [#5](https://github.com/mercyblitz/thinking-in-spring-boot-samples/issues/5) |          |
| 113页正文第2行 | “这些“starer”提供自动装配”                                   | “starer”调整为“starter”                        | 拼写 | [bilaisheng](https://github.com/bilaisheng)         | [#6](https://github.com/mercyblitz/thinking-in-spring-boot-samples/issues/6) |          |
|                |                                                              |                                                |      |                                                     |                                                              |          |





## [返回](/books/thinking-in-spring-boot/)