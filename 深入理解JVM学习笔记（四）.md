# JVM类加载

## 类加载器的阶段分析

在Java代码中，**类型（比如是类，接口，Enum等等一些而不是对象，它们是对象的本质和载体）**的加载，连接和初始化都是在程序运行期间完成的。

* 加载：最常见是将已经存在的类的.class（字节码文件）从磁盘/硬盘加载到内存（查找并加载二进制数据）
* 连接
  * 验证：确保类的正确性
  * 准备：为类的**静态变量**分配内存，并将其初始化为**默认值**——比如int类型在此阶段还是默认值0，赋值还在初始化阶段
  * 解析：把类中的**符号引用（间接找到**）转换为**直接引用（直接找到）**
* 初始化：给变量赋正确的初始值——上边说的int类型值到此处在真正的赋值
* 使用
* 卸载

> 程序员提供了很大的空间，前三点用的多，比较重要

能够导致JVM结束生命的几种情况

* 程序中执行System.exit()方法
* 程序正常结束
* 程序出现异常或者错误
* 由于操作系统导致出现错误

## 类的加载，连接，初始化

### 类加载

JVM规范允许类加载器在预料某个类将要被使用时就预先加载它，如果预先加载过程中遇到了.class文件缺失或错误，类加载器必须在程序**首次主动使用该类时**报告错误（LinkageError），如果没有被使用也就不会报错。

#### 双亲委派机制

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190820180324500.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019082018134515.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70) 

#### JDK的几种类加载器(代码)

```Java
package com.wrial.jvm.classLoader;
/*
 * @Author  Wrial
 * @Date Created in 23:49 2019/8/27
 * @Description 循环打印出所有的类加载器
 */

public class Test11 {
    public static void main(String[] args) {
        ClassLoader classLoader = ClassLoader.getSystemClassLoader();
        System.out.println(classLoader);

        while (classLoader!=null){
            classLoader = classLoader.getParent();
            System.out.println(classLoader);
        }
    }

}

```

```Java
> Task :Test11.main()
sun.misc.Launcher$AppClassLoader@73d16e93
sun.misc.Launcher$ExtClassLoader@15db9742
null
```

可以看到最低的是`AppClassLoader`，接下来是`ExtClassLoader`，最后是`null`，对于`null`，在`JavaDoc`中也提到过，可能`getParent`返回`null`，来代表是启动类加载器（也就是`null`代表启动类加载器）

Returns the parent class loader for delegation. Some implementations **may * use <tt>null</tt> to represent the bootstrap class loader**. This method * will return <tt>null</tt> in such implementations if this class loader's * parent is the bootstrap class loader.

#### 获取类加载器的途径

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190828133539643.png)

```Java
package com.wrial.jvm.classLoader;
/*
 * @Author  Wrial
 * @Date Created in 12:13 2019/8/28
 * @Description
 */

import java.io.IOException;
import java.net.URL;
import java.sql.DriverManager;
import java.util.Enumeration;

public class Test12 {

    public static void main(String[] args) throws IOException, ClassNotFoundException {

        ClassLoader contextClassLoader = Thread.currentThread().getContextClassLoader();        System.out.println(contextClassLoader);
        String resourceName = "com/wrial/jvm/classLoader/Test11.class  ";
        Enumeration<URL> resources = contextClassLoader.getResources(resourceName);
        while (resources.hasMoreElements()) {
            URL url = resources.nextElement();
            System.out.println(url);
        }

        Class<?> aClass = Class.forName("com.wrial.jvm.classLoader.Test10");
        ClassLoader classLoader = aClass.getClassLoader();
        System.out.println(classLoader);

        ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();
        System.out.println(systemClassLoader);

        

    }
}

```

```Java
> Task :Test12.main()
sun.misc.Launcher$AppClassLoader@73d16e93
file:/E:/IDEA/JVM/build/classes/java/main/com/wrial/jvm/classLoader/Test11.class%20%20
sun.misc.Launcher$AppClassLoader@73d16e93

```

> 数组（数组类型）的class不是由类加载器创建的（其余的是由类加载器创建的），而是由运行时的JVM创建的。因此数组对象getClassLoader和数组元素类型一致，如果是原生类型就没有类加载器。

