---
title: JVM从入门到放弃
typora-root-url: ../../source/
date: 2019-09-16 21:03:40
tags:
- java
- 面试
- jvm
---

# 垃圾收集器总结

![image-20190916210424179](/imgs/image-20190916210346672.png)

<!--more-->

# 类加载机制

## 类加载流程

- 加载：获取class文件，将其中的**静态存储结构**转化为方法区的**运行时数据结构**，生成Class对象
- 验证：文件格式验证，元数据验证，字节码验证，符号引用验证（解析阶段发生）
- 准备：为类变量分配内存并设置初始值0，如果是static final则直接赋值
- 解析：将常量池中的符号引用替换为直接引用的过程
- 初始化：执行类构造器<clinit()>方法

> clinit方法：编译器自动收集类变量赋值动作和static代码块语句合并产生的
>
> 父类的clinit方法先于子类的clinit方法执行

（解析过程有可能在初始化阶段后才完成）

## 什么时候会发生类初始化？

- new/getstatic/putstatic/invokestatic : new关键字实例化对象/读取或设置static字段/调用静态方法
- 使用java.lang.reflect包进行反射调用的时候
- 初始化子类的时候，首先初始化父类
- 虚拟机启动时，初始化主类
- 动态语言支持，java.lang.invoke.MethodHandle实例最后解析的结果REF_getStatic/REF_putStatic/REF_invokeStatic的方法句柄，该方法句柄需要首先进行初始化

## 不会初始化的情况

- 通过子类引用父类的静态字段，不会导致子类初始化
- 通过数组定义来引用类，不会触发此类的初始化

```java
SuperClass sca = new SuperClass[10]; //不会导致SuperClass的初始化
```

- 常量的使用不会触发定义常量的类的初始化

```java
public class ConstClass{
  public static final String HELLO = "hello world";
}
public class NotInitialization{
  public static void main(String[] args){
    System.out.println(ConstClass.HELLO); //不会导致ConstClass的初始化
  }
}
```

- 接口的初始化不会触发父接口的初始化

# 双亲委派模型

| 类加载器         | 加载内容                                                     | 使用说明                          |
| ---------------- | ------------------------------------------------------------ | --------------------------------- |
| 启动类加载器     | 加载<JAVA_HOME>\lib目录中的类库                              | 无法直接引用                      |
| 扩展类加载器     | 加载<JAVA_HOME>\lib\ext目录中的类库                          | 开发者可直接使用                  |
| 应用程序类加载器 | getSystemClassLoader()返回，也叫系统类加载器   加载用户类路径上的所有类库 | 程序默认类加载器,开发者可直接使用 |