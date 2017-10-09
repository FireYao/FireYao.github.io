---
title: 丢掉xml使用JavaConfig配置Spring
date: 2017-10-09 15:30:49
categories:
- spring
tags: [spring]
toc: true
reward: false
---

### Spring JavaConfig

最近撸了一遍**Spring action 4**,发现里面讲的都不再使用xml文件来配置spring,全都采用Java代码来配置.

用Java代码配置的话,感觉要比xml更便于维护,而且用代码肯定比xml更爽嘛

下面来一步步用JavaConfig搭一个Spring工程

那在用xml配置的时候,项目都是从加载web.xml文件再扫描到各种spring-*.xml文件

那不用xml文件,项目从哪里启动呢?

那就要靠这个类了,`AbstractAnnotationConfigDispatcherServletInitializer`,这个就相当于web.xml啦,在这里面可以配置上下文,DispatcherServlet,过滤器等等bean;

首先咱先创建一个类`SpittrWebAppInitialzer`

<!-- more -->

```java
package com.fireyao;

import org.springframework.web.filter.CharacterEncodingFilter;
import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

import javax.servlet.Filter;

public class SpittrWebAppInitialzer extends AbstractAnnotationConfigDispatcherServletInitializer {
    /**
     * 配置root上下文,如Jpa数据源等等的配置
     * @return
     */
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{RootConfig.class};
    }

    /**
     * 配置dispatcher servlet
     * @return
     */
    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{WebConfig.class};
    }

    /**
     * 将DispatcherServlet映射到 "/"
     * 指定开始被servlet处理的url,配置从/开始  
     * @return
     */
    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }

    /**
     * 这里注册的所有过滤器,都会映射到DispatcherServlet
     * 就是说这里的过滤器过滤规则是 /*
     * 所有的请求都会先到这里注册的过滤器中
     *
     * @return
     */
    @Override
    protected Filter[] getServletFilters() {
        return new Filter[]{
                new CharacterEncodingFilter("UTF-8", true)
        };
    }
}
```

在`SpittrWebAppInitialzer`类里面加载了`RootConfig`和`WebConfig`两个配置类,

再创建这两个类以及相关的配置(以下省略package和import)

`RootConfig`

```java
/**
 * 相当于applicationContext.xml
 */
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(basePackages = {"com.fireyao.repository"},
        entityManagerFactoryRef = "entityManagerFactory",
        transactionManagerRef = "transactionManager")
@PropertySource(value = {"classpath:db.properties", "classpath:hibernate.properties", "classpath:app.properties"})
@ComponentScan(basePackages = "com.fireyao",
        excludeFilters = {
                @ComponentScan.Filter(
                        type = FilterType.ANNOTATION, value = EnableWebMvc.class
                )})
@EnableAspectJAutoProxy(proxyTargetClass = true)
/**
 *   proxyTargetClass = true ==> 使用cglib代理
 *   proxyTargetClass = false(默认) ==> 使用JDK代理
 */
public class RootConfig {


    @Value(value = "${db.driver:org.postgresql.Driver}")
    private String DRIVERCLASSNAME;

    @Value("${db.username}")
    private String USERNAME;

    @Value("${db.password}")
    private String PASSWORD;

    @Value("${db.jdbcURL}")
    private String URL;

    @Value("${hibernate.hbm2dll.create_namespaces}")
    private String CREATE_NAMESPACES;

    @Value("${hibernate.hbm2ddl.auto}")
    private String HBM2DDL_AUTO;

    @Value("${hibernate.show_sql}")
    private String SHOW_SQL;

    @Value("${hibernate.format_sql}")
    private String FORMAT_SQL;

    @Value("${hibernate.generate_statistics}")
    private String GENERATE_STATISTICS;
	
  /**
   * 配置数据源
   */
    @Bean(name = "dataSource")
    public DruidDataSource dataSource() {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName(DRIVERCLASSNAME);
        dataSource.setUrl(URL);
        dataSource.setUsername(USERNAME);
        dataSource.setPassword(PASSWORD);

        /*  配置初始化大小、最小、最大*/
        dataSource.setInitialSize(5);
        dataSource.setMinIdle(5);
        dataSource.setMaxActive(20);
        /* 配置获取连接等待超时的时间*/
        dataSource.setMaxWait(30000);
        /*配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒*/
        dataSource.setTimeBetweenEvictionRunsMillis(60000);
        /*配置一个连接在池中最小生存的时间，单位是毫秒*/
        dataSource.setMinEvictableIdleTimeMillis(300000);
        /*申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunsMillis，执行validationQuery检测连接是否有效*/
        dataSource.setTestWhileIdle(true);
        dataSource.setValidationQuery("select 1");
        return dataSource;
    }

    @Bean
    public HibernateJpaVendorAdapter hibernateJpaVendorAdapter() {
        return new HibernateJpaVendorAdapter();
    }
	
    @Bean(name = "entityManagerFactory")
    public LocalContainerEntityManagerFactoryBean entityManagerFactory(DruidDataSource dataSource,HibernateJpaVendorAdapter hibernateJpaVendorAdapter) {
        LocalContainerEntityManagerFactoryBean entityManagerFactory = new LocalContainerEntityManagerFactoryBean();
        entityManagerFactory.setDataSource(dataSource);
        entityManagerFactory.setJpaVendorAdapter(hibernateJpaVendorAdapter);
        entityManagerFactory.setPackagesToScan("com.fireyao.domain");

        /*指定JPA属性；如Hibernate中指定是否显示SQL的是否显示、方言等*/
        Map<String, Object> jpaProp = new HashMap();
        jpaProp.put("hibernate.dialect", new PostgisDialect());
        jpaProp.put("hibernate.hbm2ddl.auto", HBM2DDL_AUTO);
        jpaProp.put("hibernate.show_sql", SHOW_SQL);
        jpaProp.put("hibernate.generate_statistics", GENERATE_STATISTICS);
        jpaProp.put("hibernate.format_sql", FORMAT_SQL);
        jpaProp.put("hibernate.hbm2dll.create_namespaces", CREATE_NAMESPACES);
        entityManagerFactory.setJpaPropertyMap(jpaProp);
        return entityManagerFactory;
    }

    /**
     * 事务管理器
     *
     * @param entityManagerFactory
     * @return
     */
    @Bean(name = "transactionManager")
    public JpaTransactionManager transactionManager(LocalContainerEntityManagerFactoryBean entityManagerFactory) {
        return new JpaTransactionManager(entityManagerFactory.getObject());
    }
}
```

