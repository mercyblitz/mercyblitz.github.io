# Spring Boot Web 应用加速



默认情况下，Spring Boot Web 应用会装配一些功能组件 Bean。


在大多数 Web 应用场景下，可以选择性地关闭一下自动装配的Spring 组件 Bean，以达到提升性能的目的。


## 配置项优化

### Spring Boot Web 应用加速 完整配置项

````properties
management.add-application-context-header = false
spring.mvc.formcontent.putfilter.enabled = false

spring.autoconfigure.exclude = org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration,\
org.springframework.boot.autoconfigure.gson.GsonAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.XADataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.JndiDataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.transaction.jta.JtaAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.WebSocketAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.WebSocketMessagingAutoConfiguration,\
org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration,\
org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.mustache.MustacheAutoConfiguration,\
org.springframework.boot.autoconfigure.mail.MailSenderAutoConfiguration,\
org.springframework.boot.autoconfigure.mail.MailSenderValidatorAutoConfiguration,\
org.springframework.boot.actuate.autoconfigure.TraceRepositoryAutoConfiguration,\
org.springframework.boot.actuate.autoconfigure.TraceWebFilterAutoConfiguration,\
org.springframework.boot.actuate.autoconfigure.MetricFilterAutoConfiguration
````

### 配置项汇总

````properties
spring.autoconfigure.exclude = org.springframework.boot.actuate.autoconfigure.TraceRepositoryAutoConfiguration,\
org.springframework.boot.actuate.autoconfigure.TraceWebFilterAutoConfiguration,\
org.springframework.boot.actuate.autoconfigure.MetricFilterAutoConfiguration
````

### 关闭 Web 请求跟踪 自动装配


#### `org.springframework.boot.actuate.autoconfigure.TraceWebFilterAutoConfiguration`


顾名思义，该自动装配用跟踪 Web 请求，通过Servlet Filter `org.springframework.boot.actuate.trace.WebRequestTraceFilter` 记录请求的信息（如：请求方法、请求头以及请求路径等），其计算的过程存在一定的开销，使用场景罕见，故可选择关闭。


* 配置项


````properties
spring.autoconfigure.exclude = org.springframework.boot.actuate.autoconfigure.TraceWebFilterAutoConfiguration
````


#### `org.springframework.boot.actuate.autoconfigure.TraceRepositoryAutoConfiguration`

当`org.springframework.boot.actuate.autoconfigure.TraceWebFilterAutoConfiguration`关闭后，其请求信息存储介质`org.springframework.boot.actuate.trace.TraceRepository`没有存在的必要，故可选择关闭。


* 配置项


````properties
spring.autoconfigure.exclude = org.springframework.boot.actuate.autoconfigure.TraceRepositoryAutoConfiguration
````


### 关闭 Web 请求结果指标 自动装配


#### `org.springframework.boot.actuate.autoconfigure.MetricFilterAutoConfiguration`


该组件将自动装配`org.springframework.boot.actuate.autoconfigure.MetricsFilter`，该 Filter

主要记录Web 请求结果指标（如：相应状态码、请求方法执行时间等），该信息一定程度上与反向代理服务器（nginx）功能重叠，故可选择关闭。


* 配置项


````properties
spring.autoconfigure.exclude = org.springframework.boot.actuate.autoconfigure.MetricFilterAutoConfiguration
````




### 可关闭 Servlet Web 组件

#### `org.springframework.web.filter.HttpPutFormContentFilter`


* 引入版本

`org.springframework.web.filter.HttpPutFormContentFilter` 由 Spring
Framework 3.1 版本引入，分发在 `org.springframework:spring-web` 中。


* 使用场景

通常 Web 场景中，浏览器通过 HTTP `GET` 或者 `POST` 请求 提交 Form 数据，而非浏览
器客户端（如应用程序）可能通过 HTTP `PUT` 请求来实现。

当 HTTP 请求头`Content-Type` 为 `application/x-www-form-urlencoded` 时
，Form 数据被 encoded。而 Servlet 规范中， `ServletRequest.getParameter*()`
方法仅对 HTTP `POST` 方法支持请求参数的获取，如：

