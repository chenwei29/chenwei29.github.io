---
title: springsercurity
date: 2021-07-07 23:33:04
tags: [spring,学习]
---

## springsecurity概要

​	springsecurity是spring家族的一员,springsecurity是基于spring框架来进行开发的,提供了一套Web应用安全性的完整的解决方案.(认证和授权)

​	用户认证值得是:验证某个用户是否为系统中的合法用户,也就是该用户是否允许访问该系统.一般是提供账号和密码来进行验证,

​	用户授权是指验证某个用户是否有权限执行某个操作.在一个系统中,不同的用户的索拥有的权限是不同的.

### springsecurity和shiro进行对比

springsecurity的特点:

- 和spring无缝整合(因为就是属于spring下的)
- 全面的权限控制
- 专门为web开发而设计
  - 旧版本不能脱离Web环境使用
  - 新版本对整个框架进行了分层抽取,分成了核心模块和Web模块,单独引入核心模块就可以脱离Web环境
- 比较重量级(需要依赖于spring,需要引入很多的依赖.)

shiro是Apache旗下的轻量级权限控制框架,shiro的特点:

- 轻量级(不依赖于其他依赖,启动起来,服务也比较少)
- 通用性
  - 好处 :不局限于Web环境,可以脱离Web环境使用
  - 坏处: 在Web环境下一些特定的需求需要手动编码定制.

总结:

​	springsecurity发展了很久,但是使用的不多,而且在SSM中整个springsecurity是比较麻烦的操作,所以虽然springsecurity功能强大,但是使用的没有shiro多,对于大部门的项目来说,shiro的功能足够使用了,自动有了springboot之后,提供了自动配置化方案,可以使用更少的配置来使用springsecurity.因此,一般来说,常见的安全管理技术栈的组合为:

- SSM+shiro
- springboot/springcloud + springsecurity

以上无论怎么组合都是可行的,只是推荐而已.

## 入门案例

简单做一个简单的入门案例:

1. 创建一个springboot工程

2. 引入相关的依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-security</artifactId>
   </dependency>
   ```

   

3. 编写controller进行测试.
