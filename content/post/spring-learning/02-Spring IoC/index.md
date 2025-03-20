---
title: "Spring学习笔记02 - Spring IoC"
description:
date: "2024-03-17T14:44:31+08:00"
slug: "spring-learning-02"
image: ""
license: false
hidden: false
comments: false
draft: false
tags: ["Spring", "Java", "Spring IoC"]
categories: ["Spring"]
# weight: 1 # You can add weight to some posts to override the default sorting (date descending)
---

## 什么是 Spring IoC?

Spring IoC（Inversion of Control，控制反转）是 Spring 框架的核心机制，其核心思想是将对象的创建、依赖管理和生命周期交给容器统一控制，而非由开发者手动通过 new 创建对象。

通过 IoC，代码的耦合度降低，程序的灵活性和可维护性显著提高。

*例如，如果一个用户服务需要数据库连接，容器会确保数据库连接被正确注入，而无需手动创建。*

- 负责管理对象的生命周期和依赖关系。
- 它通过依赖注入（Dependency Injection, DI）来实例化、配置和组装被称为“bean”的对象。
- 配置元数据可以是 XML、Java 注解或 Java 代码，增强了系统的模块化和灵活性.

## 依赖查找

### 价值

- 灵活应对运行时需求：有些依赖需要根据实时条件获取，比如按不同场景或配置，动态选择不同实现，这时直接注入一个固定 bean 可能不够灵活，需要主动查找。
- 工具或框架类的特殊需求：一些工具类或公共组件并不便于被注入到特定对象中，反而使用依赖查找更符合设计。比如在某些自定义 Starter 或底层框架中，可能会在初始化流程中需要访问容器本身。
- 在特定场景临时获取 bean：有时你需要延迟获取某些依赖，或想在调用时才加载/初始化（如懒加载），这时依赖查找也比提前注入更合适。

### 实现

#### 通过 `BeanFactory` 基础接口

```java
// 1. 创建容器
BeanFactory factory = new XmlBeanFactory(new ClassPathResource("applicationContext.xml"));

// 2. 查找 Bean（按名称）
UserService userService = (UserService) factory.getBean("userService");

// 3. 按类型查找（Spring 5+）
UserService userService = factory.getBean(UserService.class);

// 4. 按注解查找
UserService userService = factory.getBeansWithAnnotation(UserService.class);
```

**特点**：

- 最原始的 IoC 容器接口
- 需要手动处理类型转换
- 适用于简单场景（现代项目更常用 `ApplicationContext`）

#### 通过 `ApplicationContext` 接口

```java
// 1. 初始化容器（XML 配置）
ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

// 2. 按名称查找
UserService userService = (UserService) context.getBean("userService");

// 3. 按类型查找（推荐）
UserService userService = context.getBean(UserService.class);

// 4. 按名称+类型查找（无需强制转换）
UserService userService = context.getBean("userService", UserService.class);
```

**特点**：

- 企业级容器接口，支持事件、国际化等扩展功能
- 推荐使用 `getBean(Class<T>)` 避免类型转换错误

#### 通过 `ObjectProvider`（延迟/可选依赖查找）

```java
// 注入 ObjectProvider（Spring 4.3+）
@Autowired
private ObjectProvider<UserService> userServiceProvider;

// 延迟获取 Bean
UserService userService = userServiceProvider.getIfAvailable();

// 处理多个候选 Bean
UserService userService = userServiceProvider.getIfUnique();

// 显式指定条件（如通过 @Qualifier 的变体）
UserService userService = userServiceProvider.getObject();
```

**特点**：

- 支持延迟加载和可选依赖
- 解决 `NoUniqueBeanDefinitionException` 问题
- 结合 `@Lazy` 注解可实现懒加载

#### 通过 `@Autowired` + `ApplicationContextAware`

```java
@Component
public class MyBean implements ApplicationContextAware {
    private ApplicationContext context;

    @Override
    public void setApplicationContext(ApplicationContext context) {
        this.context = context;
    }

    public void doSomething() {
        UserService userService = context.getBean(UserService.class);
    }
}
```

**特点**：

- 将容器上下文注入到 Bean 中
- 适用于需要动态查找的场景

#### 通过注解扫描（基于 `@ComponentScan`）

```java
@Configuration
@ComponentScan("com.example")
public class AppConfig {}

// 初始化容器
ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);

// 查找带有 @Component 的 Bean
UserRepository repo = context.getBean(UserRepository.class);
```

