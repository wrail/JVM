# 初识JVM

> JVM是Java的核心，没有JVM那么Java就不会运行，因此深入了解JVM更有利于理解Java。

## 什么是JVM？

> JVM是可运行Java代码的假想的计算机，包括一套字节码指令集，一组寄存器，一个栈，一个垃圾回收，一个堆和一个存储方法域。

JVM是Java Virtual Machine的简称。意为Java虚拟机

JDK和JVM的发展：

1996年 SUN JDK 1.0 Classic VM纯解释运行,在此时的Java正因为纯解释运行而让运行速度十分的慢。

1997年 JDK1.1 发布AWT、内部类、JDBC、RMI、反射。

1998年 JDK1.2 Solaris Exact VMJIT 解释器混合Accurate Memory Management 精确内存管理，数据类型敏感提升的GC性能，从此时开始进进入的Java2的时代，此时加入了Swing。

2000年 JDK 1.3 Hotspot 作为默认虚拟机发布，加入JavaSound。

2002年 JDK 1.4 Classic VM退出历史舞台。Assert，正则表达式，NIO ，IPV6，日志API，加密类库。

2004年发布 JDK1.5 即 JDK5 、J2SE 5 、Java 5。泛型，注解，装箱，枚举，可变长的参数，Foreach循环

JDK1.6 JDK6脚本语言支持JDBC 4.0，Java编译器 API

2011年 JDK7发布，G1，动态语言增强，64位系统中的压缩指针，NIO 2.0

2014年 JDK8发布，Lambda表达式，语法增强，Java类型注解

2016年JDK9，模块化

等等

JVM历史：

使用最为广泛的JVM为HotSpot，HotSpot 为Longview Technologies开发，被SUN收购

2006年 Java开源 并建立OpenJDK，HotSpot成为Sun JDK和OpenJDK中所带的虚拟机

2008 年 Oracle收购BEA，得到JRockit VM

2010年Oracle 收购 Sun，得到Hotspot

Oracle宣布在JDK8时整合JRockit和Hotspot，优势互补在Hotspot基础上，移植JRockit优秀特性。

## 讲到JVM就必须提一提Java

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190405124403566.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

Java是依赖于JVM运行的，但是JVM运行的语言不限于Java，只要符合JVM规范都可以被运行。

Java代码经编译为class文件（javac）然后使用classloader装，然后执行class（执行分为解释执行和编译执行（client compiler，server compiler））。

具体如下图，可能会有不对的地方，后边会详细的进行JVM学习来进行评判是不是这样一个体系图。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190405150919415.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)






