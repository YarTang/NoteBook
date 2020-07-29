# 一、创建型模式 (Creational)

## 概述

创建型设计模式抽象了对象实例化过程，帮助一个系统独立于如何创建、组合和表示其对象。类创建模式使用继承改变被实例化的类，对象创建模式将实例化委托给另一个对象，或者推迟到子类中。

## 1. 单例 (Singleton)

### 概述

在Java应用中，单例对象能保证在一个JVM中，该类只有一个实例对象存在， 并提供一个全局的访问点。这样的模式有几个好处：

- 某些类创建比较频繁，对于一些大型的对象，这是一笔很大的系统开销。
- 省去了 new 操作符，降低了系统内存的使用频率，减轻 GC 压力。

主要思想：类自身保存它的唯一实例，保证没有其他实例可以被创建，并且它提供一个访问该实例的方法。

JDK 中的单例有 `Runtime.getRuntime()`, `NumberFormat.getInstance()` 等

### 结构

![image-20200726170716588](C:\Users\YarTang\AppData\Roaming\Typora\typora-user-images\image-20200726170716588.png)





### 1.1 饿汉式

思路：作为类的静态全局变量，加载该类时实例化。
缺点：真正使用该实例之前（也有可能一直没用到），就已经实例化，浪费资源。
对于 Hotspot VM，如果没涉及到该类，实际上是首次调用 getInstance() 时才实例化。

```java
public class HungrySingleton {
    private static HungrySingleton instance = new HungrySingleton();

    private HungrySingleton() {

    }

    /**
     * 基于类加载机制，达到线程安全的效果
     * @return 返回单例模式创建的实例化对象
     */
    public static HungrySingleton getInstance() {
        return instance;
    }

    public void method() {
        System.out.println(instance + " method OK");
    }

    public static void main(String[] args) {
        HungrySingleton hungrySingleton = HungrySingleton.getInstance();
        hungrySingleton.method();
    }

}
```

### 1.2 懒汉式

思路：在 getInstance() 上实现同步
缺点：每次调用 getInstance() 都会加锁，但实际上只有首次实例化时才需要，后续的加锁都是浪费，导致性能大降

```java
public class LazySingleton {
    private static LazySingleton instance = null;

    private LazySingleton() {

    }

    private static synchronized LazySingleton getInstance() {
        if (instance == null) {
            instance = new LazySingleton();
        }
        return instance;
    }

    private void method() {
        System.out.println(instance + " method OK");
    }

    public static void main(String[] args) {
        LazySingleton instance = LazySingleton.getInstance();
        instance.method();
    }

}
```

### 1.3 双重检查的懒汉式

不同步的情况下检查到尚未创建，再同步检查到尚未实例化时，才实例化。以便大大减少同步的情况。

```java
public class LazySingletonWithDoubleCheck {
    private volatile static LazySingletonWithDoubleCheck instance = null;

    private LazySingletonWithDoubleCheck() {

    }

    public static LazySingletonWithDoubleCheck getInstance() {
        if (instance == null) {
            synchronized (LazySingletonWithDoubleCheck.class) {
                if (instance == null) {
                    instance = new LazySingletonWithDoubleCheck();
                }
            }
        }
        return instance;
    }

    public void method() {
        System.out.println(instance + " method OK");
    }

    public static void main(String[] args) {
        LazySingletonWithDoubleCheck instance = LazySingletonWithDoubleCheck.getInstance();
        instance.method();
    }
}
```

### 1.4 静态内部类方式

思路：全局静态成员放在内部类中，只有内部类被引用时才实例化，达到延迟实例化的目的。

- 确保实例化至 getInstance() 的调用
- 无需加锁，并发性能较好

```java
public StaticHolderSingleton {
    /**
     * 延迟加载实例
     */
    private static class InstanceHolder {
        private static StaticHolderSingleton instance = new StaticHolderSingleton();
    }

    private StaticHolderSingleton() {

    }

    public static StaticHolderSingleton getInstance() {
        return InstanceHolder.instance;
    }

    public void method() {
        System.out.println(this + " method OK");
    }

    public static void main(String[] args) {
        StaticHolderSingleton instance = StaticHolderSingleton.getInstance();
        instance.method();
    }
}
```

### 1.5  使用枚举

解决线程同步，还可以防止反序列化

```java
public enum EnumSingleton {
    INSTANCE;
    public void m() {
        
    }
    public static void main(String[] args) {
        System.out.println(EnumSingleton.INSTANCE);
    }
}
```



## 2. 生成器 (Builder)

将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。

![image-20200726170631938](C:\Users\YarTang\AppData\Roaming\Typora\typora-user-images\image-20200726170631938.png)

JDK 中的实现主要是 `StringBuilder`, `StringBuffer` 等

### 2.1 StringBuilder 的简单实现