**特点**：

- 结合组件扫描自动注册 Bean
- 隐式依赖查找的基础

#### 通过 JNDI 查找（传统方式）

```java
// 在 Spring XML 配置中声明 JNDI 资源
<jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/mydb"/>

// 通过容器查找
DataSource dataSource = (DataSource) context.getBean("dataSource");
```

**特点**：

- 适用于 Java EE 环境整合
- 现代项目更多使用 `@Bean` 配置数据源

#### 通过静态方法（Web 环境专用）

```java
// 获取 WebApplicationContext（如 Servlet 环境）
WebApplicationContext context =
    WebApplicationContextUtils.getWebApplicationContext(servletContext);

// 查找 Bean
UserService userService = context.getBean(UserService.class);
```

#### 对比

| 方式                   | 适用场景        | 优点            | 缺点             |
| -------------------- | ----------- | ------------- | -------------- |
| `BeanFactory`        | 简单容器操作      | 轻量级           | 功能有限           |
| `ApplicationContext` | 企业级应用（主流选择） | 功能全面          | 稍重             |
| `ObjectProvider`     | 处理延迟/可选依赖   | 解决多候选 Bean 问题 | 需要 Spring 4.3+ |
| `@Autowired` + 上下文   | 动态查找场景      | 灵活            | 侵入性强           |
| 注解扫描                 | 自动装配的基础     | 简化配置          | 需配合组件扫描        |

**推荐实践**：

- 优先使用依赖注入（被动获取）
- 仅在需要动态获取时使用依赖查找（如工厂模式、插件化架构）
- 现代项目首选 `ApplicationContext.getBean()` 或 `ObjectProvider`

## 依赖注入

依赖注入（Dependency Inject, DI）是 Spring 框架的核心功能，体现了控制反转（IoC）的设计原则。通过 DI，Spring IoC 容器负责创建和管理 bean 的依赖，并将这些依赖注入到目标 bean 中。注入方式包括：

- **构造函数注入**：通过构造函数传递依赖。
  - 优点
    - 使用 final 关键字，不可变，对象创建后，依赖不可变
    - 依赖关系明确，显著降低了组件之间的耦合性，使代码更易于测试和维护。
    - 易于测试，可以直接传递模拟依赖
    - Spring 会在启动时确保所有依赖都已注入，避免运行时空指针异常
  - 缺点
    - 如果依赖多，构造函数可能过长
    - 缺乏灵活性，创建后无法更改依赖
- **setter 注入**：通过 setter 方法设置依赖。
  - 优点
    - 灵活，可在对象创建后设置或更改依赖
    - 适合可选依赖，可能不总是需要的
    - 易于扩展，如果后期需要添加新依赖，只需添加新的 setter 方法
  - 缺点
    - 对象可变，可能导致状态不一致
    - 未设置依赖，导致空指针异常
- **字段注入**：通过注解（如 @Autowired）直接在字段上注入依赖，Spring 会使用反射机制在对象创建后自动设置字段的值。
  - 优点
    - 代码简洁，无需编写 setter 或构造函数
    - 使用简单，易于实现
    - Spring 会在启动时确保依赖注入，减少手动配置
  - 缺点
    - 与 Spring 框架紧密耦合，增加依赖
    - 依赖关系不显眼，测试可能更复杂

```java
public class MyService {
    private final MyDependency dependency;

 // 构造函数注入
    @Autowired
    public MyService(MyDependency dependency) {
        this.dependency = dependency;
    }

    public void doSomething() {
        dependency.execute();
    }
}


public class MyService {
    private MyDependency dependency;

 // Setter 注入
    @Autowired
    public void setDependency(MyDependency dependency) {
        this.dependency = dependency;
    }
}

public class MyService {
 // 字段注入
    @Autowired
    private MyDependency dependency;
}
```

## 依赖来源

**依赖对象的来源**主要就是 Spring IoC 容器（`BeanFactory` 或 `ApplicationContext`）本身。容器根据 XML、注解、Java Config 等方式加载 bean 定义并实例化这些对象，管理它们的生命周期，然后通过依赖注入或依赖查找将它们提供给应用使用。

- 自定义 Bean
  - 自己编写并交给 Spring 管理的类，比如带有 `@Component`、`@Service`、`@Repository`、`@Controller` 等注解的类，或者在 XML/JavaConfig 中显式声明的 Bean。
