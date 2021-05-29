---
title: springboot
date: 2021-05-15 15:25:30
tags: [学习,spring]
---

## springboot简介

 约定大于配置 : 按约定编程,是一种软件设计范式,目的是减少开发人员减少做决定的数量(就是规定制定文件放在什么位置).

 springboot中的约定有 : Maven的目录结构.默认resource文件夹存放配置文件.默认生成的编译后的代码都在target文件夹下.springboot默认的配置文件为:application开头等等一些.

 含有丰富的start,拿来即用.

## 快速上手

 在springboot项目中,加入打包插件后打成jar包 ,在 cmd界面启动jar包 命令: java -jar [文件名.jar]

 修改配置或者写自己的配置:首先创建一个application.properties或者application.yml文件(约定大于配置),比如修改端口号 server.port=xxxx; 增加虚拟路径: server.servlet.context-path= url

## 微服务和分布式的区别

 微服务: 是一种架构放个,一个应用可以拆分为一组小型服务,每个服务都可以运行在自己的进程内,也就是可独立部署和升级.服务间使用轻量级http(推荐)交互

 分布式: 将一个大的系统分成多个模块,,业务模块部署到不同的机器上,各个业务模块之间通过接口来进行交互,
​ 分布式容易出现的问题: 远程调用,服务发现,负载均衡,服务容错, 配置管理,服务监控,链路追踪,日志管理,任务调度,事务管理.

 集群: 就是很多相同的服务,一个或者多台服务器部署相同应用构成一个集群.

 微服务重在解耦合,使得每个模块都独立,分布式重在资源的共享和加快计算机的响应速度.一个是分散能力,一个是分散压力.

## springboot注解

 默认的扫描规则: 在启动类(`@SpringbootApplication`)所在包及子包是默认扫描的,
​ 想要改变扫描路径.可以手动修改,在`@SpringbootApplication(scanBasePackages="xxx")`就行,或者`@ComponentScan` (包扫描) 指定扫描路径.

 `@SpringBootApplication`等同于`@SpringBootConfiguration` + `@EnableAutoConfiguration` + `@ComponentScan`

 `@Configuration` 表示这个一个配置类 ,相当于spring中的配置文件。在springboot中,proxyBeanMethods是默认=true的(proxyBeanMethods: 是不是代理类的方法,如果是true,那么就是代理对象调用方法,去检查容器中,是否有该组件,有就从容器中拿.如果是false,那么就不是代理对象了,就是每次都是新new一个对象)
​ spring的配置模式FULL 和 LITE 模式 : proxyBeanMethods=true为FULL 全局配置模式,LITE就是轻量级模式.用来管理组件依赖问题.
​ LITE的优点是 在springboot项目启动的时候,不会去检查那些组件在没在容器中,启动更快.使用情况,如果只是在容器中注册组件,而不依赖其他的组件,别的也不依赖这个组件,建议用false

 `@Bean` 在配置类中标注在方法的时候，返回一个name为方法名类型为返回值的组件，默认是单例的。

 `@Import` 位置: 只要在容器中的组件上就行(要能扫描到). 此注解中的参数为一个Class的数组,可以自动的在容器中调用指定的无参构造方法,创建出组件类型的对象.默认组件的名字就是全类名.

 `@ConditionalOnxxx`条件装配: 满足他的条件的时候则进行组件的注入

 `@ImportResource` 导入一个资源文件

 `@ConfigurationProperties` 配置绑定, 位置: 在此类标注`@Component`或在配置类上`@EnableConfigurationProperties` 指定该类才能使用此注解,其中属性 prefix,表示与配置文件的哪 个前缀进行绑定

 `@EnableConfigurationProperties` 开启属性配置功能,位置 : 在配置类上. 此注解的属性为Class,指的是开启这个类的配置配置功能,还可以把该类自动注入到容器中.

## springboot自动配置

 每项的配置都有着默认值,配置文件的值最终会绑定在对应的类上,容器启动时,就会把这些配置文件对应的值赋值到对应的类上(一般是 xxxxxproperties)

 springboot的自动配置是按需加载所有的自动配置项. 导入对应的starter才会导入对应的配置类.所有的自动配置来自`springboot-boot-autoconfigure`包中
