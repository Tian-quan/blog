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
    * 第13条 使类和成员的可访问性最小化
    * 第14条 在公有类中使用访问方法而非公有域
    * 第15条 使可变性最小化
    * 第16条 复合优先于继承
***
***
# chapter03
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