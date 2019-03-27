# masterSpringMVC
### 一 springbooot介绍

##### 1 SpringBoot启动配置介绍

@SpringBootApplication 实质组合了3个其他注解：

1 @Configuration  声明此类是spring的常规配置

2 @EnableAutoConfiguration  指导springboot进行自动配置

3 ComponentScan 

##### 2 springboot 初始化过程

1 初始化springmvc的dispatcherServlet

2 搭建转码过滤器，是客户端请求正确转码

3 搭建视图解析器，告诉spring去哪查找视图，以及使用哪种方言(JSP Thymeleaf)

4 配置静态资源位置

5 配置所支持的地域及资源bundle

6 配置multipart解析器 保证文件上传正常

7 将tomcat或jetty包含进来，从而能在web服务器上运行

8 建立错误页面

##### 3 SpringBoot源码简要介绍

###### 3 .1了解DispatcherServletAutoConfiguration源码

分析源码我们可以了解我们已经配置好了分发器servlet和multipart解析器

###### 3.2 视图解析器、静态资源以及区域配置

 WebMvcAutoConfiguration 

WebMvcAutoConfigurationAdapter

视图解析代码

```java
	@Bean
		@ConditionalOnMissingBean
		public InternalResourceViewResolver defaultViewResolver() {
			InternalResourceViewResolver resolver = new 							               InternalResourceViewResolver();
			resolver.setPrefix(this.mvcProperties.getView().getPrefix());
			resolver.setSuffix(this.mvcProperties.getView().getSuffix());
			return resolver;
		}
```

静态资源代码

注意：webJars 是 javascript包管理器的替代方案(npm)

1 webJars前缀的包会在类路径解析

2 静态资源在此位置 “/META-INF/resources/”“/resources/”“/static/”或“/public/”。

```java
	
private static final String[] CLASSPATH_RESOURCE_LOCATIONS = {
			"classpath:/META-INF/resources/", "classpath:/resources/",
			"classpath:/static/", "classpath:/public/" };
private static final String[] RESOURCE_LOCATIONS;

	static {
		RESOURCE_LOCATIONS = new String[CLASSPATH_RESOURCE_LOCATIONS.length
				+ SERVLET_RESOURCE_LOCATIONS.length];
		System.arraycopy(SERVLET_RESOURCE_LOCATIONS, 0, RESOURCE_LOCATIONS, 0,
				SERVLET_RESOURCE_LOCATIONS.length);
		System.arraycopy(CLASSPATH_RESOURCE_LOCATIONS, 0, RESOURCE_LOCATIONS,
				SERVLET_RESOURCE_LOCATIONS.length,                                           CLASSPATH_RESOURCE_LOCATIONS.length);
	}
```



```java
	@Override
		public void addResourceHandlers(ResourceHandlerRegistry registry) {
			if (!this.resourceProperties.isAddMappings()) {
				logger.debug("Default resource handling disabled");
				return;
			}
			Integer cachePeriod = this.resourceProperties.getCachePeriod();
			if (!registry.hasMappingForPattern("/webjars/**")) {
				customizeResourceHandlerRegistration(registry
						.addResourceHandler("/webjars/**")
						.addResourceLocations("classpath:/META-INF/resources/webjars/")
						.setCachePeriod(cachePeriod));
			}
			String staticPathPattern = this.mvcProperties.getStaticPathPattern();
			if (!registry.hasMappingForPattern(staticPathPattern)) {
				customizeResourceHandlerRegistration(
						registry.addResourceHandler(staticPathPattern)
								.addResourceLocations(
										this.resourceProperties.getStaticLocations())
								.setCachePeriod(cachePeriod));
			}
		}
```

地域解析器

```
@Bean
@ConditionalOnMissingBean
@ConditionalOnProperty(prefix = "spring.mvc", name = "locale")
public LocaleResolver localeResolver() {
   if (this.mvcProperties
         .getLocaleResolver() == WebMvcProperties.LocaleResolver.FIXED) {
      return new FixedLocaleResolver(this.mvcProperties.getLocale());
   }
   AcceptHeaderLocaleResolver localeResolver = new AcceptHeaderLocaleResolver();
   localeResolver.setDefaultLocale(this.mvcProperties.getLocale());
   return localeResolver;
}
```

###### 3.3 错误与转码配置

ErrorMvcAutoConfiguration

- 定义一个Bean，defaultErrorAttributes 通过特定属性暴露了有用的错误信息，包括状态、错误码等

- 定义BasicErrorControllerBean ，表示MVC控制器，展现错误页面
- 运行将whitelabel错误页面设置为无效 通过application.properties 中的error.whitelable.enabled=false
- 借助模板引擎提供自己错误页面，例如 erro.html ,  ErrorTemplateMissingCondition会对此进行检查

转码 HttpEncodingAutoConfiguration 使用spring提供的CharacterEncodingFilter类

###### 3.4  嵌入式Servlet容器 Tomcat配置

EmbeddedServletContainerAutoConfiguration类中配置者tomcat jetty

**Http端口**：

​	在application.properties 中定义 server.port 

​	当server.prot设为-1 禁用http 

​	设为0  随机启动端口 利于测试

**SSL配置**

**其他配置**：

SpringBoot内置3项内容：

JacksonAutoConfiguration 使用jackson进行json序列化

在HttpMessageConvertersAutoConfiguratation中，声明默认的HttpMessageConverter

在JmxAutoConfiguration中，声明JMX功能

### 二 精通MVC架构

###### 1 贫血模型介绍

面向对象的范式应用到领域对象之中，若没有则称之为贫血的领域模型

特征：

- 模型是由简单的java对象构成(POJO)，只有get和set方法
- 所有业务逻辑都是在服务层处理
- 对模型的校验在本模型外部进行，例如控制器

贫血模型是较差的实践方式

避免领域贫血途径：

- 服务层适合进行应用级别的抽象（如 事务处理），而不是业务逻辑

- 领域对象应始终处于合法状态。如通过校验器或jsr-303 校验注解，让校验过程在表单对象进行
- 将输入转换成有意义的领域对象
- 将数据层安装Respository的方式实现，例如参考Spring Data规范
- 将领域逻辑与底层持久化框架解耦
- 尽可能使用实际对象

###### 2 SpringMVC

###### 3 构件Spring Social twitter

