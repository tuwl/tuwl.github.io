---
title: Field injection is not recommended
date: 2018-06-03 15:32:38
categories:
- 翻译
tags:
- Java
- Spring

---
[原文链接](https://blog.marcnuri.com/field-injection-is-not-recommended/)

# 引言
当运行一个静态代码分析工具或者IDE检查分析你的代码时，你可能遇到以下的关于你的@Autowired字段的警告。
> Field injection is not recommended

![](https://ws1.sinaimg.cn/large/bdef7c44gy1fry8jz4lo1j20lz029dfs.jpg)

这篇文章展示了在Spring中能够使用的不同类型的注入，以及每一种的推荐使用模式。

<!--more-->
# 注入类型
虽然现在的 spring framework (5.0.3) 文档只定义了两种主要的注入类型，实际上存在三种：

* 基于构造函数的依赖注入
* 基于 setter 的依赖注入
* 基于字段的依赖注入

最后一种就是静态代码检查工具所抱怨的，但是它的使用十分经常和广泛。
你甚至在 Spring 的引导可以看到这个注入方式，虽然文档里不推荐。

![](https://ws1.sinaimg.cn/large/bdef7c44gy1fry8kf3nmmj20m70opmza.jpg)

## 基于构造函数的依赖注入
在基于构造函数的依赖注入中，类的构造函数被 @Autowired 注解并包括了被注入的不同数量的参数对象。
```java
@Component
public class ConstructorBasedInjection {

    private final InjectedBean injectedBean;

    @Autowired
    public ConstructorBasedInjection(InjectedBean injectedBean) {
        this.injectedBean = injectedBean;
    }

}
```
基于构造函数的依赖注入的主要有点就是你可以用final声明你的注入字段，它们会在类生成实例的过程中被创建。这对于被需要的依赖来说是便利的。

## 基于 setter 的依赖注入
在基于 setter 的依赖注入中， setter 函数被 @Autowired 所注解。只要bean通过一个无参构造函数或者一个无参静态工厂函数被实例化， Spring 容器会调用这些 setter 函数从而注入 bean 的依赖。
```java
@Component
public class ConstructorBasedInjection {

    private InjectedBean injectedBean;

    @Autowired
    public void setInjectedBean(InjectedBean injectedBean) {
        this.injectedBean = injectedBean;
    }

}
```
## 基于字段的依赖注入
在基于字段的依赖注入中，字段/属性被 @Autowired 所注解。只要类被创建实例了 Spring 容器将会 set 这些字段。
```java
@Component
public class ConstructorBasedInjection {

    @Autowired
    private InjectedBean injectedBean;

}
```
如你所见，这是最简洁的依赖注入方法，它避免了增加样板代码，并且没有为类声明一个构造函数的必要。代码看起来很不错，简洁整齐，但是正如代码检查工具已经提醒我们的，这种方法有一些缺点。

# 基于字段的依赖注入的缺点
不允许声明final字段
如果声明了final/不变的字段，基于字段的依赖注入将不会工作。字段必须在类实例化的过程中创建。唯一一种声明 final 依赖的方法是基于构造函数的依赖注入。

## 有助于违反单一负责原则
如你所知，在面向对象的计算机编程中， SOLID 缩写定义了五个使你的代码便于理解，有弹性，可维护的设计原则。
SOLID 的 S 代表了单一责任原则，这意味着一个类应该只对软件应用的一个单一部分的功能负责，所有它的服务应该对它的责任精简的对齐。
通过基于字段的依赖注入，十分容易在你的类中拥有许多依赖，一切看起来很好。如果替代使用基于构造函数的依赖注入，当越来越多的依赖添加到你的类中，构造函数会变得越来越臃肿，代码开始变味，发送了有些东西是错误的信号。
拥有一个多于十个参数的构造函数是一个清晰的信号标明类有太多的合作者，这可能是一个好的时机去开始分离类变成小一点且更多的可维护的部分。
所以虽然基于字段的依赖注入没有直接对破坏单一负责原则负责，它帮助隐藏了代码应该简洁的信号。

## 与依赖注入容器紧密耦合
使用基于字段的依赖注入的主要原因是避免 getter setter 的样板代码或者为你的类创建一个构造函数。最终这意味着这是唯一的方法，字段通过 spring 容器实例化类来安置，通过反射来注入它们，否则字段将会保持为空且你的代码将会变的被破坏/无用的。
依赖注入设计模式将类依赖关系的创建与类本身的创建分离，将此责任转移到类注入器，从而允许程序设计松散耦合并遵循单一责任和依赖倒置原则（再次 SOLID ），因此，通过自动装配其字段来实现类的自动装配的解耦最终无效因为与类注入器（在 Spring 的例子中）的耦合，使得该类在 Spring 容器外无用。
这意味这如果你想要在应用容器之外使用你的类，比如说单元测试，你必须强制使用 spring 容器去实例化你的类，除了反射之外没有其他的方法去设置自动装配的字段。

## 隐藏的依赖
当使用一个依赖注入模式时，受影响的类需要通过使用一个公共接口清楚的暴露这些依赖，或者在构造函数里暴露需要的依赖，或者可选的 setter 方法。当使用基于字段的依赖注入时，类本质上在对外界隐藏它的依赖。

# 结论
我们已经看到了，无论何时，无论它看上去多优雅基于字段的依赖注入应该被禁止因为它的众多缺点。推荐的方法是使用基于构造函数的依赖注入和基于 setter 的依赖注入。前者推荐在需要依赖不能被修改或者保证他们不为空的时候使用，后者推荐需要可选依赖的时候使用