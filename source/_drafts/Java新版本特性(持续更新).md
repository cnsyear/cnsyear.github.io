---
title: Java新版本特性(持续更新)
toc: true
abbrlink: a2f40993
date: 2020-12-14 22:13:41
tags: Java
categories: Java基础
---

[Java版本历史](https://zh.wikipedia.org/zh-cn/Java%E7%89%88%E6%9C%AC%E6%AD%B7%E5%8F%B2)

2017年9月，Java平台的主架构师 Mark Reinhold 发出提议，要求将Java的功能更新周期从之前的每两年一个新版本缩减到每六个月一个新版本。该提议获得了通过，并在提出后不久生效。

<!--more-->
Java 8 与 Java 11 为目前提供支持的LTS（长期支持）版本；Java 10 是上一个快速发布版本，且已不再被支持。2018年9月，随着 Java 11 的发布，Java 10 自当日起不再被支持。Oracle 将在 2019 年 1 月前为商业用途中的 Java 8 长期支持，而针对非商用的更新将继续提供，直至 2020 年 12 月；此外，AdoptOpenJDK 也为 Java 8 提供免费更新。针对 Java 11 的长期支持将不再由 Oracle 提供，而是改由 OpenJDK 社区的 AdoptOpenJDK 提供。

![image](/static/img/12.png)

不用忙于学习新版本特性，可以等到LTS版本发布后再补即可~ 因为目前项目还停留在Java8的时代，接下来也不太会升级到Java11，期待着Java17的王者归来。

#### 一、Java8新特性
Lambda表达式、强大的 Stream API、时间日期 API、ConcurrentHashMap、MetaSpace。Java8 的新特性使 Java 的运行速度更快、代码更少（Lambda 表达式）、便于并行、减少空指针异常。


#### 二、Java9新特性
模块化系统，REPL工具：jshell命令，多版本兼容jar包，语法的新变化：接口私有方法、异常处理、钻石操作符、String存储结构变化等，新增API：Stream、List、Set、图像处理等。


#### 三、Java11新特性
新的局部变量的语法、更方便的调试运行程序的方式jshell及直接运行源代码、令人瞩目的ZGC, JFR、新HttpClient API、兼容Unicode10的新的字符串API等。