### 连接

类被加载后，就进入了连接的阶段。连接就是将已经读入到内存的类的二进制数据合并到虚拟机的运行时环境中去。

#### 类验证

对于类的验证有很多方面的验证，比如

* 类文件结构的检查
* 语义检查
* 对字节码的验证
* 二进制兼容性的验证

等等

#### 准备

为类的**静态变量**分配内存，并将其初始化为**默认值**——比如int类型在此阶段还是默认值0

```Java
package com.wrial.jvm.classLoader;
/*
 * @Author  Wrial
 * @Date Created in 18:23 2019/8/19
 * @Description
 */

public class Test9 {

    static int a;
    static long b;

    static {
        a = 2;
        b = 3L;
    }

}
```

此时就会给a和b分配空间（4+8=12字节），并赋初值为0.

#### 解析

把类中的**符号引用（间接找到**）转换为**直接引用（直接找到）**

### 初始化

* 在静态变量声明处
* 在静态代码块中

会根据静态变量的先后顺序来初始化

#### 类初始化的条件

Java程序对类的使用分为两种

- 主动使用
- 被动使用

**所有的**Java虚拟机实现必须在**每个类或者接口**被Java程序**“首次主动使用”时才会初始化**。

**主动使用** 

1. 创建类的实例

2. 访问某个类或接口的静态变量（JVM指令（助记符）——getStatic），或者是对该静态变量赋值（JVM指令（助记符）——putStatic），调用类的静态方法（JVM指令（助记符）——invokeStatic）

3. 反射（比如Class.forName("com.wrial.Test");）

4. 初始一个类的子类，父类也是主动使用。

5. 被标记为启动类的类，如main，Test。

6. JDK7提供动态语言支持，如果发现getStatic，putStatic，invokeStatic对应的类没有初始化，就去初始化。

**被动使用**

除过上边的主动使用，其余全是被动使用，不会导致初始化的产生，但是有可能会加载，连接。

#### 概念

是指将.class文件中的二进制数据读入到内存，将其放在运行时数据的方法去内，然后创建一个java.lang.Class对象用来封装方法区内的数据。（规范并未说Class对象在哪里）

#### 加载.class的方式

* 从本地系统中直接加载
* 从网络下载.class文件
* 从zip，jar等归档文件中加载.class文件
* 从专有的数据库中提取.class文件
* 将Java源文件动态编译为.class文件（比如web中jsp最终会为转为一个servlet，servlet就是一个class）

#### 类初始化的时机

Java虚拟机初始化一个类时，要求所有父类被初始化，但这种不适用于接口。

* 初始化一个类时，并不会先初始化它所实现的接口
* 初始化一个接口，并不会初始化它的父接口

只有当程序首次使用特定接口的静态变量时才会初始化。

#### 案例

下面通过一个例子来表示一下类加载主动的含义

```Java
package com.wrial.jvm.classLoader;
/*
 * @Author  Wrial
 * @Date Created in 23:19 2019/8/18
 * @Description 深刻理解主动使用
 */

public class Test1 {
    public static void main(String[] args) {
        System.out.println(Child.str);
    }
}

class Parent{

    static String str = "hello parent";
    static {
        System.out.println("parent static block");
    }

}
class Child extends Parent{

    static String str1 = "hello child";
    static {
        System.out.println("child static block");
    }

}
```

上边输出的结果会是什么呢？

初始化只有在主动使用时次啊会被初始化，而主动使用也无非就是上面那几点

```Java
//执行结果
> Task :Test1.main()
parent static block
hello parent
```

**可以看到的是，调用的Child类的静态代码块并没有执行，因为并没有访问Child类的静态变量，因此只加载了父类（符合条件2）**

如果代码是这样

```Java
package com.wrial.jvm.classLoader;
/*
 * @Author  Wrial
 * @Date Created in 23:19 2019/8/18
 * @Description 深刻理解主动使用
 */

public class Test1 {
    public static void main(String[] args) {
        System.out.println(Child.str1);
    }
}
class Parent{

    static String str = "hello parent";
    static {
        System.out.println("parent static block");
    }

}
class Child extends Parent{

    static String str1 = "hello child";
    static {
        System.out.println("child static block");
    }
}

```

