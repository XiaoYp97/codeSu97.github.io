---
title: "Spring学习笔记01 - Spring Framework"
description:
date: "2024-03-10T19:49:27+08:00"
slug: "spring-learning-01"
image: ""
license: false
hidden: false
comments: false
draft: false
tags: ["Spring", "Java", "Spring Framework"]
categories: ["Spring"]
# weight: 1 # You can add weight to some posts to override the default sorting (date descending)
---

{{< quote author="Spring Framework" source="spring.io" url="<https://spring.io/projects/spring-framework">}}>
Spring Framework 为基于 Java 的现代企业应用程序提供了一个全面的编程和配置模型，适用于任何类型的部署平台。

Spring 的一个关键要素是应用层面的基础架构支持：Spring 专注于企业应用程序的 “管道”，因此团队可以专注于应用程序级的业务逻辑，而无需与特定部署环境进行不必要的绑定。
{{< /quote >}}

## Spring Framework 有哪些核心模块?

- spring-core
  - Spring 基础的API模块，如资源管理、泛型处理
  - 提供框架的基本工具类和核心工具，如依赖注入（DI）的实现、IoC 容器的基本功能等。
- spring-beans
  - 用于管理和配置应用程序中的 bean，Bean对象的创建、生命周期管理，依赖查找、依赖注入
- sprign-context
  - 事件驱动、注解驱动、模块驱动等
  - 建立在 Core 和 Beans 之上，提供类似于 JNDI 的功能，以及国际化、事件传播、资源加载等功能。它类似于一个轻量级的容器，便于集成不同的框架。
- spring-expression
  - Spring 表达式 语言模块
  - 提供强大的表达式语言支持，可以在 XML 或注解中动态计算值，增强了配置和动态功能的灵活性。
- spring-aop
  - Spring AOP 处理，如动态代理、AOP字节码提升
  - 实现面向切面编程的支持，通过切面（Aspect）来定义横切关注点，如日志、安全、事务等，从而实现与业务逻辑的分离。这一模块使得应用程序能够在不修改核心业务代码的情况下添加额外的行为。
- spring-jdbc
  - 封装了 JDBC 操作，提供了模板化方法来简化数据库访问和异常处理，减少了样板代码。
- spring-orm
  - 为主流 ORM 框架（如 Hibernate、JPA、MyBatis 等）提供支持，简化数据持久化操作的配置和集成。
- spring-tx
  - 提供声明式事务管理，帮助开发者在不同的数据访问技术之间保持一致的事务管理策略。
- spring-web
  - 为开发 web 应用提供基本的支持，包括 WebSocket、Multipart 文件上传、以及 HTTP 请求和响应处理。
- spring-webmvc
  - 实现了 Model-View-Controller (MVC) 设计模式，提供了一整套用于构建基于请求-响应模型的 Web 应用程序的组件。通过该模块，可以轻松构建松耦合、可测试的 Web 应用。
- spring-test
  - 提供对 Spring 应用的单元测试和集成测试的支持，包括对 JUnit 和 TestNG 的集成，帮助开发者在不依赖外部服务器的情况下模拟和测试应用的行为，确保代码质量和稳定性。

## 优势

- 模块化和灵活性：Spring 允许开发者选择需要的组件，支持 XML 和基于注解的配置，适合各种项目需求。
- 轻量级和便携性：无需繁重的应用服务器，可以在像 Tomcat 这样的 servlet 容器上运行，降低资源占用。
- 依赖注入和控制反转（IoC）：促进组件松耦合，易于测试和扩展。
- 面向切面编程（AOP）：模块化横切关注点，如日志、安全和事务管理，提升代码复用性。
- 丰富的生态系统：提供数据访问、Web 服务、安全等多个模块，满足企业级应用需求。
- 易于测试：依赖注入功能简化测试数据注入，确保应用可靠性。
- 一致的 API：为异常处理、事务管理等提供统一接口，简化开发。
- 长期维护和可靠性：Spring 历史悠久，持续更新，适合长期项目。

## 不足

- 学习曲线陡峭：对于新手或不熟悉 DI 和 AOP 的开发者，学习成本较高。
- 复杂性：功能丰富可能导致项目复杂，尤其对小型项目不必要。
- 配置冗长：尽管注解减少了 XML 配置，但某些设置仍可能繁琐。
- 安全问题：Spring 提供安全特性，但配置不当可能导致如跨站脚本攻击（XSS）等漏洞，需额外注意。
- 性能开销：抽象层可能引入轻微性能开销，但通常对大多数用例影响不大。
