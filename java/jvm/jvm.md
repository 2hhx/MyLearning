# JVM内存结构

## JVM 内存模型

　　根据 JVM 规范，JVM 内存共分为虚拟机栈、堆、方法区、程序计数器、本地方法栈五个部分。

![img](https://images2015.cnblogs.com/blog/820406/201603/820406-20160326200119386-756216654.png)

**1、虚拟机栈**：**每个线程有一个私有的栈，随着线程的创建而创建。**栈里面存着的是一种叫“栈帧”的东西，**每个方法会创建一个栈帧**，栈帧中存放了局部变量表（基本数据类型和对象引用）、操作数栈、方法出口等信息。

**2、堆被分为：**

Eden区 —— 新对象或者生命周期很短的对象会存储在这个区域中，这个区的大小可以通过-XX:NewSize和-XX:MaxNewSize参数来调整。新生代GC（垃圾回收器）会清理这一区域。
 Survivor区 —— 那些历经了Eden区的垃圾回收仍能存活下来的依旧存在引用的对象会待在这个区域。这个区的大小可以由JVM参数-XX:SurvivorRatio来进行调节。
 老年代 —— 那些在历经了Eden区和Survivor区的多次GC后仍然存活下来的对象（当然了，是拜那些挥之不去的引用所赐）会存储在这个区里。这个区会由一个特殊的垃圾回收器来负责。年老代中的对象的回收是由老年代的GC（major GC）来进行的。

**3、方法区**用于存放已被虚拟机加载的**类信息、常量、静态变量，即编译器编译后的代码**。

- JDK1.7开始了方法区的部分移除：**符号引用(Symbols)**移至**native heap**，**字面量(interned strings)**和**静态变量(class statics)**移至**java heap。** 元空间中只放类的元信息。

## 永久代和元空间

**方法区是JVM的规范，具体实现为永久代和元空间。并且只有hotpot虚拟机才有永久代，其他虚拟机没有永久代**

由于方法区主要存储类的相关信息，所以对于动态生成类的情况比较容易出现永久代的内存溢出。**JDK 1.6下，会出现“PermGen Space”的内存溢出，而在 JDK 1.7和 JDK 1.8 中，会出现堆内存溢出**，并且 JDK 1.8中 PermSize 和 MaxPermGen 已经无效。 **JDK 1.7 以后将字符串常量由永久代转移到堆中，并且 JDK 1.8 中已经不存在永久代的结论。**

**元空间与永久代都是对jvm方法区的实现，它们之间最大的区别在于：元空间并不在虚拟机中，而是使用本地内存。**

在元空间中保存的数据比永久代中纯粹很多，就是类的元数据，这些信息只对编译期或JVM的运行时有用。

## 字符常量池和静态变量

在JDK6.0及之前版本，字符串常量池和静态变量是放在Perm Gen区(也就是方法区)中；
在JDK7.0版本，字符串常量池和静态变量被移到了堆中了。

## 类的加载过程

1. 加载: 查找并加载类的二进制数据(Class文件)
   - 把类的信息放到方法区
   - 堆：Class文件对应的类实例
2. 验证：确保加载的类信息是正确的 
3. 准备：为类的静态变量进行初始化，分配空间并赋予初始值（int=0，引用类型=null）
4. 解析：将符号引用转换为直接引用：如下，a只是一个符号，要将该符号指向一个引用地址

```
Animal a = new Cat();
a.sleep();
a = new Dog();
 a.sleep();
a = new Tiger();
 a.sleep();
```

5. 初始化：JVM堆类进行初始化，对静态变量赋予正确值（在准备阶段只是开辟空间并赋予初始值）
   - 静态代码块用于类的初始化

## 类加载器

1. APPClassLoader(加载类路径下面的类)
2. ExtClassLoader（APPClassLoader.getParent，加载JDK/JRE/LIB下的javax.类）
3. BootStrapClassLoader（C语言写的，ExtClassLoader.getParent加载JDK/JRE/LIB下的java.类）
4. 用户自定义类加载器

双亲委派模型：两个过程，一次向上，一次向下。一个类加载器收到了加载请求，首先自己不加载这个类而是把请求委托给他的父亲，依次向上到顶层，父加载器没加载到这个类才会向下委派子加载器加载该类。

## 强引用、弱引用、软引用

强引用：不会被GC，Heap内存空间不足，Java虚拟机宁愿抛出`OutOfMemoryError`错误，使程序异常终止，也不会靠随意回收具有强引用 对象来解决内存不足的问题。

软引用：还有用但并非必须的对象。内存充足，`GC`时就保留，内存不够，`GC`再来收集

弱引用：一旦GC就会被回收

## 垃圾收集算法

### 哪些要GC，哪些不要GC

- 引用计数法（被引用次数为0视为垃圾，但存在循环引用问题）
- 可达性分析法（什么是GC ROOT）
  - 虚拟机栈中引用的对象
  - 方法区中的常量和静态变量
  - 本地方法栈中的对象

```
每个中括号代表一个对象，GC ROOT是不会被回收的，如果被它引用(可达，就不会被回收)，如果像右边部分游离在之外不可达GC ROOT，计算互相之间有引用也会被回收  
----------------------[GC ROOT]--------------------
                    /
                 []         []
               /   \          \
             []     []         []
----------------------------------------------------             
```





**GC算法Garbage Collection**

- 标记-清除（把要回收的垃圾标记，再清楚--会产生内存碎片）
- 复制（有一半空间被浪费，将要回收的垃圾标记，将一半内存中不是垃圾的对象复制到空的内存中，再将原来耗尽的内存清空）
- 标记-整理（将垃圾标记，进行整理放到不回收的内存后面，空的内存之前，再将垃圾清除，增加空的内存）【相当于内存整理了】
- 分代回收算法（算法的集合）
  - 新生代（存活时间很短）：复制
  - 老年代（从新生代复制过来的） 

### 分代GC：MinorGC、FullGC

 从年轻代空间（包括 Eden 和 Survivor 区域）回收内存被称为 Minor GC。Full GC 是清理整个堆空间—包括年轻代和老年代。

**堆被分解为较小的三个部分。具体分为：新生代[Eden：S0：S1=8：1：1]、老年代。**



![img](https://pic1.zhimg.com/80/v2-c2a3154a8d2837ed46642676f051bbb8_720w.jpg)

- **MinorGC的过程：采用复制算法。**
  1. 首先，把Eden和S0区域中存活的对象复制到S1区域（**如果有对象的年龄以及达到了老年的标准，一般是15，则复制到老年代区或者大对象也会直接放到老年代**）
  2. 同时把这些对象的年龄+1（如果S1不够位置了就放到老年区）
  3. 然后，清空Eden和S0中的对象；最后，S1和S0互换，原S1成为下一次GC时的S0区。
- 老年代：老年代因为对象的存活率高（复制的代价就要高），也没有担保空间，所以采用**标记清除(CMS回收器)或标记整理算法(G1回收器)**
  - CMS收集：
    1. stop the world初步标记（初步标记 GCROOT直接关联的对象）STW
    2. 并发标记（进行GCROOTS跟踪过程），可以和用户线程一起执行
    3. stop the world重新标记（2阶段的用户线程会改变堆的状态，所以要修正在2阶段产生的垃圾）STW
    4. 并发清除
- **当老年代也满了装不下的时候，就会抛出OOM（Out of Memory）异常**。

### 为什么年轻代用复制，老年代不用复制算法

**GC算法总结**

　　内存效率：复制算法 > 标记清楚算法 > 标记清楚整理算法 （时间复杂度）

　　内存整齐度：复制算法 = 标记清除整理算法 > 标记清除算法（内存碎片）

　　内存利用率：标记清除整理算法 = 标记清除算法 > 复制算法

**年轻代需要频繁GC，对象存活率极低，又有老年代担保，复制算法效率高，没有内存碎片。**

老年代GC频率低，对象存活率较高又没有没有额外的空间分配对它进行担保，不能使用复制算法。

### CMS回收器：标记-清除

CMS收集器仅作用于**老年代**的收集，**是基于`标记-清除算法`的**，它的运作过程分为4个步骤：

- 初始标记（CMS initial mark）【STW】
- 并发标记（CMS concurrent mark）
- 重新标记（CMS remark）【STW】
- 并发清除（CMS concurrent sweep）

CMS回收器（concurrent-mark-sweep）：这个算法使用了**多个线程**（concurrent）来**扫描堆并标记**（mark）那些不再使用的可以**回收(sweep)的对象**。这个算法在两种情况下会进入一个**”stop the world”**的模式：

- 当进行根对象的初始标记的时候 （老生代中线程入口点或静态变量可达的那些对象）
- 当这个算法在并发运行的时候应用程序改变了堆的状态使得它不得不回去再次确认自己标记的对象都是正确的。

问题：

在回收新生代及年老代时出现了竞争条件的情况（后台线程还未扫描完无用对象前堆就已经用光）。如果回收器需要将年轻的对象提升到年老代中，而这个时候年老代没有多余的空间了，它就只能先进行一次STW(Stop The World)的full GC了——这种情况正是CMS所希望避免的

假设你的**堆小于4G**，而你**又希望分配更多的CPU资源以避免应用暂停**，那么这就是你要选择的回收器。

然而，如果**堆大于4G**的话，——**G1回收器。**

### G1回收器：标记-整理

但是G1同时回收老年代和年轻代，而CMS只能回收老年代，需要配合一个年轻代收集器。另外G1的分代更多是逻辑上的概念，G1将内存分成多个等大小的region，`Eden`/ `Survivor`/`Old`分别是一部分region的逻辑集合，物理上内存地址并不连续。

![img](https://upload-images.jianshu.io/upload_images/1722445-20e287ca91b9096f.png?imageMogr2/auto-orient/strip|imageView2/2/w/739)

**G1重新定义了堆空间，打破了原有的分代模型**，将堆划分为一个个区域。这么做的目的是在进行收集时不必在全堆范围内进行，这是它最显著的特点

G1（ Garbage first）：G1回收器将堆分为多个区域，大小从1MB到32MB不等，并使用多个后台线程来扫描它们。G1回收器会优先扫描那些包含垃圾最多的区域，这正是它的名字的由来（Garbage first）。

这一策略**减少了后台线程还未扫描完无用对象前堆就已经用光的可能性**，而那种情况回收器就必须得暂停应用，这就会导致STW回收。

- G1从整体来看是基于**标记-整理算法**实现的收集器，G1运作期间不会产生内存空间碎片，收集后能提供规整的可用内存
- 虽然还保留有新生代和老年代的概念，但新生代和老年代不再是物理隔离的了。