````java
public intetfacce ServletRequest {

    ......

    public String getParameter(String name);

    public Enumeration<String> getParameterNames();

    public String[] getParameterValues(String name);

    public Map<String, String[]> getParameterMap();

    ......

}
````

故 以上方法无法支持 HTTP `PUT` 或 HTTP `PATCH` 请求方法（请求头`Content-Type`
为`application/x-www-form-urlencoded`）。

`org.springframework.web.filter.HttpPutFormContentFilter` 正是这种场景的解
决方案。

Spring Boot 默认场景下，将
`org.springframework.web.filter.HttpPutFormContentFilter` 被
`org.springframework.boot.autoconfigure.web.WebMvcAutoConfiguration` 自动
装配，以下为 Spring Boot 1.4.1.RELEASE 以及更好版本定义（可能存在一定的差异）：

````java
@Configuration
@ConditionalOnWebApplication
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class,
		WebMvcConfigurerAdapter.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class,
		ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {

    ......

    @Bean
    @ConditionalOnMissingBean(HttpPutFormContentFilter.class)
    @ConditionalOnProperty(prefix = "spring.mvc.formcontent.putfilter", name = "enabled", matchIfMissing = true)
    public OrderedHttpPutFormContentFilter httpPutFormContentFilter() {
        return new OrderedHttpPutFormContentFilter();
    }

    ......

}
````

综上所述，`org.springframework.web.filter.HttpPutFormContentFilter` 在绝大
多数 Web 使用场景下为非必须组件。


* 配置项

如果应用依赖 Spring Boot 版本 为 1.4.1.RELEASE 以及更高的版本，可通过如下配置，
进行将 `org.springframework.web.filter.HttpPutFormContentFilter` 关闭：

````properties
spring.mvc.formcontent.putfilter.enabled = false
````


#### `org.springframework.web.filter.HiddenHttpMethodFilter`


* 引入版本

`org.springframework.web.filter.HiddenHttpMethodFilter` 由 Spring
Framework 3.0 版本引入，分发在 `org.springframework:spring-web` 中。


* 使用场景

当 Web 服务端同一资源（URL）提供了多请求方法的实现，例如 URI ：/update 提供了
HTTP `POST` 以及 HTTP `PUT` 实现），通常 Web 场景中，浏览器仅支持 HTTP `GET`
或者 `POST` 请求方法，这样的话，浏览器无法发起 HTTP `PUT` 请求。

为了浏览器可以消费 HTTP `PUT` 资源， 需要在服务端将 HTTP `POST` 转化成
HTTP `PUT` 请求，为了解决这类问题，Spring 引入
`org.springframework.web.filter.HiddenHttpMethodFilter` Web 组件。

当浏览器 发起 HTTP `POST` 请求时，可通过增加请求参数（默认参数名称："_method"）
的方式，进行HTTP 请求方法切换，
`org.springframework.web.filter.HiddenHttpMethodFilter` 获取参数"_method"
值后，将参数值作为 `HttpServletRequest#getMethod()`的返回值，给后续 `Servlet`
实现使用。

出于通用性的考虑，`org.springframework.web.filter.HiddenHttpMethodFilter`
通过调用 `#setMethodParam(String)` 方法，来修改转换请求方法的参数名称。


Spring Boot 默认场景下，将
`org.springframework.web.filter.HttpPutFormContentFilter` 被
`org.springframework.boot.autoconfigure.web.WebMvcAutoConfiguration` 自动
装配，以下为 Spring Boot 1.4.1.RELEASE 以及更好版本定义（可能存在一定的差异）：

````java
@Configuration
@ConditionalOnWebApplication
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class,
		WebMvcConfigurerAdapter.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class,
		ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {

    ......

    @Bean
    @ConditionalOnMissingBean(HiddenHttpMethodFilter.class)
    public OrderedHiddenHttpMethodFilter hiddenHttpMethodFilter() {
        return new OrderedHiddenHttpMethodFilter();
    }

    ......

}
````

综上所述，`org.springframework.web.filter.HiddenHttpMethodFilter` 也是特殊
场景下所需，故可以关闭之。


* 配置项

按目前最新的 Spring Boot 1.5.2.RELEASE 版本中实现，也没有提供类似
`spring.mvc.formcontent.putfilter.enabled` 这样的配置项关闭，无法关闭。

