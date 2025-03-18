---
title: "如何理解 Final 关键字？"
description:
date: "2024-03-21T16:58:52+08:00"
slug: "java-final"
image: ""
license: false
hidden: false
comments: false
draft: true
tags: ["Java", "final"]
categories: ["Java"]
# weight: 1 # You can add weight to some posts to override the default sorting (date descending)
---

在 Java 中，`final` 关键字是一个修饰符，用于定义变量、方法或类的不可变性或不可继承性。

它的核心作用是限制修改或继承，从而增强代码的安全性、可读性和设计约束。

## 作用范围

### 修饰变量

局部变量：必须在声明时或使用前赋值一次，之后不可修改。

```java
final int x = 10;
// x = 20; // 编译错误：无法修改 final 变量
```

成员变量：必须在声明时、构造方法或初始化块中赋值一次。

```java
class MyClass {
    final int MAX_VALUE; // 非静态 final 变量
    final static int MIN_VALUE = 0; // 静态 final 常量

    public MyClass() {
        MAX_VALUE = 100; // 在构造方法中赋值
    }
}
```

引用类型变量

`final` 修饰的是引用，而非对象本身。引用不可变，但对象内部状态可能可变。

```java
final List<String> list = new ArrayList<>();
list.add("Java"); // 允许修改对象内容
// list = new ArrayList<>(); // 编译错误：引用不可变
```

### 修饰方法

方法不可被重写：子类无法覆盖父类的 final 方法。

```java
class Parent {
    public final void print() {
        System.out.println("Parent");
    }
}

class Child extends Parent {
    // @Override // 编译错误：无法重写 final 方法
    // public void print() { ... }
}
```

设计意图：

- 防止关键方法（如算法逻辑、安全校验）被子类意外修改。

### 修饰类

类不可被继承：

```java
final class StringUtils { // 工具类通常设计为 final
    // 工具方法...
}
// class ExtendedUtils extends StringUtils { } // 编译错误
```

设计意图：

- 防止继承破坏类的内部逻辑（如 `String`、`Integer` 等不可变类）。
- 强制使用组合而非继承（如工具类）。

## 设计意义

### 安全性

- 常量定义：通过 `final` 变量定义全局常量（如配置参数），防止意外修改。
- 不可变对象：结合私有字段和 `final` 修饰符，实现线程安全的不可变对象。

```java
public final class ImmutablePoint {
    private final int x;
    private final int y;

    public ImmutablePoint(int x, int y) {
        this.x = x;
        this.y = y;
    }
    // 只有 getter，没有 setter
}
```

### 性能优化

- 编译优化：`final` 常量在编译时可能被直接替换为字面量（类似宏）。
- JVM 优化：`final` 方法可能被内联（Inline）以提高执行效率。

### 设计约束

- 强制规范：通过 `final` 限制扩展或修改，明确类的职责（如工具类）。
- 防止破坏性继承：避免子类覆盖方法导致父类逻辑失效。

## 注意事项

### 引用类型变量

`final` 只能保证引用不变，但对象内部状态可能被修改（除非对象本身不可变）。

```java
final int[] arr = {1, 2, 3};
arr[0] = 100; // 允许修改数组内容
```

### final 与不可变对象

`final` 是实现不可变对象的必要条件，但非充分条件。需结合以下条件：

- 所有字段用 final 修饰。
- 字段为基本类型或不可变对象。
- 不对外暴露修改内部状态的方法（如 setter）。

### final 参数

方法参数可以声明为 `final`，防止在方法内被意外修改（增强可读性）。

```java
public void process(final String input) {
    // input = "new value"; // 编译错误
}
```

## 强制约束机制

### 对变量的强制约束

**编译阶段检查:** 编译器会严格检查 final 变量的赋值次数。以下代码会直接编译失败：

```java
final int x;
x = 10;
x = 20; // 编译错误：Variable 'x' might already have been assigned
```

**运行时不可修改:** 即使通过字节码操作（如直接修改 .class 文件），试图改变 final 变量的值也会导致 IllegalAccessError。

（注：常规开发中无法绕过此限制。）

### 对方法的强制约束

**子类重写直接报错：** 如果子类尝试重写父类的 final 方法，编译器会直接拒绝：

```java
class Parent {
    public final void doSomething() {}
}

class Child extends Parent {
    @Override // 编译错误：Cannot override the final method
    public void doSomething() {}
}
```

### 对类的强制约束

**禁止继承的编译检查：** 任何尝试继承 final 类的行为都会导致编译错误

```java
final class UtilityClass {}
class SubClass extends UtilityClass {} // 编译错误：Cannot inherit from final class
```