这个结果就很清晰，复合条件2，4，5

结果为

```Java
> Task :Test1.main()
parent static block
child static block
hello child
```

那我们再改一改代码，给含有main的里面加一个静态代码块

```Java
package com.wrial.jvm.classLoader;
/*
 * @Author  Wrial
 * @Date Created in 23:19 2019/8/18
 * @Description 深刻理解主动使用
 */

public class Test1 {
    static {
        System.out.println("Test1 main static block");
    }
    public static void main(String[] args) {
        System.out.println(Child.str1);
    }
}

class Parent{

    static String str = "hello parent";
    static {
        System.out.println("parent static block");
    }

}
class Child extends Parent{

    static String str1 = "hello child";
    static {
        System.out.println("child static block");
    }

}
```

运行结果

```Java
> Task :Test1.main()
Test1 main static block
parent static block
child static block
hello child

```

> 经过这个例子就应该对于主动使用有了明确的认识了

但是由上面可知，类的初始化是发生在第三阶段的，第一阶段是加载，第二阶段是连接，那我们怎么知道刚开始第一个例子中Child有没有被加载和连接呢？

下面可以使用`-XX:TraceClassLoading` JVM参数来跟踪

![1566143351455](C:\Users\weiao\AppData\Roaming\Typora\typora-user-images\1566143351455.png)

配置好参数，就来运行一下代码

```Java
package com.wrial.jvm.classLoader;

public class Test1 {
    static {
        System.out.println("Test1 main static block");
    }
    public static void main(String[] args) {
        System.out.println(Child.str);
    }
}
class Parent{

    static String str = "hello parent";
    static {
        System.out.println("parent static block");
    }

}
class Child extends Parent{

    static String str1 = "hello child";
    static {
        System.out.println("child static block");
    }

}
```

控制台输出的很多，我们就看主要的部分，我们可以看到的是Child是被加载和连接到内存的，只是没有进行初始化。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190818235322730.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

#### JVM参数的几种形态

* -XX:+<option>    表示开启option选项
* -XX:-<option>     表示关闭option选项
* -XX:<option>=<value> 表示将option的值设置为value

> JVM参数也就这三种形态

#### 颠覆三观的案例

```Java
package com.wrial.jvm.classLoader;
/*
 * @Author  Wrial
 * @Date Created in 0:19 2019/8/19
 * @Description
 */

public class Test2 {

    public static void main(String[] args) {
        System.out.println(Parent1.str);
    }
}

class Parent1{

    static final String str = "hello world!";

    static {
        System.out.println("static block Parent");
    }
}
```

能否猜到这段代码的运行结果

```Java
> Task :Test2.main()
hello world!
```

WHAT，怎么只出现了一个hello world ，不是按照规律来说应该还有静态代码块吗？

是因为final声明的是一个常量，**常量在编译期间就会存到调用这个常量的方法所在的类的常量池中**，也就是Test2的常量池中

我们可以看到Parent1并没有被初始化，因此我们可以做一个颠覆三观的举动

从编译后的代码可以看到Parent1的存在，我们可以尝试把Parent1的class删除掉，然后再运行，还能运行吗？

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190819002954503.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190819003128804.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

可以从上图看到，已经删除了Parent1，不进行重新编译我们的程序还是可以运行的

因此我就很好奇，为什么会出现这种原因？

有两种验证方式

1. 看build下面编译好的.class文件

   ![1566146138411](C:\Users\weiao\AppData\Roaming\Typora\typora-user-images\1566146138411.png)

   可以看到我在Test2的输出中引用的是一个Parent.str，然而在它编译后就直接替换为字符串了，因此，这也就是为什么编译后删除了Parent1.class 也不影响的原因了，因此它根本不加载Parent1这个类

2. 通过javap -c 进行反编译看详细过程

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190819004209127.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

   

上边是反编译的结果，我们可以看到`getstatic`是io相关的，`ldc`助记符表示将int，float，String推到栈顶。