![img](springboot/OH@P]I3%KDUXH}H54%{HUUN.png)

 要先分析springboot,首先要从他的启动类注解`@SpringBootApplication`开始

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class})})
public @interface SpringBootApplication{}
```

 1 `@SpringBootConfiguration`

 在此注解中,除开元注解就只有`@Configuration`,代表当前是一个配置类

 2 `@ComponentScan` 指定包扫描

```java
 3 `@EnableAutoConfiguration`   中描述的是:
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {}
```

 3.1 `@AutoConfigurationPackage` , 此注解中就是包含一个`@Import({Registrar.class}` 利用Register来给容器导入一系列组件,对于Register来说里面的代码为:

```java
static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {
    Registrar() {
    }

    public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
        //去批量注册一些组件
        AutoConfigurationPackages.register(registry, (String[])(new AutoConfigurationPackages.PackageImports(metadata)).getPackageNames().toArray(new String[0]));
    }

    public Set<Object> determineImports(AnnotationMetadata metadata) {
        return Collections.singleton(new AutoConfigurationPackages.PackageImports(metadata));
    }
}
```

 对于register的方法就是, 得到包名,然后封装到一个数组中,就是表示把这个包下的组件全部添加到容器之中。这就是为什么将是main方法所在的包及其子包的组件注册到容器中的原因了。

 3.2 `@Import({AutoConfigurationImportSelector.class})`对于AutoConfigurationImportSelector类来说中有一个selectImports（）方法：

```java
public String[] selectImports(AnnotationMetadata annotationMetadata) {
    if (!this.isEnabled(annotationMetadata)) {
        return NO_IMPORTS;
    } else {
        AutoConfigurationImportSelector.AutoConfigurationEntry autoConfigurationEntry = this.getAutoConfigurationEntry(annotationMetadata);
        return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
    }
}
```

 在上面的代码中，需要了解的就是getAutoConfigurationEntry(annotationMetadata) 给容器中怎么批量导入组件:

```java
protected AutoConfigurationImportSelector.AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
        if (!this.isEnabled(annotationMetadata)) {
            return EMPTY_ENTRY;
        } else {
            AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
            List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
            configurations = this.removeDuplicates(configurations);
            Set<String> exclusions = this.getExclusions(annotationMetadata, attributes);
            this.checkExcludedClasses(configurations, exclusions);
            configurations.removeAll(exclusions);
            configurations = this.getConfigurationClassFilter().filter(configurations);
            this.fireAutoConfigurationImportEvents(configurations, exclusions);
            return new AutoConfigurationImportSelector.AutoConfigurationEntry(configurations, exclusions);
        }
    }
```



 以上代码中 重要的是`List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);  configurations = this.removeDuplicates(configurations)` 下面的就是对于结果进行排除和删除的筛选操作. 此代码就是获取所有需要导入到容器中的组件.
​ 在此方法中的实现:

```
List<String> configurations = SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());
```



 利用工厂加载 `private static Map<String, List<String>> loadSpringFactories(ClassLoader classLoader)`得到所有的组件.分析 loadSpringFactories 方法

```java
//先查找一个资源文件 资源文件的位置META-INF/spring.factories
//在spring-boot-AutoConfigure的jar中的spring.factories文件下,有着大量的xxxAutoConfiguration,全场景的自动配置都在其中,说明springboot启动时,就要给容器中加载的所有配置类
```



由于并不是所有的组件都是我们必须的,所以springboot就有按需开启自动配置项.在很多类的注解有`@Conditionalxxx`,所以要满足对应的条件才能生效

 在某些方法上会同时标注`@ConditionalOnBean(xxx.class)` 和 `@ConditionalMissingBean(name=xxx)` 这这种情况. 第一个是容器中有该类型的组件,第二个是没有这个名字的组件,比如:

```java
@Bean
        @ConditionalOnBean({MultipartResolver.class})
        @ConditionalOnMissingBean( name = {"multipartResolver"})
        public MultipartResolver multipartResolver(MultipartResolver resolver) {
            return resolver;}}
```

 目的就是防止你乱命名~~.

 总结就是: springboot会先加载所有的自动配置类(xxxAutoConfiguration)导入组件,组件的值在xxxxproperties中拿值,xxxxproperties从配置文件中获取值.有一系列定制化配置,在springboot启动的组件中,主要是先以用户的为主,用户没有配置的,springboot会再根据自己的配置文件来进行配置.

## 疑问点

 什么时候使用@Component而不能用@Bean?