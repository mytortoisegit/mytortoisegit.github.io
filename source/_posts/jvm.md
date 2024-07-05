---
title: jvm
date: 2024-07-01 09:03:39
tags: jvm 
---



## 什么是JVM

​	定义：JAVA Virtual Machine -java程序的运行环境（java 二进制字节码的运行环境）

​	好处：一次编译处处运行，自动内存管理，垃圾回收功能，数组下标越界 越界检查

​	程序员必备

​	java代码的执行

<img src="/images/1720060780441.png" alt="数据类型" />

## jVM内存结构

### 运行时数据区

​	jvm在运行过程中会把他所管理的内存划分成<font color="red">若干</font> 不同的 <font color="red">数据区域</font> ！

​	线程私有： 程序计数器、虚拟机栈、本地方法栈

​	线程共享：堆、方法区

<img src="/images/1719796187750.png" alt="数据类型" />

#### 线程私有区域

**程序计数器**：指向 当前线程正在执行的字节码指令的地址（行号），唯一不会oom的区域。

​		为什么要有程序计数器？

​		java是多线程的，意味着线程切换，确保多线程情况下的程序正常执行，引入程序计数器。

<img src="/images/1720061131696.png" alt="数据类型" />

​		使用寄存器实现

​		cpu时间片切换时，程序计数器记录下次执行位置，唯一一个不会出现内存溢出。

栈：数据结构，先进后出 ，

​		为什么jvm使用栈？

​		非常好的兼容 方法中调用方法

**虚拟机栈**：	-Xss 1M   除windows其他系统默认设置大小

​		1.  存储<font color="red">当前线程</font> 运行方法所需的数据，参数，局部变量，返回地址等。

​		每个线程运行时所需要的内存，成为虚拟机栈，

​		每个栈由多个栈帧组成，对应着每次方法调用时所占内存。

<img src="/images/1720061665583.png" alt="数据类型" />

​		栈演示

​			<img src="/images/1720062164666.png" alt="数据类型" />

​		

​		每个栈帧中方法运行结束，对应的内存占用也结束出栈。

​		内存划分不是越大越好，一般采用默认大小。

​		线程共享的要考虑线程安全，线程私有的不需要考虑线程安全，

​		局部变量在方法调用时，每个方法有独立的栈帧，不存在线程安全。

​		局部变量作为参数被传入时，不是线程安全的，

​		局部变量作为返回值时，不是线程安全的，

​	结论：方法内局部变量线程是否安全，要考虑变量是否脱离了方法。

​			<img src="/images/1720063500940.png" alt="数据类型" />

​	2.栈内存溢出

​		1、栈帧过多导致栈内存溢出，递归调用

​		2、栈帧过大导致栈内存溢出， Json转换时，递归转换（员工对象中有部门，部门对象中有员工）

​		异常：StackOverflowError

​	3.线程运行诊断

​		1、cpu占用过多

​			定位：top  定位哪个进程占用cpu过高

​			            ps h -eo pid,tid,%cpu |grep + 进程id   （进一步定位哪个线程引起cpu占用过高）

​				    使用java工具精准定位：

​					 jstack + 进程id，将上方查出的线程10进制 转为16进制 找到对应java类与问题行号。

​			<img src="/images/1720069759389.png" alt="数据类型" />

​		2、程序运行时间长没有结果

​				定位： jstack + 进程id   下拉到最后查看

​						<img src="/images/1720069915032.png" alt="数据类型" />

​						<img src="/images/1720070092778.png" alt="数据类型" />



**本地方法栈**：保存的是当前线程的native方法的信息，调用非java方法时，使用的内存空间



#### 线程共享的区域

##### **方法区**（永久代（JDK1.7以前叫法）、元空间（JDK1.8））

​	类信息、常量、静态变量、即时编译期编译后的代码

​                            <img src="/images/1720075919180.png" alt="数据类型" />

​	2、方法区内存溢出：

​	元空间默认使用系统的内存，很难出现内存溢出。

​	可以添加参数设置成虚拟机内存：-XX:MaxMetaspaceSize=8m