下面就来简单的说一说一些常见助记符及其含义

#### 常量助记符及其含义

这些助记符都可以在JDK源码中看到，这些都是一个一个的类

* `ldc`：表示将int,short或String类型的常量值从常量池中推送到栈顶
* `bipush`：表示将单字节（-128 ~ 127）的常量值推送到栈顶
* `sipush`：表示将一个短整型的常量值（-32768 ~ 32767）推送到栈顶
* iconst_0(-1~5) 表示将常量0送到栈顶（iconst_-1 ~ iconst_5）

测试过的代码如下

```Java
package com.wrial.jvm.classLoader;
/*
 * @Author  Wrial
 * @Date Created in 0:19 2019/8/19
 * @Description
 */
public class Test2 {

    public static void main(String[] args) {
        System.out.println(Parent1.i);
    }
}

class Parent1 {

    static final String str = "hello world!"; //ldc
    static final short i = 127; //bipush
    static final short j = 128; //sipush
    static final int m = 0; //iconst_0
    static final short n = 1;  //iconst_1
    static final short k = 5; //iconst_5
    static final int l = 6;  //bipush
    static final int c = 32768;  //ldc

    static {
        System.out.println("static block Parent");
    }
}

```

#### 又来一个案例

```Java
package com.wrial.jvm.classLoader;
/*
 * @Author  Wrial
 * @Date Created in 10:51 2019/8/19
 * @Description
 */

import java.util.UUID;

public class Test3 {
    public static void main(String[] args) {
        System.out.println(Test4.uuid);
    }
}

class Test4{
    static final String uuid = UUID.randomUUID().toString();
    static {
        System.out.println("Test4 static block!");
    }
}
```

这段代码的输出结果会是什么呢？

```Java
> Task :Test3.main()
Test4 static block!
4fea93db-bcbf-4b3f-b37a-dca12d1267b5
```

为什么会是这样的一个结果呢？`uuid`不是`final`修饰的么，但是要注意是的后边是`UUID`随机生成的32位随机码，所有是要在运行阶段才能确定这个常量的值。

在`Javap`反编译一下，会发现这个值并不是`ldc`什么的，助记符而是`getstatic`，这也进一步说明为什么这个类会被初始化。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190819105830591.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

实例化案例

```Java
public class Test5 {

    public static void main(String[] args) {

        Test6 test6 = new Test6();
        System.out.println("分割");
        Test6 test66 = new Test6();
    }
}
class Test6{
    static {
        System.out.println("Test6 static block!");
    }
}
```

输出结果

```Java
> Task :Test5.main()
Test6 static block!
分割
```

可以看到的是，类的初始化只在第一次实例化的时候才进行。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190819110901181.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

从上图可以看出第一次会有`ldc`并且`astore_1`，第二次没有`ldc`，`astore_2`

如果改成三个实例化的话，可以看到`astore_3`

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190819111035710.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

那把代码改为下面这样会输出什么结果呢？

```Java
package com.wrial.jvm.classLoader;
/*
 * @Author  Wrial
 * @Date Created in 11:01 2019/8/19
 * @Description
 */

public class Test5 {

    public static void main(String[] args) {

        Test6[] test6s = new Test6[1];
        System.out.println(test6s.getClass());
        System.out.println("=======================");
        Test6 test6 = new Test6();
        System.out.println(test6.getClass());

    }
}
class Test6{
    static {
        System.out.println("Test6 static block!");
    }
}
```

可以看到`new` 一个`Test6`类型的数组并没有使`Test6`进行初始化，可以看到它动态的生成了一个`Lcom.xxx`并且还是一个数组类型，一个`[`代表一重数组。

```Java
> Task :Test5.main()
class [Lcom.wrial.jvm.classLoader.Test6;
=======================
Test6 static block!
class com.wrial.jvm.classLoader.Test6
```