- 容器内建的 Bean
  - Spring 容器在启动或运行过程中，会自动创建并维护一些基础设施类或辅助功能类，供框架内部或用户使用
  - 比如 `ApplicationContext` 自身、`Environment`、`ResourceLoader`、`ConversionService`、`MessageSource`、`BeanFactoryPostProcessor`、`BeanPostProcessor` 以及各种内部用来处理注解、AOP、事务等功能的 Bean
  - `Environment environment = applicationContext.getBean(Environment.class);`
- 容器内建的依赖，（依赖注入）
  - 指的是那些不一定以 Bean 形式出现，但由容器提供、可被注入或查找的依赖。例如：
  - `BeanFactory` / `ApplicationContext`：在使用 `@Autowired` 时可以直接注入 `ApplicationContext`；
  - 环境变量、配置属性：通过 `@Value("${property}")` 或 `Environment` 对象获取；
  - 事件发布器（`ApplicationEventPublisher`）、任务调度器（`TaskScheduler`）等容器级别的基础依赖。
  - 这些依赖在某些情况下也会以 Bean 的形式出现，但总体上它们属于 Spring 内部或与运行环境强相关的内容，是容器层面“提供”的依赖。

## 元信息

- **Bean 的标识**
  - 包括 Bean 名称（`id` 或 `name`）、Bean 所对应的类名、别名等。
- **Bean 的作用域（Scope）**
  - 常见的有 `singleton`、`prototype`、`request`、`session` 等。
  - 用于决定容器如何缓存或创建 Bean 实例。
- **依赖注入信息**
  - 包括构造器参数、Setter 方法注入、字段注入等。
  - 哪些依赖是必须的？哪些是可选的？
- **生命周期回调（Lifecycle callbacks）**：
  - 初始化方法（`init-method` 或 `@PostConstruct`）、
  - 销毁方法（`destroy-method` 或 `@PreDestroy`）等
- **条件或环境**：
  - 比如 `@Conditional` 注解可以决定是否加载某个 Bean；
  - 或者 `@Profile` 控制在特定环境（dev, test, prod）下是否启用某些 Bean。
- **其他配置**：
  - 是否使用 AOP、事务等功能；
  - 是否启用延迟加载（lazy-init）；
  - 是否是自动装配（autowire），等等。

### 配置元信息

- Bean 定义配置
  - 基于 XML 文件
  - 基于 Properties 文件
  - 基于 Java 注解
  - 基于 Java API
- IoC 容器配置
  - 基于 XML 文件
  - 基于 Java 注解
  - 基于 Java API
- 外部化属性配置
  - 基于 Java 注解，@Value

## @Autowired 和 @Resource 的区别

### 注入机制

- **@Autowired** 是基于类型的注入。
  - 它会查找与字段或参数类型匹配的 bean。例如，如果一个字段是 MyDependency 类型，Spring 会注入所有实现 MyDependency 接口的 bean。
- **@Resource** 是基于名称的注入。
  - 它会根据字段名称或显式指定的名称查找 bean。例如，如果字段名为 dependency，它会查找名为 "dependency" 的 bean。
  - @Resource 还可以用于注入资源（如 DataSource），而不仅仅是 bean，这在某些复杂场景下可能更灵活。

### 是否必需

- **@Autowired** 默认是必需的。如果找不到匹配的 bean，Spring 会抛出异常。但你可以通过 required = false使它可选。
- **@Resource** 默认是可选的。如果找不到匹配的 bean，字段会保持为 null，无需额外配置。

### 标准与框架

- **@Autowired** 是 Spring 框架专有的注解，适合深度集成 Spring 的项目。
- **@Resource** 是 Java 标准的一部分（JSR-250），在 Java EE 环境中也常用，适合希望减少框架依赖的项目。

### 使用场景

- 如果你需要基于类型的注入，且依赖是必需的，推荐使用 @Autowired。例如：

 ```java
 public class MyService {
  @Autowired
  private MyDependency dependency; // 注入类型为 MyDependency 的 bean
 }
 ```

- 如果你需要基于名称的注入，或依赖是可选的，推荐使用 @Resource。例如：

 ```java
 public class MyService {
  @Resource(name = "specificDependency")
  private MyDependency dependency; // 注入名为 "specificDependency" 的 bean
 }
 ```
