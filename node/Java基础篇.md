# 第1章 什么是面向对象

## 1.1 面向过程和面向对象

- 面向过程：是一种以过程为中心的编程思想，是一种自顶向下的编程模式；简单来说程序员把问题分解成一个个步骤，每个步骤依次调用。
- 面向对象：程序员将问题分解成一个个步骤，对每个步骤进行相应的抽象，形成对象，通过不同对象之间的调用，综合解决问题。

> 举个栗子：
>
> 比如向造一辆车，首先要定义车的各种属性，然后将各种属性封装在一起，抽象成一个Car类。

## 1.2 面向对象的三大基本特征

1. 封装（Encapsulation）

​       隐藏对象内部的复杂性，只对外公开简单的接口。便于外界调用，从而提高系统的可扩展性、可维护性。通俗的说，把该隐藏的隐藏起来，该暴露的暴露出来。这就是封装性的设计思想。

​	体现1：属性私有化

```java
public class User {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

​	体现2：私有化方法

```java
public class Circular {

    public Integer r;

    public Double s;

    private Double countS(){
        return (1.0/2) * Math.PI * Math.pow(this.r,2);
    }

    public Integer getR() {
        return r;
    }

    public void setR(Integer r) {
        this.r = r;
    }

    public Double getS() {
        this.s = countS();
        return this.s;
    }

}
```

​	体现3：私有化构造器

```java
public class SingleDemo {
    private static volatile SingleDemo INSTANCE;
    
    private SingleDemo(){
        
    }
    
    public static SingleDemo getInstance(){
        if (INSTANCE == null) {
            synchronized (SingleDemo.class) {
                if (INSTANCE == null){
                    INSTANCE = new SingleDemo();
                }
            }
        }
        return INSTANCE;
    }
}
```

2. 继承（Inheritance）

​       子类继承父类的特征和行为，使得子类对象（实例）具有父类的实例域和方法，或子类从父类继承方法，使得子类具有父类相同的行为。

> 通过继承创建的新类称为”子类“或”派生类“
>
> 被继承的类，称为“基类”、“父类”、“超类”

举个栗子：

```java
public class MyClassLoader extends ClassLoader{
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        return super.findClass(name);
    }
}
```

3. 多态（Polymorphism）

​	一个类实例的相同方法在不同情形下有不同的表现形式。

​	关于栗子在第2章介绍。

## 1.3 面向对象的五大基本原则

1. 单一职责原则：一个类最好只做一件事，只有一个引起它的变化的原因。
2. 开放封闭原则：软件实体应该是可扩展且不可修改的。
3. - 对扩展开放，意味着当前有新的需求时，可以对现有代码进行扩展，以适应新的情况。
   - 对修改封闭，意味着类一旦设计完成，就可以独立完成其工作，而不要对其进行任何尝试的修改。


3. 里氏替换原则：子类必须能够替换其基类。
4. 接口隔离原则：使用多个小的专门接口，而不要使用一个大的总接口。
5. 依赖倒置原则：程序要依赖于抽象接口，而不是具体的实现。

# 第2章 面向对象的核心概念