​			                            <img src="/images/1720076603576.png" alt="数据类型" />

​	场景： spring  mybatis 都使用代理cglib ，动态生成类字节码，动态生成类。

​	3、运行时常量池：

​         常量池，就是一张表，虚拟机指令根据这张常量表找到要执行的类名、方法名、参数类型、字面量等信息

​	运行时常量池，常量池在.class文件中，当类被加载时，常量池信息加到到内存中放入运行时常量池，并把里边的符号地址变为真实内存地址。

​	 **常量池在反编译后是Constant Pool区域** 

​	

​	4、StringTable （串池）特性： 1.6在永久代，1.8在堆中

​	1.串池中的字符串只是符号，第一次用到时才会变为对象。

​	2.利用串池的机制，来避免重复创建字符串对象。

​	3.字符串**变量**拼接的原理是StringBuilder(1.8)，属于动态拼接，不在串池中，而在堆中。

​	4.字符串**常量**拼接的原理是编译期优化。

​	5.可以使用intern()方法，将字符串对象尝试放入串池，有不放入，无则放入1.8。

​					        <img src="/images/1720080193678.png" alt="数据类型" />

​	6.位置StringTable位于堆内存中 。

​	7. StringTable垃圾回收

​	8.StringTable调优 :  

​			1）-XX:StringTableSize=200000 ，将StringTable 桶的大小设置为20万，存入48万数据0.4s

​					 如果系统字符串常量个数非常多，可以适量提高桶大小。

  			2）考虑将字符串对象是否入池：使用intern()方法避免重复，减少堆内存使用。

##### **直接内存** Direct Memory

- 常见于NIO操作时，用于数据缓冲区

- 分配回收成本较高，但读写性能高

- 不受jvm内存管理

- 直接内存的使用是通过Unsafe对象完成直接内存的分配与回收的，且回收需要主动调用freeMemory方法。

- 直接内存的释放是采用java中虚引用对象Cleaner来监测DirectByteBuffer，当DirectByteBuffer被回收，由ReferenceHandler线程通过Cleaner的clean方法调用 freeMemory 释放直接内存。

- 禁用显示GC时（防止程序员主动调gc()），需要手动释放直接内存时，使用Unsafe调用freeMemory释放。

  

						        <img src="/images/1720082619357.png" alt="数据类型" />

读取磁盘数据时，直接内存可以在系统与java直接建立缓冲区，java直接读取缓冲区数据，避免系统自身一块缓冲区，java一块缓冲区，速度更快。

```java

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

public class Demo1_20 {

    static final String FROM="D:\\tool\\bilbili\\download\\合集_2024-02-24_【零基础】八字命理绝学 ｜字字句句皆真传\\第二集：神煞应用.mp4";
    static final String TO="D:\\tool\\bilbili\\download";
    static final int _1MB=1024*1024;

    public static void main(String[] args) {
        io();
        directBuffer();
    }

    // 使用直接内存缓冲流方式
    private static void directBuffer() {
        long startTime = System.nanoTime();

        try(FileChannel from= new FileInputStream(FROM).getChannel();
            FileChannel to=new FileOutputStream(TO).getChannel())
        {
            ByteBuffer bf = ByteBuffer.allocateDirect(_1MB);
            while (true) {
                // 读取数据
                int len = from.read(bf);
                if (len == -1) break;
                // 管道翻转
                bf.flip();
                // 写数据
                to.write(bf);
                bf.clear();
            }
        }catch (IOException e){
            e.printStackTrace();
        }
        long endTime = System.nanoTime();
        System.out.println("directBuffer 用时"+(endTime-startTime)/1000000+"ms");


    }

    // 传统读写数据操作
    private static void io(){

        long startTime = System.nanoTime();

     try(  FileInputStream from= new FileInputStream(FROM);
           FileOutputStream to=new FileOutputStream(TO);)
     {
         byte[] buf = new byte[_1MB];
         while (true) {
             // 读取数据
             int len = from.read(buf);
             if (len == -1) break;
             // 写数据
             to.write(buf,0,len);
         }
     }catch (IOException e){
         e.printStackTrace();
     }
     long endTime = System.nanoTime();
        System.out.println("IO 用时"+(endTime-startTime)/1000000+"ms");

    }
}
```



