---

title: Spring Boot 的 Maven 项目原型
date: 2017-08-11
updated: 2017-08-11
description: Spring Boot Archetype 将帮助你快速生成 Spring Boot 项目
tags:
  - Spring Boot
categories:
  - 最佳实践

---


## 概要
随着微服务的流行，Spring Boot 在广大开发者中占据了越来越重要的位置，其开箱即用、自动配置的特性也给我们带来了诸多便利。

然而 Spring Boot 只是一个基础框架，我们还是需要新建工程、技术选型、参数配置等一系列步骤才能搭建出一个完整的项目，繁冗又无趣，尤其对于初学者来说，这一过程还可能会出现各种各样莫名其妙的问题。

[Spring Boot Archetype](https://github.com/drtrang/maven-archetype-springboot) 则是为了解决上述痛点而打造，借助 `maven-archetype-plugin` 插件，预置了日志、缓存、AOP、数据访问、代码生成、文档生成等模块，并提供常见技术的最佳实践，用户只需几秒即可快速构建一个可运行的 Spring Boot 项目。


## 地址
主页：https://github.com/drtrang/maven-archetype-springboot
问题：https://github.com/drtrang/maven-archetype-springboot/issues
后续计划：https://github.com/drtrang/maven-archetype-springboot/blob/master/TODO.md


## 特点
* 基于 Spring Boot 1.5.6，内嵌 Jetty
* 增加全局异常捕获、view to json 等功能
* 集成通用 Mapper 和 PageHelper，提供 BaseService，常用 CRUD 无需编写代码
* 集成 MyBatis Generator，提供 [MBG Plugin Extension](https://github.com/drtrang/mybatis-generator-extension)，如自动生成 Service 插件、支持 Lombok 插件等等
* 集成 [Druid Spring Boot Starter](https://github.com/drtrang/druid-spring-boot)，无需显式声明数据源（支持多数据源）
* 集成 Swagger2，HTTP 接口自动生成接口文档
* 提供常用工具，如 SpringContextHolder、SqlMapper


## ISSUE
项目刚刚发布，许多方面还有不足，希望大家多提意见，如果有任何想法和讨论，都可以放到 [ISSUE](https://github.com/drtrang/maven-archetype-springboot/issues) 平台，我会及时回复。有条件的用户还可以提 PR，成为该项目的 Contributor。


## About Me
QQ：349096849
Email：donghao.l@hotmail.com
Blog：[Trang's Blog](http://blog.trang.space)
