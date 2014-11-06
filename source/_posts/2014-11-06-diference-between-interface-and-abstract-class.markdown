---
layout: post
title: "接口和抽象类的区别  Diference Between Interface and Abstract Class"
date: 2014-11-06 18:03:19 +0800
comments: true
categories: [basics,java]
---
面试过程中经常会被问道Java中接口和抽象类有什么区别，总觉得自己总结不好，也不全面。正好这里[有篇文章](http://javapapers.com/core-java/abstract-and-interface-core-java-2/difference-between-a-java-interface-and-a-java-abstract-class/)感觉写得很好。下面是内容。

1. 接口最主要的不同是，接口默认是抽象的，本身不能有任何实现。抽象类可以有为子类提供的默认实例方法。

2. 接口中的变量默认是final的。抽象类可以包含非final的变量。

3. 接口的成员默认是public的。抽象类可以有任意类型的成员，private，protected都可以。

4. 实现接口需要用implements关键字。继承抽象类用extends关键字。

5. 接口继承一个或者多个接口，但是不能继承或实现任意抽象抑或者实体类。抽象类可以继承另外一个Java类同时还可以实现多个Java接口。

6. 一个Java类可以实现多个接口，但是只能继承一个抽象类。

7. 接口是完全抽象的，不能够实例化。抽象类也是不能被实例化的，但是如果其中有main()方法时，还是可以调用。

8. 和抽象类相比，接口定义的引用在具体执行时由于需要做额外的间接寻址工作因此会慢些。(执行时需要通过搜索具体类的实现方法)
