## 类文件结构

### Java平台无关

Java虚拟机不单单只能运行Java程序，它可以运行如Jyphon，Scala，JRuby，Groovy等等众多语言，都可以编译为字节码并放在Java虚拟机中执行。

![加粗样式](https://img-blog.csdnimg.cn/20190903184345449.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

### Class文件内容

![](https://img-blog.csdnimg.cn/20190903184647420.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)



Class文件格式

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190903185127150.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)



### u1—u4

* u1：1个字节
* u2：两个字节
* u3：三个字节
* u4：四个字节

### 字节码的具体解释

* 第一个u4字节代表的是一个class文件，是固定的cafebabe
* 第二个u4字节代表JDK编译的版本号（前两个是次版本号，后两个是主版本号），Java的版本号是从45开始的，没发布一个版本就+1，比如JDK1.1—>45，JDK1.7—>41
* 接下来的u2是常量池计数器，代表常量池容量
* 接下来就是Class文件中最多的一部分，就是常量池cp_info是多个xxx_info的集合

> 无符号数：就是数值
>
> 元数据（metaData）：描述类的数据
>
> 比如SQL中一个table的元数据，就是定义table时的所有规定
>
> 表：类似一个数据结构
>
> xxxx_info{
>
> xxxx
>
> }
>
> 字面量： int m = 5;  字面量就是5，就是等号右边的东西
>
> 符号引用：符号引用包含下面三部分，类和接口的全路径名，字段名称和描述符，方法名称和描述符
>
> ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190903212930706.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

* 

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190904194437700.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190904185556914.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

* 紧接着常量池两个字节代表的是访问标志

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/2019090419275714.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190904204345800.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)