##### **Java 堆**heap（-Xms：（初始大小）  -Xmx ：（最大）  -Xmm（新生代大小））

​	1、定义：堆是需要重点关注的一块区域，涉及到内存的分配（new 关键字，反射等）与回收（回收算法，收集器等)

​	堆是内存分配和垃圾回收的重点区域，几乎所有的<font color="red">对象</font> 都是在堆中分配。

​	堆中对象需要考虑线程安全性问题。

​	2、堆内存的诊断

​	1.jps 工具：查看当前系统中有哪些java进程；

​	2.jmap工具：查看堆内存占用情况  jmap -heap  + 进程id	

​        3.jconsole 工具：图形化工具，多功能的监测工具，可以连续监测，控制台直接打开jconsole

​	4. jvisualvm 工具：使用堆转储，查询占用内存最大的对象。

​	             <img src="/images/1720075541879.png" alt="数据类型" />



## jvm中的对象

#### 	jvm中的对象分配

<img src="/images/1719816227908.png" alt="数据类型" />



#### 对象的内存布局  

对象头的大小必须是8个字节的整数

对象头存储：hashcode值，GC垃圾回收（年龄），锁状态标志，线程持有的锁，偏向线程id,偏向时间戳等等。

#### <img src="/images/1719817196796.png" alt="数据类型" />对象的访问方式

<img src="/images/1719817531364.png" alt="数据类型" />



#### 堆内存分配策略

​	堆进一步划分

​		新生代（PSYong Gen）

​			Eden   空间

​			From  survivor空间

​			To  survivor

​		老年代（ParOld Gen）

​	堆中参数设置：

​	新生代大小： -Xmn20m 表示新生代大小20M(初始和最大)

​	-XX:SurvivorRatio=8 表示Eden和Survivor的比值，缺省为8，表示Eden:From:To  =8:1:1

​	-XX:SurvivorRatio=2  表示 Eden:From:To  =2 :1 : 1

<img src="/images/1719818697685.png" alt="数据类型" />

###### 1.**对象优先在Eden分配**

###### 2.大对象直接进入老年代

<img src="/images/1719820717422.png" alt="数据类型" />

###### 3.长期存活的对象将进入老年代

​		GC通过15次后进入老年代

###### 4.动态对象年龄判断

​		From区俩个对象内存大小占到一半，年龄达到5时，直接进入老年代

###### 5.空间分配担保

​		HandlePromotionFailure：确保每次放入老年代不会发生GC

## 垃圾回收

#### 1.如何判断对象可以回收

- 可达性分析算法：Java的主要垃圾收集算法是基于可达性分析的方法

  ​	可达性分析从一组称为“GC Roots”的根对象开始，通过对象之间的引用链，标记所有能从GC Roots到达的对象（即所有被引用的对象） ，

  ```java
  List<Object> list= new LinkedList<>();  //在方法执行过程中，它是GC Roots
  ```

  ​	未被标记的对象被判定为不可达（即没有被任何活跃对象引用），即将被判定为可回收。 

  在可达性分析法中，Java 8中的垃圾收集器会从GC Roots出发，通过以下几种类型的GC Roots进行遍历和标记 

  ​	**虚拟机栈（Stack）中的引用**：虚拟机栈（线程栈）中的本地变量引用的对象。

  ​	**方法区中静态属性引用**：方法区中静态属性引用的对象。

  ​	**方法区中常量引用**：方法区中常量引用的对象。

  ​	**本地方法栈中JNI（Native方法）引用**：本地方法栈中JNI引用的对象。

- 各种引用

  ​	强引用   比如 User user=new User()

  

  ​	软引用	通过 `SoftReference` 类来实现 （内存在oom时会被回收）：内存不足时被回收

  

  ​	弱引用	通过 `WeakReference` 类来实现  （每一次GC都会被回收）

  

  ​	虚引用	通过 `PhantomReference` 类来实现 ，主要用于跟踪对象被回收的状态 

  

#### 2.垃圾回收算法

