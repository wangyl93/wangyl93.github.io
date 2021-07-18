---
layout: post
title: SpringBoot 的@Value注解太强大了，用了都说爽！
category: springboot
no-post-nav: true
tags: [springboot]
keywords: Spring Boot,@Value注解
excerpt: Spring Boot 和 @Value注解
---

## SpringBoot 的@Value注解太强大了，用了都说爽！

**一、前言**



在日常开发中，经常会遇到需要在配置文件中，存储 `List` 或是 `Map` 这种类型的数据。

Spring 原生是支持这种数据类型的，以配置 `List` 类型为例，对于 `.yml` 文件配置如下：

`t`est:`
 `list:`
  `\- aaa`
  `\- bbb`
  \- ccc`

对于 `.properties` 文件配置如下所示：

```java
test.list[0]=aaa
test.list[1]=bbb
test.list[2]=ccc
```

当我们想要在程序中使用时候，想当然的使用 `@Value` 注解去读取这个值，就像下面这种写法一样：

`@Value("${test.list}")`
`**private** List<String> testList;`

你会发现程序直接报错了，报错信息如下：

`java.lang.IllegalArgumentException: Could not resolve placeholder 'test.list' in value "${test.list}"`

这个问题也是可以解决的，以我们要配置的 key 为 `test.list` 为例，新建一个 `test` 的配置类，将 `list` 作为该配置类的一个属性：

```java
@Configuration
@ConfigurationProperties("test")
public class TestListConfig {
    private List<String> list;

    public List<String> getList() {
        return list;
    }

    public void setList(List<String> list) {
        this.list = list;
    }
}
```

 





