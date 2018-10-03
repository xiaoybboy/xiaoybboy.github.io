---
title: Java Scanner next()和nextLine()
date: 2017-04-02 13:29:44
tags: [java,Scanner]
categories: java
---
java中使用Scanner类获取数据输入十分方便，Scanner类中next()与nextLine()都可以实现字符串String的获取.它们的区别如下：
1. next() 方法从第一个有效字符（非空格，非换行符），开始扫描。当遇见第一个分隔符或结束符(空格或换行符)时，结束扫描，获取扫描到的内容，即获得第一个扫描到的不含空格、换行符的单个字符串。
2. 使用nextLine()时，则可以扫描到一行内容并作为一个字符串而被获取到。可以获取空格。
<!--more-->
## 测试
```java
public static void main(String[] args) {
	Scanner scanner = new Scanner(System.in);
	System.out.println("--->Test1:\n");
	String nextStr = scanner.next();
	System.out.println("scanner.next()得到：" + nextStr);
	String nextlineStr = scanner.nextLine();
	System.out.println("scanner.nextLine()得到：" + nextlineStr);

	System.out.println("\n--->Test2:");
	String nextlineStr2 = scanner.nextLine();
	System.out.println("scanner.nextLine()得到：" + nextlineStr2);
	String nextStr2 = scanner.next();
	System.out.println("scanner.next()得到：" + nextStr2);
	}
```
## 结果
![结果](http://i.imgur.com/vgRn5UN.png)