#### GC

​	什么时候会发生GC? 当内存不足的时候会发生GC

​	Minor GC ( 发生在PSyoungGen)

​	Full GC( 发生在 PSyoungGen、PSOldGen、Metaspace)

#### 算法

##### 	复制算法（copying）

​		优点：简单高效，不会出现碎片化问题

​		缺点：内存效率使用低，只有一半，存活对象比较多时，效率明显下降。

<img src="/images/1719999589083.png" alt="数据类型" />

​	在新生代 From区和To区 执行： 最初在From区存储对象，当通过可达性分析后，将存活对象放到To区格式化From区，当To区GC时，同理，通过可达性分析后，将存活对象放入From区。

Java中大部分对象98%是不需要回收的，因为是朝生夕死。新生代默认8：1：1    From区与To区 预留内存的20%作为存活对象保留挪来挪去。

##### 	标记-清除算法（Mark-Sweep)

​		标记一遍，清除一遍，

​		优点：利用率100% ，速度快，

​		缺点：标记和清除效率都不高，会产生大量的不连续的内存片段。

<img src="/images/1720000234527.png" alt="数据类型" />

##### 	标记-整理算法（Mark-Compact）

​		标记一遍，整理移动一次，在俩个区移动，

​		优点：利用率100%，没有内存碎片

​		缺点：效率低，要挪动内存

<img src="/images/1720000531563.png" alt="数据类型" />



#### 3.分代垃圾回收

​	分代收集：分为新生代和老年代，同时有单线程与多线程



- 新生代：新生代内存不足时，触发minor GC，Eden和From区存活对象，复制算法到To区，存活对象年龄加1，之后交换From与To。     
- minor GC会引发 stop the world 暂停其他用户线程，等垃圾回收结束，用户线程才会恢复
- 当对象寿命超过阈值时，会晋升至老年代，最大寿命15（4bit）
- 当老年代内存不足时，会先尝试minor gc，如果空间仍然不足，会触发full gc  STW时更长。 
- JVM相关参数：
  1. 堆初始大小    -Xms
  2. 堆最大大小 -Xmx 或-XX:MaxHeapSize=size
  3. 新生代大小：-Xmn或(-XX:NewSize=size+-XX:MaxNewSize=size )
  4. 幸存区比例（动态）：-XX:InitialSurvivorRatio=ratio 和-XX:+UseAdaptiveSizePolicy
  5. 幸存区比例：-XX:SurvivorRatio=ratio
  6. 晋升阈值：-XX:MaxTenuringThreshold=threshold
  7. 晋升详情：-XX:+PrintTenuringDistribution
  8. GC详情： -XX:+PrintGCDetails -verbose:gc
  9. FullGC前MinorGC: -XX:+ScavengeBeforeFullGC

​	



#### 4.垃圾回收器

​	在JDK 1.8中，默认的垃圾回收器是**Parallel GC** 

- 串行

  1. 单线程  
  2. 堆内存较小，适合个人电脑

- 吞吐量优先

  1. 多线程
  2. 堆内存较大，多核CPU
  3. 让单位时间内，STW的时间最短0.2 0.2 = 0.4

- 响应时间优先

  1. 多线程
  2. 堆内存较大，多核CPU
  3. 尽可能让单次STW的时间最短 0.1 0.1 0.1 0.1 0.1=0.5

- CMS垃圾回收器

  适用于响应时间要求较高的应用程序，尤其是那些需要提供较低延迟的交互式服务 	

  常用指令：-XX:+UseConcMarkSweepGC     -XX:+UseParNewGc            Serial0ld

  ​	 	//分别用于配置并行和并发垃圾回收线程的数量 

  ​		   -XX:ParallelGCThreads=n ~    -XX:ConcGCThreads=threads  

  ​		   -XX:CMSInitiatingOccupancyFraction=n%  //当老年代内存使用达到N%时开始CMS回收 

  ​		   -XX:+CMSScavengeBeforeRemark    // 在Full GC前先处理一次新生代回收

  流程顺序示意

  ​		<img src="/images/1720175269542.png" alt="数据类型" />

  

