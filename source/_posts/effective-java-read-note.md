---
title: Effective java 读书笔记
date: 2017-02-09 16:41:52
tags: Java
categories: Java
---

# 目录
* chapter02 -  创建和销毁对象
    * 第1条:考虑用静态工厂方法替代构造器(Factory method)
    * 第2条:遇到多个构造器参数时,要考虑用构建器(Builder)
    * 第3条:用私有构造器或者枚举类型强化单例属性(Singleton)
    * 第4条:通过私有构造器强化不可实例化的能力
    * 第5条:避免创建不必要的对象
    * 第6条:消除过期的对象引用
    * 第7条:避免使用终结方法(finalizer)
* chapter03 - 对所有对象都通用的方法
    * 第8条:覆盖equals时请遵守通用约定
    * 第9条 覆盖equals时总要覆盖hashCode
    * 第10条 始终要覆盖toString
    * 第11条 谨慎使用clone
    * 第12条 考虑实现`Comparable`接口
* chapter04 - 类和接口
    * 第13条 使类和成员的可访问性最小化
    * 第14条 在公有类中使用访问方法而非公有域
    * 第15条 使可变性最小化
    * 第16条 复合优先于继承
    * 第17条 要么为继承而设计，并提供文档说明，要么就禁止继承
    * 第18条 接口优于抽象类
    * 第19条 接口只用于定义类型
    * 第20条 类层次优于标签类
    * 第21条 用函数对象表示策略
    * 第22条 优先考虑静态成员类
* chapter05 - 泛型
    * 第23条 请不要在新代码中使用原生态类型
    * 第24条 消除非受检警告
    * 第25条 列表优先于数组
    * 第26条 优先考虑泛型
    * 第27条 优先考虑泛型方法
    * 第28条 利用有限制通配符来提升API的灵活性
    * 第29条 优先考虑类型安全的异构容器
* chapter06 - 枚举和注解
    * 第30条 用enum代替int常量
    * 第31条 用实例域代替序数
    * 第32条 用EnumSet代替位域
    * 第33条 用EnumMap代替序数索引
    * 第34条 用接口模拟可伸缩的枚举
    * 第35条 注解优先于命名模式
    * 第36条 坚持使用Override注解
    * 第37条 用标记接口定义类型
* chapter07 - 方法
    * 第38条 检查参数的有效性
    * 第39条 必要时进行保护性拷贝
    * 第40条 谨慎设计方法签名
    * 第41条 慎用重载
    * 第42条 慎用可变参数
    * 第43条 返回零长度的数组或者集合，而不是null
    * 第44条 为所有导出的API元素编写文档注释
* chapter08 - 通用程序设计
    * 第45条 将局部变量的作用域最小化
    * 第46条 for-each循环优先于传统的for循环
    * 第47条 了解和使用类库
    * 第48条 如果需要精确的答案，请避免使用float和double
    * 第49条 基本类型优先于装箱基本类型
    * 第50条 如果其他类型更适合，则尽量避免使用字符串
    * 第51条 当心字符串连接的性能
    * 第52条 通过接口引用对象
    * 第53条 接口优先于反射机制
    * 第54条 谨慎地使用本地方法
    * 第55条 谨慎地进行优化
    * 第56条 遵守普遍接受的命名惯例
* chapter09 - 异常
    * 第57条 只针对异常的情况才使用异常
    * 第58条 对可恢复的情况使用受检异常，对编程错误使用运行时异常
    * 第59条 避免不必要地使用受检的异常
    * 第60条 优先使用标准的异常
    * 第61条 抛出与抽象相对应的异常
    * 第62条 每个方法抛出的异常都要有文档
    * 第63条 在细节消息中包含能捕获失败的信息
    * 第64条 努力使失败保持原子性
    * 第65条 不要忽略异常
* chapter10 - 并发
    * 第66条 同步访问共享的可变数据
    * 第66条 同步访问共享的可变数据
    * 第67条 避免过度同步
    * 第68条 executor和task优先于线程
    * 第69条 并发工具优先于wait和notify
    * 第70条 线程安全性的文档化
    * 第71条 慎用延迟初始化
    * 第72条 不要依赖于线程调度器
    * 第73条 避免使用线程组
* chapter11 - 序列化
    * 第74条 谨慎地实现Serializable接口
    * 第75条 考虑使用自定义的序列化形式
    * 第76条 保护性地编写readObject方法
    * 第77条 对于实例控制，枚举类型优先于readResolve
    * 第78条 考虑用序列化代理代替序列化实例

***
***
# chapter03 对所有对象都通用的方法
***
## 第8条 覆盖equals时请遵守通用约定

### 实现高质量equals方法的诀窍
* 使用==操作符检查"参数是否为这个对象的应用"
* 使用`instanceof`操作符检查"参数是否为正确的类型"
* 把参数传递转换成正确的类型
* 对于该类中的每个**关键域**,检查参数中的域是否与该对象中对应域项匹配
* 写完`equals`方法之后,应该问自己三个问题:它是否对称的,传递的,一致的?(记住`Point(x,y)`和`ColorPoint(x,y,color)`的例子)