```java
public abstract class AbstractMyStringBuilder {
    /**
     * 底层数组
     */
    protected char[] value;

    /**
     * 记录字符串长度
     */
    protected int count;

    /**
     * 构造器
     * @param capacity 容量
     */
    public AbstractMyStringBuilder(int capacity) {
        count = 0;
        value = new char[capacity];
    }

    /**
     * buildPart() - 各部件的实现
     * @param c 构造配件
     * @return Product - 被构造的对象
     */
    public AbstractMyStringBuilder append(char c) {
        ensureCapacity(count + 1);
        value[count++] = c;
        return this;
    }

    public AbstractMyStringBuilder append(String str) {
        if (str == null) {
            return append("null");
        }
        int len = str.length();
        ensureCapacity(count + len);
        for (int i = 0; i < len; i++) {
            append(str.charAt(i));
        }
        count += len;
        return this;
    }

    /**
     * 保证容量足够
     * @param minimumCapacity 生成器的最小长度
     */
    private void ensureCapacity(int minimumCapacity) {
        if (minimumCapacity - value.length > 0) {
            value = Arrays.copyOf(value, newCapacity(minimumCapacity));
        }
    }

    /**
     * 返回一个容量值，至少和给定的最小容量值相等
     * @param minCapacity 给定的最小容量值
     * @throws OutOfMemoryError 如果给定的最小容量值为负或者大于 Integer.MAX_VALUE
     */
    private int newCapacity(int minCapacity) {
        int newCapacity = (value.length << 1) + 2;
        if (newCapacity - minCapacity < 0) {
            newCapacity = minCapacity;
        }
        if (newCapacity < 0) {
            if (minCapacity < 0) {
                throw new OutOfMemoryError();
            }
            newCapacity = Integer.MAX_VALUE;
        }
        return newCapacity;
    }

    /**
     * 抽象方法 toString
     * @return a string representation of this sequence of characters
     */
    @Override
    public abstract String toString();
}
```

```java
public class MyStringBuilder extends AbstractMyStringBuilder {
    public MyStringBuilder() {
        super(16);
    }

    @Override
    public String toString() {
        return new String(value, 0, count);
    }

    public static void main(String[] args) {
        MyStringBuilder msb = new MyStringBuilder();
        msb.append("Hello World");
        msb.append('!');
        System.out.println(msb);
    }
}
```

### 2.2 一个使用生成器模式的简单实例

```java
public class User {
    /**
     * User 被构建的复杂对象 - Product
     */
    private String name;
    private String password;
    private Integer id;

    public User(String name, String password, Integer id) {
        this.name = name;
        this.password = password;
        this.id = id;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", password='" + password + '\'' +
                ", id=" + id +
                '}';
    }

    /**
     * 生成器 - ConcreteBuilder
     */
    public static class UserBuilder {
        private String name;
        private String password;
        private Integer id;

        // 各部件的实现 - buildPart()
        public UserBuilder name(String name) {
            this.name = name;
            return this;
        }

        public UserBuilder password(String password) {
            this.password = password;
            return this;
        }

        public UserBuilder id(Integer id) {
            this.id = id;
            return this;
        }

        public User build() {
            return new User(this.name, this.password, this.id);
        }
    }

    public static void main(String[] args) {
        System.out.println(new UserBuilder().name("aaa").password("123").id(1).build());
    }
}
```

## 3. 抽象工厂 (Abstract Factory - Kit)

提供一个接口，创建一系列相关或者相互依赖的对象，无需指定他们具体的类。客户端可以灵活选择实现类，完成对对象的创建。

从高层次来看，抽象工厂使用了组合，即 Cilent 组合了 AbstractFactory，而工厂方法模式使用了继承。

AbstractFactory 与 Builder 类似，它们都用来创建复杂对象，但是 Builder 侧重于一步步构造一个复杂对象， AbstractFactoy 侧重多个系列的产品的对象。

Builder 最后构造完成后返回对象，AbstractFactory 产品立即返回。

![image-20200726220840135](C:\Users\YarTang\AppData\Roaming\Typora\typora-user-images\image-20200726220840135.png)

JDK 中的实例 `NumberFormat.getInstance()`

***一般来说，AbstractFactory 使用 工厂方法实现，也可以用 原型方式实现；具体的工厂 ConcreteFactory 使用单例实现。***

### 特点

- 分离了具体的类
- 易于交换产品系列
- 有利于产品的一致性
- 难以支持新种类的产品 

### 简单实现

```java
public class AbstractProductA {
}
```

```java
public class AbstractProductB {
}
```

```java
public class ProductA1 extends AbstractProductA {
}
```

```java
public class ProductA2 extends AbstractProductA {
}
```

```java
public class ProductB1 extends AbstractProductB {
}
```

```java
public class ProductB2 extends AbstractProductB {
}
```

```java
public abstract class AbstractFactory {
    abstract AbstractProductA createProductA();
    abstract AbstractProductB createProductB();
}
```

```java
public class ConcreteFactory1 extends AbstractFactory {
    AbstractProductA createProductA() {
        return new ProductA1();
    }

    AbstractProductB createProductB() {
        return new ProductB1();
    }
}
```