```Java
package com.wrial.jvm.classLoader;
/*
 * @Author  Wrial
 * @Date Created in 11:01 2019/8/19
 * @Description
 */

public class Test5 {

    public static void main(String[] args) {

        Test6[] test6s = new Test6[1];
        System.out.println(test6s.getClass()); //class [Lcom.wrial.jvm.classLoader.Test6;
        System.out.println("=======================");
        Test6 test6 = new Test6();
        System.out.println(test6.getClass());  //class com.wrial.jvm.classLoader.Test6
        System.out.println("======================");
        Test6[][] test6s1 = new Test6[1][1];
        System.out.println(test6s1.getClass()); //class [[Lcom.wrial.jvm.classLoader.Test6;
        int[] ints = new int[1];
        System.out.println(ints.getClass());  //class [I
        short[] shorts = new short[1];
        System.out.println(shorts.getClass());  //class [S
        long[] longs = new long[1];
        System.out.println(longs.getClass());  //class [J
        boolean[] booleans = new boolean[1];
        System.out.println(booleans.getClass()); //class [Z
        byte[] bytes = new byte[1];
        System.out.println(bytes.getClass()); //class [B
        Byte[] bytes1 = new Byte[1];
        System.out.println(bytes1.getClass()); //class [Ljava.lang.Byte;


    }
}
class Test6{
    static {
        System.out.println("Test6 static block!");
    }
}
```

接下来再使用Javap来看看是怎么实现的

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190819121640665.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190819121741858.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190819121845499.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

通过上边这个例子我们又可以了解到一些关于数组的助记符

#### new数组助记符

* `anewarray`：表示创建的是一个引用类型（可以是类类型的，接口类型，数组类型）的数组，并将其引用压入栈顶
* `newarray`：表示创建一个指定的基本类型（如int，boolean等）的数组，并将其压入栈顶

#### 接口初始化案例

```Java
package com.wrial.jvm.classLoader;
public class Test7 {

    public static void main(String[] args) {
        System.out.println(Child7.a);
    }
}

interface Parent7{
    int a = 6;

}
interface Child7 extends Parent7{
    int b = 4;
}

```

反编译出的字节码

```jAVA
public class com.wrial.jvm.classLoader.Test7 {
  public com.wrial.jvm.classLoader.Test7();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
       3: bipush        6
       5: invokevirtual #4                  // Method java/io/PrintStream.println:(I)V
       8: return
}

//可以看到JVM并没有去初始化Child7，这就说明这个值是常量
```

或者直接看class字节码文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190819130639264.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

​	

下面改为生成随机数，因为随机数是在运行期间完成的，所有`Child7`必须初始化

```Java
package com.wrial.jvm.classLoader;

import java.util.Random;

public class Test7 {

    public static void main(String[] args) {
        System.out.println(Child7.b);
    }
}

interface Parent7{
    int a = 6;

}
interface Child7 extends Parent7{
    int b = new Random().nextInt();
}

```

因此这次的b并不是一个常量，很明显可以在字节码中看到

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190819130932101.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

也可以反编译看看，发现`getstatic`的助记符，说明是一个静态变量而不是常量。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190819131119797.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)





结论：当一个接口在初始化的时候，并不要求其父接口完成初始化，只有真正使用到父接口里的常量的时候才会被初始化，根本原因就是接口中的中所有的变量都是`public static final`的，都可以说是常量。因此不算是直接使用。

#### 单例静态案例

```Java
package com.wrial.jvm.classLoader;
/*
 * @Author  Wrial
 * @Date Created in 15:19 2019/8/19
 * @Description
 */


public class Test8 {

    public static void main(String[] args) {
        Singleton.getInstance();
    }
}

class Singleton {

    public static int i;
    private static Singleton singleton = new Singleton();

    private Singleton() {
        i++;
        j++;
        System.out.println(i + "   " + j);

    }

    public static int j = 0;

    public static Singleton getInstance() {
        System.out.println(i + "   " + j);
        return singleton;
    }
}

```

```Java
> Task :Test8.main()
1   1
1   0
```

为什么两次输出的结果是不同的？

因为调用了静态方法而使得类被加载，连接，连接结束后i，j都是0，紧接着就是静态代码初始化阶段，由代码块从上到下，先执行构造方法，i++，j++，在构造方法后，又给j重新赋值为0，所以第二次输出的j是0

