---
title: "深入理解重载和重写及与之相关的多态性  Overloading and overriding"
img: 20180411.png
date: 2014-11-05 16:03:56 +0800
tags: [java,basics]
layout: post
---
本文分割线之前非原创，翻译自[Overloading and overriding](http://javapapers.com/core-java/overloading-and-overriding/).

重载和重写除了在名称上有些相似之外，其实是完全不同的两个东西。

重载的目的是使得我们能够用用一个统一的接口名称来调用一系列方法。这些方法的目的也许是一样的，但是它们的实现方式会根据传入的参数不同而不同。

重写涉及到继承这个概念中的问题。子类继承了父类的方法，但是它可能需要有不同的操作行为，这时候就需要在子类中重写这个父类方法。

重载本身并不是多态，同时运行时绑定重载方法也不是多态的表现。它们以及一些其他的东西都是其面向对象多态性的使用（原文：All these and more are used to exercise the object oriented property polymorphism.总感觉翻译不要，故贴上来）。

## 重载和重写的关键点

- private： 一个私有的java方法是不能被重写的，因为它对子类压根就不可见
- final：重载一个final的方法是可以的，但是不能重写它，因此父类如果将方法声明为final的就可保证所有子类的调用此方法时调用的都是父类的方法。
- final：如果两个方法有同样的参数列表，而其中一个的参数被声明为final的这种情况下这两个方法完全一样，因此不可重载。编译都通不过，因为这两个方法被视为完全一样。
- static：可以重载一个静态的Java方法但是不能重写静态的Java方法，因为静态方法在方法区中只有一个。
- static：重载是关于对象(实例）和继承而言的。一个声明为静态的方法属于整个类(对于这个的所有对象都是一样的)。因此重写它没有任何意义。
- static：对于重载，两个静态方法的重载没有什么特别的，只不过是修饰符多了个static修饰符。参数列表依然必须不同。
<!--more-->
## 重载

让我们来详细讨论下重载。在各种网站上我看到了大量的重载的例子。它们向类中添加更多的属性、方法，把它变得看起来更庞大作为重载的象征。事实上正确的方式是，当我们使用重载时外部看起来我们的类会变得更加紧凑。

![重载之前](beforeoverloading.png)

这里贴出来两个图片，第一张里面是印度的古典乐器脚踏式风琴，它是现代钢琴的鼻祖。另一幅图片是现代的电子琴。在我们的重载语境中，古典风琴是重载之前，电子琴是重载之后。

![重载之后](afteroverloading.png)

### Java Example for Method Overloading
--------------------------------------
	
	public class OverloadingExample {
		public static void main(String args[]){
			System.out.println(playMusic("C sharp","D sharp"));
			System.out.println(playMusic("C","D flat","E flat"));
		}
		public static String playMusic(String c, String d){
			return c+d;
		}
		public static String playMusic(String c, String d, String e){
			return c+d+e;
		}
	}

### Java Overloading Puzzle
---------------------------
 	package com.javapapaer.java;
		
	public class NullArguementOverloading {
		public static void main(String[] args) {
			NullArguementOverloading obj = new NullArguementOverloading();
			obj.overLoad(null);
		}
		private void overLoad(Object o){
			System.out.println("Object o arguement method.");
		}
		private void overLoad(double[] dArray){
			System.out.println("Double array argument method.");
		}
	}


猜一猜，上面的输出是啥？

### Overriding  重写
-------------------

当继承一个类的时候,根据父类方法的访问修饰符子类可以获得所有protect以上的父类方法。但是父类的方法的具体行为可能在子类中并不适合，因此我们需要根据子类对于这个方法的需求重写继承自父类的这个方法。重写后原来的旧方法对于这个子类会完全废弃。

看看下面这个猛兽皮卡，因为需求不同了，原来又小又旧的轮子被换成了巨无霸。这就是现实生活中的重写。

![Overriding](Overriding.png)

### Overriding in Java/JDK

在Java中默认所有对象都是继承自Object类。Object有个叫*equals*的方法，在String类中重写了它的默认行为。String中通过比较传入的对象与本身保存的字符串序列一一对比看是否相等。

#分割线～～～～～～～～～～～～～～～～～～～～～～～
-----------------------------------------------------
关于上面Overloading Puzzle，以免强迫症患者费心敲代码在此给出答案和解释。

程序输出：**Double array argument method.**

原因是Java对于重载的处理有个最精确原则。Object和double[]都可以和null匹配，但是两者对比而言，double[]比Object匹配得更精确，所以调用了下面的overLoad方法。如果想让程序调用Object的overLoad方法则需要一个强制类型转换`obj.overLoad((Object)null);`这样Object的overLoad就要比double[]的更精确,会输出*Object o arguement method.*。

下面这段代码也许理解起来会更清晰一些，ArrayList继承自List：

	import java.util.ArrayList;
	import java.util.List;

	public class OverridePuzzle {

		private void overloadList(List list){
			System.out.println("List arguement method.");
		}

		private void overloadList(ArrayList arrayList){
			System.out.println("ArrayList arguement method");
		}
		public static void main(String[] args) {
			OverridePuzzle op = new OverridePuzzle();
			op.overloadList(null);
		}

	}

那么它将输出:**ArrayList arguement method**，因为ArrayList比List匹配得更精确。

那又要问了，如果上文的NullArguementOverloading代码中还有个overLoad方法如下：

	private void overLoad(String str) {
		System.out.println("String argument method.");
	}

那么程序会怎样呢？答案是main函数`obj.overLoad(null)`那一行会编译报错。String的overLoad和double[]的overLoad都可以匹配，但是两者在继承属上是平行的，因此编译器也不知道到底该调用哪一个重载方法。另外这段代码还说明了一个问题：重载是在编译期就已经确定了的，并不需要等到运行时才能确定。这也是为什么说它不是多态的一个原因。

还有下面一段代码作为佐证：

	import java.util.ArrayList;
	import java.util.List;

	public class OverridePuzzle {

		private void overloadList(List list){
			System.out.println("List arguement method.");
		}

		private void overloadList(ArrayList arrayList){
			System.out.println("ArrayList arguement method");
		}
		public static void main(String[] args) {
			OverridePuzzle op = new OverridePuzzle();
			List list = new ArrayList<String>();
			op.overloadList(list);
		}

	}

程序输出：**List arguement method.**。显然这里重载对于传入的参数类型只认了引用的类型，并没有去解析实际对象的类型。如果重载是一种多态的话，它这里应该去解析实际对象的类型并调用ArrayList的方法。