```java
public class ConcreteFactory2 extends AbstractFactory {
    AbstractProductA createProductA() {
        return new ProductA2();
    }

    AbstractProductB createProductB() {
        return new ProductB2();
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        AbstractFactory abstractFactory = new ConcreteFactory1();
        AbstractProductA productA = abstractFactory.createProductA();
        AbstractProductB productB = abstractFactory.createProductB();
        // do something with productA and productB
    }
}
```

## 4. 工厂方法 (Factory Method - Virtual Constructor)

定义一个用于创建对象的接口，让子类决定实例化哪一个类，FactoryMethod 是一个类的实例化延迟到子类。

![image-20200726220906490](C:\Users\YarTang\AppData\Roaming\Typora\typora-user-images\image-20200726220906490.png)

工厂模式的一个工厂接口的子类只能实例化一个产品；抽象工厂能实例多个产品

 ### 简单实现

```java
public abstract class AbstractCreator {
    abstract public Product factoryMethod();
    public void anOperation() {
        Product product = factoryMethod();
    }
}
```

```java
public interface Product {

}
```

```java
public class ConcreteProduct implements Product {

    public ConcreteProduct() {
        System.out.println(this);
    }
}
```

```java
public class ConcreteCreator extends AbstractCreator {
    @Override
    public Product factoryMethod() {
        return new ConcreteProduct();
    }

    public static void main(String[] args) {
        Product product = new ConcreteCreator().factoryMethod();
    }
}
```

## 5. 原型 (Prototype)

### 概述

用原型实例指定创建对象的种类，通过拷贝这些原型实例创建新的对象。

JDK 中的实例：java.lang.Object.clone()

### 结构

![image-20200726225534186](C:\Users\YarTang\AppData\Roaming\Typora\typora-user-images\image-20200726225534186.png)



### 简单实现

```java
public abstract class AbstractPrototype {
    abstract AbstractPrototype myClone();
}
```

```java
public class ConcretePrototype extends AbstractPrototype {

    private String filed;

    public ConcretePrototype(String filed) {
        this.filed = filed;
    }

    @Override
    AbstractPrototype myClone() {
        return new ConcretePrototype(filed);
    }


    public static void main(String[] args) {
        AbstractPrototype prototype = new ConcretePrototype("123");
        AbstractPrototype prototype1 = prototype.myClone();
        System.out.println(prototype + "\n" + prototype1);
    }
}
```

# 二、结构型模式 (Structural)

结构型模式涉及如何组合类和对象获得更大的结构，类结构型模式主要采用继承机制组合接口和类，对象结构型模式采用组合或聚合来组合对象。

## 1. 适配器 (Adapter - Wrapper)

### 概述

将一个类的接口转换成客户希望的另外一个接口，使原本由于接口不兼容不能在一起工作的类能一起工作。

JDK 中的使用：`Arrays.asList()`, `Collections.list()` 等

### 结构

![image-20200728114758965](C:\Users\YarTang\AppData\Roaming\Typora\typora-user-images\image-20200728114758965.png)

- Target - 用户使用的接口
- Adaptee - 一个已经存在的接口，需要适配 
- Adapter - 将 Adaptee 接口与 Target 接口适配

### 简单实现

```java
public interface Duck {
    void quack();
}
```

```java
public interface Turkey {
    void gobble();
}
```

```java
public class WildTurkey implements Turkey{
    @Override
    public void gobble() {
        System.out.println("gobble");
    }
}
```

```java
public class TurkeyAdapter implements Duck{
    Turkey turkey;

    public TurkeyAdapter(Turkey turkey) {
        this.turkey = turkey;
    }

    @Override
    public void quack() {
        turkey.gobble();
    }

    public static void main(String[] args) {
        Turkey turkey = new WildTurkey();
        Duck duck = new TurkeyAdapter(turkey);
        duck.quack();
    }
}
```

## 2. 桥接 (Bridge - Handle / Body)

将抽象部分与其实现方式分离，使它们独立变化。

## 3. 组合 (Composite)

## 4. 装饰

## 5. 外观

## 6. 享元

## 7.代理 

# 三、行为型模式 (Behavioral)

## 1. 策略模式

### 概述

定义一系列算法，将这些算法封装起来，并使它们可以互相替换。使算法可以独立于使用它的客户变化。

JDK 中的使用：`java.util.Comparator`

### 结构

![image-20200729115149736](C:\Users\YarTang\AppData\Roaming\Typora\typora-user-images\image-20200729115149736.png)

- Context 是使用该算法族的类，动态的调用 Strategy 中的具体算法。
- Strategy 定义了一个算法族，ConcreteStrategy 实现了具体的算法。

### 简单实现

```java
public interface QuackBehavior {
    void quack();
}
```

```java
public class Quack implements QuackBehavior {
    @Override
    public void quack() {
        System.out.println("quack");
    }
}
```

```java
public class Squeak implements QuackBehavior {
    @Override
    public void quack() {
        System.out.println("squeak");
    }
}
```

```java
public class Duck {

    public void quack(QuackBehavior quackBehavior) {
        quackBehavior.quack();
    }

    public static void main(String[] args) {
        Duck duck = new Duck();
        duck.quack(() -> System.out.println("hah"));
        duck.quack(new Quack());
    }
}
```