`WebConfig`

```java
/**
 * 相当于springmvc-servlet.xml
 */
@Configuration
@EnableWebMvc//启用spring mvc
@ComponentScan(basePackages = "com.fireyao.controller") //启用组件扫描
public class WebConfig extends WebMvcConfigurerAdapter {


    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        FastJsonHttpMessageConverter fastConverter = new FastJsonHttpMessageConverter();
        FastJsonConfig fastJsonConfig = new FastJsonConfig();
        fastJsonConfig.setSerializerFeatures(SerializerFeature.PrettyFormat);
        //处理中文乱码问题
        List<MediaType> fastMediaTypes = new ArrayList<>();
        fastMediaTypes.add(MediaType.APPLICATION_JSON_UTF8);
        fastConverter.setSupportedMediaTypes(fastMediaTypes);
        fastConverter.setFastJsonConfig(fastJsonConfig);
        converters.add(fastConverter);
    }


    /**
     * Thymeleaf视图解析器
     *
     * @param springTemplateEngine
     * @return
     */
    @Bean
    public ThymeleafViewResolver viewResolver(SpringTemplateEngine springTemplateEngine) {
        ThymeleafViewResolver viewResolver = new ThymeleafViewResolver();
        viewResolver.setTemplateEngine(springTemplateEngine);
        viewResolver.setCharacterEncoding("utf-8");
        return viewResolver;
    }

    /**
     * 模版引擎
     *
     * @param iTemplateResolver
     * @return
     */
    @Bean
    public SpringTemplateEngine templateEngine(ITemplateResolver iTemplateResolver) {
        SpringTemplateEngine templateEngine = new SpringTemplateEngine();
        templateEngine.setTemplateResolver(iTemplateResolver);
        return templateEngine;
    }

    /**
     * Thymeleaf3.0之后
     * Thymeleaf模版解析器
     *
     * @return
     */
    @Bean
    public ITemplateResolver iTemplateResolver() {
        SpringResourceTemplateResolver templateResolver = new SpringResourceTemplateResolver();
        templateResolver.setTemplateMode("HTML5");
        templateResolver.setPrefix("/WEB-INF/templates/");
        templateResolver.setSuffix(".html");
        templateResolver.setTemplateMode("HTML");
        templateResolver.setCharacterEncoding("utf-8");

        templateResolver.setCacheable(false);
        return templateResolver;
    }

    /**
     * Thymeleaf3.0之前
     * Thymeleaf模版解析器
     * @return
     */
  /*  @Bean
    public TemplateResolver templateResolver() {
        TemplateResolver resolver = new ServletContextTemplateResolver();
        resolver.setPrefix("/WEB-INF/VIEWS/");
        resolver.setSuffix(".html");
        resolver.setTemplateMode("HTML5");
        resolver.setCacheable(false);
        return resolver;
    }*/


    /**
     * 配置静态资源的处理
     * 要求DispatcherServlet将对静态资源的请求转发到Servlet容器中默认的Servlet上
     * 而不是使用DispatcherServlet本身来处理此类请求。
     *
     * @param configurer
     */
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    /**
     * 配置视图解析器
     * ==> JSP视图
     *
     * @return
     */
    /*@Bean
    public ViewResolver viewResolver() {
        InternalResourceViewResolver resolver = new InternalResourceViewResolver();
        resolver.setPrefix("/WEB-INF/views/");
        resolver.setSuffix(".jsp");
        resolver.setExposeContextBeansAsAttributes(true);
        return resolver;
    }*/
}
```

`@Configuration` 标注为配置类

`@EnableTransactionManagement`注解开启注解式事务的支持。

`@EnableJpaRepositories `注解开启对Spring Data JPA Repostory的支持

`@PropertySource` 扫面db.properties等配置文件,可以用`@Value`注解取到properties中的值

`@ComponentScan` 配置扫描类包 相当于`<context:component-scan base-package="com.fireyao"/>`

`@EnableAspectJAutoProxy` 表示开启AOP代理自动配置

> @EnableAspectJAutoProxy中proxyTargetClass属性
> ​	proxyTargetClass = true ==> 使用cglib代理
> ​	proxyTargetClass = false(默认) ==> 使用JDK代理

那spring最基本的JavaConfig就这样了.

是不是看上去很舒服,果然还是要用Java代码才爽.

maven依赖就不贴出来了.

附上源码地址 [FireYao/springAction4](https://github.com/FireYao/springAction4)

