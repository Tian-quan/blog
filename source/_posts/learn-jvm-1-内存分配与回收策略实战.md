---
title: learn-jvm-1-内存分配与回收策略实战
date: 2017-03-08 13:21:49
tags: [Java,jvm]
categories: Java
---
# 参考资料
* [Java Garbage Collection Basics](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html#t3)
* [JVM参数释义](http://www.cnblogs.com/redcreen/archive/2011/05/04/2037057.html)

# Debug环境
笔者是在IDEA + jvm调试工具环境下调试.
* 虚拟机启动参数配置
```
-XX：+PrintGCDetails  --虚拟机在发生垃圾收集行为时打印内存回收日志，并且在进程退出的时候输出当前的内存各区域分配情况。
-Xms20M  --初始堆大小
-Xmx20M  --最大堆大小
-Xmn10M  --年轻代大小
-XX:+UseSerialGC --指定垃圾回收器为串行垃圾回收器
-XX:PretenureSizeThreshold=3145728  --超过阀值(3MB)直接分配到老年代,只对Serial和ParNew两款收集器有效
```
* 常用调试命令

```
查看虚拟机唯一ID(Local Virtual Machine Identifier,LVMID),通常情况下与进程ID是一个值
> jps
```

```
查看堆栈信息
> jmap
> jmap -heap $LVMID  //查看堆内存
> jmap -dump:format=b,file=xx.bin  $LVMID  //生成dump文件, 可用jhat命令分析生成的内存镜像文件
```

```
查看堆栈内存变化信息
> jstat
> jstat -gcutil -h 20 $LVMID 500 1000     // (每隔20行显示表头,每隔500ms显示一次,一共显示1000次)
option 可选
    -gcnew
    -gcnewcapacity
    -gcold
    -gcoldcapacity
    -gcpermcapacity
    -gcutil
表头含义(https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstat.html)
-gcutil option
Summary of garbage collection statistics.
S0: Survivor space 0 utilization as a percentage of the space's current capacity.
S1: Survivor space 1 utilization as a percentage of the space's current capacity.
E: Eden space utilization as a percentage of the space's current capacity.
O: Old space utilization as a percentage of the space's current capacity.
M: Metaspace utilization as a percentage of the space's current capacity.
CCS: Compressed class space utilization as a percentage.
YGC: Number of young generation GC events.
YGCT: Young generation garbage collection time.
FGC: Number of full GC events.
FGCT: Full garbage collection time.
GCT: Total garbage collection time.
```

# 内存分配与回收策略

## 对象优先在Eden分配
```java
public class TestAllocation {
    public static int _1MB = 1024 * 1024;
    /**
     * 测试垃圾回收
     * jvm添加参数:-Xms20M -Xmx20M -Xmn10M -XX:+PrintGCDetails
     * -Xms20M  --初始堆大小
     * -Xmx20M  --最大堆大小
     * -Xmn10M  --年轻代大小
     * 这3个参数限制了Java堆大小为20MB,不可扩展,其中10MB分配给新生代,剩下的10MB分配给老年代。
     * -XX:SurvivorRatio=8  决定了新生代中Eden区与一个Survivor区的空间比例是8:1
     * -XX:+PrintGCDetails  告诉虚拟机在发生垃圾收集行为时打印内存回收日志，并且在进程退出的时候输出当前的内存各区域分配情况。
     */
    public static void test_3_6_1(){
        byte[] a1 = new byte[2 * _1MB];
        byte[] a2 = new byte[2 * _1MB];
        byte[] a3 = new byte[2 * _1MB];
        byte[] a4 = new byte[_1MB >> 1];
        a1 = null;
        a2 = null;
        a3 = null;
        byte[] a5 = new byte[_1MB >> 2];//第一次发生Minor GC (YGC)
    }
    public static void main(String[] args){
        test_3_6_1();
    }
}
```
调试到第一次发生YGC的地方,GC导致的内存占用变化如下图所示.
![YGC](/img/learn-jvm-1/firstYGC.png)

## 直接分配大对象到老年代
```java
public class Test_3_6_2 {
    public static int _1MB = 1024 * 1024;
    /**
     * 测试直接分配大对象到老年代
     * jvm添加参数:-Xms20M -Xmx20M -Xmn10M -XX:+PrintGCDetails -XX:+UseSerialGC -XX:PretenureSizeThreshold=3145728
     * -Xms20M  --初始堆大小
     * -Xmx20M  --最大堆大小
     * -Xmn10M  --年轻代大小
     * -XX:+UseSerialGC --指定垃圾回收器为串行垃圾回收器
     * -XX:PretenureSizeThreshold=3145728  --超过阀值(3MB)直接分配到老年代,只对Serial和ParNew两款收集器有效
     */
    public static void test_3_6_2(){
        byte[] b = new byte[4* _1MB]; //直接分配到老年代
    }
    public static void main(String[] args){
        test_3_6_2();
    }
}
```
内存占用变化如下图所示.
![YGC](/img/learn-jvm-1/allocateToOldGeneration.png)

## 长期存活的对象进入老年代
```java
public class Test_3_6_3 {
    public static int _1MB = 1024 * 1024;
    /**
     * 测试长期存活的对象进入老年代
     * jvm添加参数:-Xms20M -Xmx20M -Xmn10M -XX:+PrintGCDetails -XX:MaxTenuringThreshold=1 -XX:+PrintTenuringDistribution
     * -Xms20M  --初始堆大小
     * -Xmx20M  --最大堆大小
     * -Xmn10M  --年轻代大小
     * -XX:MaxTenuringThreshold=1 -- 对象转移到老年代阀值(每次垃圾回收对象age+1,age大于阀值时对象被移到老年代中)
     * -XX:+PrintTenuringDistribution
     */
    public static void test_3_6_3(){
        byte[] a1 = new byte[_1MB >> 2];
        byte[] a2 = new byte[2 * _1MB];
        byte[] a3 = new byte[2 * _1MB];
        byte[] a4 = new byte[2 * _1MB];
        a2 = null;
        a3 = null;
        a4 = null;

        a2 = new byte[2 * _1MB];//第1次YGC
        a3 = new byte[2 * _1MB];
        a4 = new byte[2 * _1MB];
        a2 = null;
        a3 = null;
        a4 = null;

        a2 = new byte[2 * _1MB];//第2次YGC,a1被移到老年代
    }
    public static void main(String[] args){
        test_3_6_3();
    }
}
```
内存占用变化如下图所示.
![ageToOld](/img/learn-jvm-1/ageToOldGenerationMemory.png)

第2次YDC后堆内存信息
![ageToOld](/img/learn-jvm-1/ageToOldGenerationHeap.png)


## 动态对象年龄判定
```java
public class Test_3_6_4 {
    public static int _1MB = 1024 * 1024;
    /**
     * 测试动态对象年龄判定
     *
     * 为了能更好地适应不同程序的内存状况，
     * 虚拟机并不是永远地要求对象的年龄必须达到了MaxTenuringThreshold才能晋升老年代，
     * 如果在Survivor空间中相同年龄所有对象大小的总和大于Survivor空间的一半，
     * 年龄大于或等于该年龄的对象就可以直接进入老年代，
     * 无须等到MaxTenuringThreshold中要求的年龄。
     *
     * jvm添加参数:-Xms20M -Xmx20M -Xmn10M -XX:+PrintGCDetails -XX:MaxTenuringThreshold=15 -XX:+PrintTenuringDistribution
     * -Xms20M  --初始堆大小
     * -Xmx20M  --最大堆大小
     * -Xmn10M  --年轻代大小
     * -XX:MaxTenuringThreshold=1 -- 对象转移到老年代阀值(每次垃圾回收对象age+1,age大于阀值时对象被移到老年代中)
     * -XX:+PrintTenuringDistribution
     */
    public static void test_3_6_4(){
        byte[] a1 = new byte[_1MB >> 2];
        byte[] a2 = new byte[_1MB >> 2];//a1+a2大于survivo空间一半
        byte[] a3 = new byte[3 * _1MB];
        byte[] a4 = new byte[3 * _1MB];
        a4 = null;
        byte[] a5 = new byte[2 * _1MB];//此时发生YGC, a1+a2 --> S0, a3 --> old
    }
    public static void main(String[] args){
        test_3_6_4();
    }
}
```
内存占用变化如下图所示.
![ageToOld](/img/learn-jvm-1/maxTenuringThreshold.png)


## 空间分配担保
```java
public class Test_3_6_5 {
    public static int _1MB = 1024 * 1024;
    /**
     * 测试空间分配担保
     *
     * 当出现大量对象在Minor GC后仍然存活的情况（最极端的情况就是内存回收后新生代中所有对象都存活），
     * 就需要老年代进行分配担保，把Survivor无法容纳的对象直接进入老年代.
     *
     * jvm添加参数:-Xms20M -Xmx20M -Xmn10M -XX:+PrintGCDetails
     * -Xms20M  --初始堆大小
     * -Xmx20M  --最大堆大小
     * -Xmn10M  --年轻代大小
     */
    public static void test_3_6_5(){
        byte[] a1 = new byte[ 4 * _1MB ];
        byte[] a2 = new byte[ 4 * _1MB ];
        byte[] a3 = new byte[ 3 * _1MB];//没有GC, 直接在老年代分配空间给a3
    }
    public static void main(String[] args){
        test_3_6_5();
    }
}
```
内存占用变化如下图所示.
![ageToOld](/img/learn-jvm-1/allocateToOld.png)