- G1垃圾回收器

  Garbage First   jdk1.9

  1. 同时注重吞吐量、低延时，默认暂停目标是200ms

  2. 超大堆内存，会将堆划分为多个大小相等的Region

  3. 整体上是标记+整理算法，俩个区域直接是复制算法

  4. 相关参数： -XX: +UserG1GC       //使用G1开关

     ​		    -XX: G1HeapRegionSize=size    //设置Region大小

     ​		    -XX: MaxGCPauseMillis=time	  //

  5. 回收阶段

     1. 第一阶段

     - 在分配大量内存相等空间后，首先存入eden区

     <img src="/images/1720162801095.png" alt="数据类型" />

     - 将幸存对象放入幸存区

       <img src="/images/1720162915830.png" alt="数据类型" />

     - 再次回收，将对象放入老年代

       <img src="/images/1720162972603.png" alt="数据类型" />

     

     1. 第二阶段

     <img src="/images/1720163117658.png" alt="数据类型" />

     

     1. 第三阶段 混合收集阶段

        

        <img src="/images/1720163248182.png" alt="数据类型" />

        

  6. JDK 8u20字符串去重优化

     

     

     

     

     

     

#### 5.垃圾回收调优

- 掌握GC相关的VM参数，会基本的空间调整

- 掌握相关工具

- 明白一点：调优跟应用，环境有关，

  1. 查看虚拟机运行参数

     "C:\Program Files\Java\jdk1.8.0_281\bin\java" -XX:+PrintFlagsFinal -version | findstr "GC"

     

<img src="/images/1720168443901.png" alt="数据类型" />

1.   调优领域

   - 内存
   - 锁竞争
   - cpu占用
   - io

2. 确定目标

   - 低延迟还是高吞吐量，选择合适的回收器
   - CMS、G1、ZGC：低延迟
   - ParallelGC：高吞吐量

3. 最快的GC是不发生GC

   - 查看FullGC前后的内存占用，考虑以下问题
     - 数据是不是太多？ 代码问题？ select * from t  
     - 数据表是否太臃肿？对象图，对象大小  返回全部结果集
     - 是否存在内存泄漏？  

4. 新生代调优

   - 新生代的特点

     - 所有的new 操作的内存分配非常廉价：TLAB thread-local allocation buffer

     - 死亡对象的回收代价是零

     - 大部分对象用过即死

     - Minor GC的时间远远低于Full GC

       不是越大调的越好，复制算法，越大导致时间越长

   - 大小选择

     - 新生代能容纳所有【并发量*（请求-响应）】的数量   

       ​	一次请求响应过程中占内存512k，并发量1000，新生代理想内存=512M

       ​	一次请求加并发量不超过新生代内存，不会或较少触发垃圾回收。

     - 幸存区大到能保留或容纳【 当前活跃对象+需要晋升对象】

     - 晋升阈值配置得当，让长时间存活对象尽快晋升

       - -XX:MaxTenuringThreshold=threshold    设置晋升阈值
       - -XX:+PrintTenuringDistribution    打印相关信息

5. 老年代调优

   以CMS为例

   - CMS老年代越大越好

   - 先尝试不做调优，如果没有Full GC ，否则先尝试调优新生代

   - 观察发生Full GC时老年代内存占用，将老年代内存预设调大 1/4 ~1/3

     - 参数指定在老年代（Old Generation）内存占用达到百分之多少时，开始触发CMS垃圾回收 

     - -XX:CMSInitiatingOccupancyFraction=percent   一般75~80%之间

       

​	**所有新生代的算法都是复制算法**

​	**在老年代中 CMS收集器使用标记-清除算法**

<img src="/images/1720001071888.png" alt="数据类型" />

​	

<img src="/images/1720001281078.png" alt="数据类型" />



<img src="/images/1720001758269.png" alt="数据类型" />

GMS垃圾回收器示意图

<img src="/images/1720002046211.png" alt="数据类型" />



G1垃圾回收器

​		<img src="/images/1720004007340.png" alt="数据类型" />



stop the world现象

​		<img src="/images/1720014966667.png" alt="数据类型" />

