因此我们可以大胆的猜测，如果把 `public static int j = 0;`	提前，就会一致。

```Java
package com.wrial.jvm.classLoader;
/*
 * @Author  Wrial
 * @Date Created in 15:19 2019/8/19
 * @Description
 */


public class Test8 {

    public static void main(String[] args) {
        Singleton.getInstance();
    }
}

class Singleton {

 
    private static Singleton singleton = new Singleton();
    public static int i;
    public static int j = 0;
  

    private Singleton() {
        i++;
        j++;
        System.out.println(i + "   " + j);

    }
    public static Singleton getInstance() {
        System.out.println(i + "   " + j);
        return singleton;
    }
}

```

把j的赋值提前到构造方法之前，那么这个结果是什么呢？

```Java
> Task :Test8.main()
1   1
1   0   //instacne方法中
```

为什么还是一样呢？不是已经放在构造方法之上了吗？当然不对了，虽然在构造方法之上，但是可以看到它是在new Singleton之下的，先进行了构造方法的初始化等等，等对象初始化结束之后，i和j的值就都是1，但是Singleton类初始化并未结束，然后根据代码给变量赋值，又因为是static的，所以对象里面的j值也就改变了。

因此，要改好这个问题，必须把静态变量的赋值放在new Singleton前面

```Java
package com.wrial.jvm.classLoader;
/*
 * @Author  Wrial
 * @Date Created in 15:19 2019/8/19
 * @Description
 */


public class Test8 {

    public static void main(String[] args) {
        Singleton.getInstance();
    }
}

class Singleton {

    public static int j = 0;
    private static Singleton singleton = new Singleton();
    public static int i;

    private Singleton() {
        i++;
        j++;
        System.out.println(i + "   " + j);

    }
    public static Singleton getInstance() {
        System.out.println("instance  "+i + "   " + j);
        return singleton;
    }
}

```

```Java
> Task :Test8.main()
1   1
instance  1   1
```

执行过程

1. `getInstance`而使得`Singleton`类被加载，连接和初始化
2. `j `，`i `在连接过程中被赋值为`0`，`singleton`被赋值为`null`，并给类变量分配内存。
3. 初始化阶段1——> j=0
4. 初始化阶段2——>执行`SIngleton`构造方法（i=1，j=1）
5. 初始化阶段3——> i不变（为1）
6. 初始化阶段4——> `getInstace`返回Singleton实例

由此看来，关于类加载这方面也不是一个简单的事情。

到这里应该对这个例子掌握的很透彻了，下面这个进行测试

```Java
package com.wrial.jvm.classLoader;
public class Test8 {

    public static void main(String[] args) {
        Singleton.getInstance();
    }
}

class Singleton {

    public static int j = 1;
    private static Singleton singleton = new Singleton();
    public static int i = 0;

    private Singleton() {
        i++;
        j++;
        System.out.println(i + "   " + j);

    }
    public static Singleton getInstance() {
        System.out.println("instance  "+i + "   " + j);
        return singleton;
    }
}

```

```Java
> Task :Test8.main()
1   2
instance  0   2
```

#### 过程图

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190819161130860.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)



> static块和普通程序块的区别就是，static是类加载的时候完成的，代码块是类实例化时完成的。

#### 案例



```Java
package com.wrial.jvm.classLoader;
/*
 * @Author  Wrial
 * @Date Created in 23:01 2019/8/27
 * @Description
 */

public class Test10 {

    public static void main(String[] args) throws ClassNotFoundException {
        ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();
        Class<?> aClass = systemClassLoader.loadClass("com.wrial.jvm.classLoader.a");
        System.out.println(aClass);
        System.out.println("-----------");
        Class<?> aClass1 = Class.forName("com.wrial.jvm.classLoader.a");
        System.out.println(aClass1);
    }

}

class a {

    static{
        System.out.println("加载a");
    }
}

```

```Java
CLassLoader只加载类，并不初始化，此例的初始化是在forName也就是实例化对象时

class com.wrial.jvm.classLoader.a
-----------
加载a
class com.wrial.jvm.classLoader.a

```





