### 覆写equals的告诫
* 覆写`equals`时总要覆写`hashCode`(见第9条)
* 不要企图让`equals`方法过于智能
* 不要将`equals`方法参数声明中的`Object`对象替换为其他类型.

### equals示例
```java
public final class PhoneNumber {
    boolean booleanF;
    byte byteF;
    char charF;
    short shortF;
    int intf;
    float floatf;
    double doublef;
    Object objectF;
    Object[] arrayF;

    /*由IDEA自动生成*/
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof MyObject)) return false;

        MyObject myObject = (MyObject) o;

        if (booleanF != myObject.booleanF) return false;
        if (byteF != myObject.byteF) return false;
        if (charF != myObject.charF) return false;
        if (shortF != myObject.shortF) return false;
        if (intf != myObject.intf) return false;
        if (Float.compare(myObject.floatf, floatf) != 0) return false;
        if (Double.compare(myObject.doublef, doublef) != 0) return false;
        if (!objectF.equals(myObject.objectF)) return false;
        // Probably incorrect - comparing Object[] arrays with Arrays.equals
        return Arrays.equals(arrayF, myObject.arrayF);
    }
}
```

***
## 第9条 覆盖equals时总要覆盖hashCode

### Object规范[JavaSE6]
* 在应用程序的执行期间,只要对象的`equals`方法的比较操作所用到的信息没有被修改,
那么对这同一个对象调用多次,`hashCode`方法必须始终如一地返回同一个整数.
在同一个应用程序的多次执行过程中,每次执行返回的整数可以不一致.
* 如果两个对象根据`equals`方法比较结果是相等的,那么调用者两个对象中任意对象的
`hashCode`方法都必须产生同样的整数结果.
* 如果两个对象根据`equals`方法比较结果是不相等的,那么调用这两个对象任意对象的
`hashCode`方法,则不一定要产生不同的整数结果.
**总结**
```java
equals等 => hashcode等; equals不等 ≠> hashcode不等
```

### 覆写hashCode方法的简单办法
```java
public class MyObject {
    boolean booleanF;
    byte byteF;
    char charF;
    short shortF;
    int intf;
    float floatf;
    double doublef;
    Object objectF;
    Object[] arrayF;

    /*由IDEA自动生成,使用Equals中涉及到的每个域*/
    @Override
    public int hashCode() {
        int result;
        long temp;
        result = (booleanF ? 1 : 0);
        result = 31 * result + (int) byteF;
        result = 31 * result + (int) charF;
        result = 31 * result + (int) shortF;
        result = 31 * result + intf;
        result = 31 * result + (floatf != +0.0f ? Float.floatToIntBits(floatf) : 0);
        temp = Double.doubleToLongBits(doublef);
        result = 31 * result + (int) (temp ^ (temp >>> 32));
        result = 31 * result + objectF.hashCode();
        result = 31 * result + Arrays.hashCode(arrayF);
        return result;
    }
}
```
***
## 第11条 谨慎使用clone
### 深拷贝简单例子
```java
ublic class CloneObject implements Cloneable{
    Object[] elememts = new Integer[]{1,23,4};
    Integer anInt = 1;
    List<String> list = new LinkedList<>(Arrays.asList("1","2"));

    @Override
    public CloneObject clone() {
        try {
            CloneObject clone = (CloneObject) super.clone();
            clone.anInt = new Integer(this.anInt); //深拷贝
            clone.elememts = this.elememts.clone();
            clone.list =(List<String>) ((LinkedList<String>)list).clone(); //浅拷贝
            for (int i=0; list != null && i<list.size() ; ++i){
                clone.list.set(i, new String(list.get(i))); //new String对String对象深拷贝.
            }
            return clone;
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return null;
    }

    public static void main(String[] args)
    {
        CloneObject a = new CloneObject();
        CloneObject b = a.clone();

        System.out.println(a==b);                           //false
        System.out.println(a.anInt == b.anInt);             //false
        System.out.println(a.elememts == b.elememts);       //false
        System.out.println(a.list == b.list);               //false
        System.out.println(a.list.get(0) == b.list.get(0)); //false
    }
}
```


# chapter04 类和接口
***

## 第23条 请不要在新代码中使用原生态类型
```java
//参数化类型,表示可以包含人和对象类型的集合
//有受检异常
 List<Object> list = new ArrayList<>();
list.add(new String("a"));
Integer b = list.get(0);//受检异常

//通配符类型,表示只能包含某种位置对象类型的集合
//有受检异常
 List<Object> list = new ArrayList<>();
list.add(new String("a"));//受检异常

//原生态类型,脱离了泛型系统
//运行时异常
Set s = new HashSet();
s.add(new String("str"));
Integer a = (Integer)list.get(0); //java.lang.ClassCastException
```


# chapter09 异常
***

## 第60条 优先使用标准的异常
![常用异常](/img/effective-java-read-note/common-exception.png)