---
title: "Spring学习笔记02 - Spring IoC"
description:
date: "2024-03-17T14:44:31+08:00"
slug: "spring-learning-02"
image: ""
license: false
hidden: false
comments: false
draft: true
tags: ["Spring", "Java", "Spring IoC"]
categories: ["Spring"]
# weight: 1 # You can add weight to some posts to override the default sorting (date descending)
---

## 什么是 Spring IoC?

Spring IoC（Inversion of Control，控制反转）是 Spring 框架的核心机制，其核心思想是将对象的创建、依赖管理和生命周期交给容器统一控制，而非由开发者手动通过 new 创建对象。

通过 IoC，代码的耦合度降低，程序的灵活性和可维护性显著提高。

