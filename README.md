# 前言

最近系统复习设计模式，边学习边实现，学习过程中我主要参考了程杰的《大话设计模式》和[这个网站](http://c.biancheng.net/view/1317.html)。大话中的示例代码是C++，后者网站虽然用的是Java，但每一个示例都用窗体展示，夹杂了大量无关模式的awt和swing代码，不利于集中理解模式运作过程。网上大量关于设计模式的文章总是充斥着很多抽象说明和到处复制粘贴的UML图，对初学者理解模式本身反而造成干扰。例如下面这种抽象描述，实际上是对模式完全理解之后的高度概括，很多文章上来就写这种抽象定义，完全是浪费读者时间。这种描述对模式识别的理解是不是必须的，甚至对初学者是有害的。
> 状态（State）模式的定义：对有状态的对象，把复杂的“判断逻辑”提取到不同的状态对象中，允许状态对象在其内部状态发生改变时改变其行为。

在本仓库中，我会侧重每一个模式的代码实现，去芜存菁地讲解每一个模式，每一个代码示例我都已确保在本地成功运行，且代码尽可能遵守Java编程规范(主要是阿里的规范)。每一个模式讲解顺序如下：
- **模式说明**：简要说明模式使用场景和该模式相比简单粗暴的方法(通常是if-esle或者单个类封装)的好处，另外像组合模式里有透明方式和安全方式，单例模式里有饿汉方式和懒汉方式，模板模式里有钩子方法也会简要说明。
- **结构**：列明该模式用到的抽象和具体类。
- **代码演示**：可执行的符合Java编程规范的模式示例代码，并对关键语句都加上了注释。

以“单例模式为例”，讲解文档呈现如下样貌。

********

## 模式说明

某些场景要求一个对象只能有一个实例，可以通过单例模式实现。由于只能有一个实例，因此该类的构造器必须用`private`修饰来禁止通过`new`操作符来新建实例。类内持有一个`private`修饰的本类类型的实例，又因为只能有一个实例，所以使其为类成员，如该类类名`Singleton`，可声明为：
`private static Singleton instance;`
通过类内的一个`public`修饰的`getIntance`方法来获取实例，该方法也是`static`的。根据实例产生的时机分为饿汉模式和懒汉模式，懒汉模式中有线程安全和不安全的写法。

**饿汉模式**
类属性Singleton instance在声明时就实例化。
`private static Singleton instance = new Singleton();`


**懒汉模式**
类属性`Singleton`只声明但不实例化，将实例化动作放到`getInstance`方法中，则只有调用`getInstance`时才能获取`Singleton`的实例。懒汉模式因为在调用`getInstance`时才获得实例，存在多线程竞争的问题，因此可以结合`volatile`(避免指令重排)和`synchronized`(保证原子性)以双锁检测方式将`Singleton`写成线程安全的类。
​

​本示例演示饿汉写法，非线程安全懒汉写法和双锁检测线程安全写法。
​

## 结构
​单例类
  持有一个本类的类属性，维护一个`getInstance`方法。
​

## 代码演示
```java
package com.yukiyama.pattern.creation;

/**
 * 单例模式
 */
public class SingletonDemo {

    public static void main(String[] args) {
        SingletonSimple s1 = SingletonSimple.getInstance();
        SingletonSimple s2 = SingletonSimple.getInstance();
        // 输出“true”，说明s1与s2是同一个实例
        System.out.println(s1 == s2);
        SingletonHungry s3 = SingletonHungry.getInstance();
        SingletonHungry s4 = SingletonHungry.getInstance();
        // 输出“true”，说明s3与s4是同一个实例
        System.out.println(s3 == s4);
        Singleton s5 = Singleton.getInstance();
        Singleton s6 = Singleton.getInstance();
        // 输出“true”，说明s3与s4是同一个实例
        System.out.println(s5 == s6);
    }

}

/**
 * 饿汉模式
 * 在类加载时创建常量化实例，不存在多线程导致可能出现多个实例的问题
 */
class SingletonHungry{
    // 在定义SingletonHungry类型的属性时直接实例化，类内可以访问private构造器
    private static final SingletonHungry instance = new SingletonHungry();
    
    // 将构造器声明为private，外部无法用new获取
    private SingletonHungry() {}
    // 外部通过一个public的getInstance()方法获取该类实例
    public static SingletonHungry getInstance() {
        return instance;
    }
}

/**
 * 多线程下的双锁检测(Double-Check Locking)单例
 * 懒汉模式
 */
class Singleton{
    // 以volatile修饰
    private static volatile Singleton instance;
    
    private Singleton() {}
    public static Singleton getInstance() {
        // 第一次判断的目的是避免每次getInstance()都加锁
        // 若已经存在实例，直接返回
        if(instance == null) {
            synchronized (Singleton.class) {
                // 再次判断是防止两个线程在instance==null时
                // 同时进入第一个if内，由于加锁，其中一个先new了
                // 实例，此时必须再判断一次防止第二个也new一个实例
                if(instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}

/**
 * 懒汉模式
 * 非线程安全版
 */
class SingletonSimple {
    private static SingletonSimple instance;
    
    // 将构造器声明为private，外部无法用new获取
    private SingletonSimple() {}
    // 外部通过一个public的getInstance()方法获取该类实例
    public static SingletonSimple getInstance() {
        // 每次获取前判断该类实例是否已存在，若无则new一个
        if (instance != null) {
            instance = new SingletonSimple();
        }
        return instance;
    }

}
```
