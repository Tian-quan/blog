---
title: learn-jvm-2-BTrace初探
date: 2017-03-11 14:21:49
tags: [Java,jvm]
categories: Java
---

# 参考资料
* [Btrace入门到熟练小工完全指南](http://calvin1978.blogcn.com/articles/btrace1.html)
* [git - BTrace](https://github.com/btraceio/btrace)

# 环境准备
* jdk 1.8
* jvisualvm 安装BTrace插件
* Atom 用来写BTrace脚本,别用写字板写
* BTrace的基本知识参考[Btrace入门到熟练小工完全指南](http://calvin1978.blogcn.com/articles/btrace1.html)

# 计算方法调用过程时间
利用@Duration来获取方法调用时间, 顺便验证Array和HashMap在较少量数据下,哪个容器查找更快.
## 测试代码
```java
public class BtraceDuration {
    private static final int size = 400;
    private static Map<Integer, Integer> map = new HashMap<>();
    private static int[] arr = new int[size];

    static {
        for (int i = 0; i < size; i++) {
            map.put(i, i);
            arr[i] = i;
        }
    }

    public static boolean findInMap(int v) {
        return map.get(v) != null;
    }

    public static boolean findInArr(int v) {
        for (int i : arr) {
            if (i == v) return true;
        }
        return false;
    }

    public static void main(String[] args) throws IOException {
        Scanner scanner = new Scanner(System.in);
        int input;
        while (true) {
            input = scanner.nextInt(); // 用于阻塞程序,便于观察
            findInArr(input);
            findInMap(input);
        }
    }
}
```

## BTrace代码
```java
import com.sun.btrace.annotations.*;
import static com.sun.btrace.BTraceUtils.*;

@BTrace
public class TracingScript {

@OnMethod(clazz = "/cn.stq.learn.jvm.btrace\\..*/", method = "/find.*/", location = @Location(Kind.RETURN))
public static void testDuration(@ProbeClassName String pcn,@ProbeMethodName String pmn,  @Duration long duration, @Return boolean result) {
    // duration 是纳秒,除以1000000得毫秒
    print(strcat(pcn,"#"));
    print(strcat(pmn, ": "));
    print(strcat(str(duration), "ns  "));
    println(strcat("result->", str(result)));
}
}
```

## 结果
![Duration](/img/learn-jvm-2/Duration.png)

***
# 谁调用了这个函数

## 测试代码
见**_计算方法调用过程时间_**的测试代码

## BTrace代码
```java
import com.sun.btrace.annotations.*;
import static com.sun.btrace.BTraceUtils.*;

@BTrace
public class TracingScript {
  @OnMethod(clazz = "/cn.stq.learn.jvm.btrace\\..*/", method = "/find.*/")
  public static void testInvokeStack(@ProbeClassName String pcn,@ProbeMethodName String pmn, int param) {
    print(strcat(pcn,"#"));
    println(strcat(pmn, ""));
    println(strcat("param:", str(param)));
    jstack(); // 打印调用栈
  }
}
```

## 结果
![invoke stack](/img/learn-jvm-2/invoke-stack.png)

