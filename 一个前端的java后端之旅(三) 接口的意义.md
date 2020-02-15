# 一个前端的java后端之旅(三) 接口的意义

![enter description here](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1581729607349.png)

距离写完上一篇文章已经有很长一段时间了，这段时间刚开始比较忙，后来就过年，出现疫情，在家懒了几天，也看了一些java书籍或网络资料中关于interface的介绍与讲解，但是仍然没有get到自己想要点。

我不想满足于知道接口是如何定义的，是用来被类实现的，我想知道为什么要用接口，一段代码我认为很稳定了我为什么还要违心的再去写一个接口，我不想以后假如我从事后端java，就像一个机器一样，每次一个需求来临，我就机械化的定义接口，定义类，why？为什么要这样？这篇文章便来探讨一下（也是看了三遍慕课网七月老师的java全栈课才解决了自己的困惑，当然也是基于学习这门课自己没基础才来学习分享这些东西的）。

## 1. 类不够用吗 为什么要加入接口？
上篇文章中简单说了面向对象的东西，其中有个核心便是类（`class`），类是对于对象的抽象，我们可以将某一类的事物用类来描述，然后通过实例化的方式生产对象，这已经是一层抽象了，这还不足够用来表达我们的程序逻辑嘛？

当然不够，如果够了怎么会有接口(`interface`)的出现。java语言之所以庞大，写起来也不够动态语言简洁，它能实现的东西动态语言也能实现，但它为什么能经久不衰，在企业实现大型项目时也是首选，这就要说到它的稳定性，它不是为了追求时尚而生的，而是为了追求稳定，为此新java版本也放弃了很多新的特性，大型项目的稳定性也是最重要的，因此java的追求也正是大型项目的追求。

但java如何实现稳定性，首先它的语言每次版本变化后语法的变更并不会很大，其次java作为一种静态语言，在编码过程中通过静态检查也可以直接避免语法层面的错误，由于其反射机制，idea，eclipse等编辑器也做的非常智能，各种错误提示，自动导入类，同步修改类名等都非常精确。

而且对于java的学习并不是简单的语法学习，因为java也并不是简单的语法堆积，其中必要的设计模式和软件工程相关知识也是我们必须学习的。在java世界里，OCP(开闭原则)是一个最基础的原则，它能够引导我们创建一个稳定、灵活的系统。

开闭原则的定义为：Software entities like classes,modules and functions should be open for extension but closed for modifications.（一个软件实体如类，模块和函数应该对扩展开放，对修改封闭）

其最重要的内容便是`对扩展开放，对修改封闭`。在java代码的书写过程中，我们便要严格遵循这个规则，为了更好地实现OCP，引入`interface`的概念是必要的，在当前java中最为火热的概念中ICO，DI等（IOC实现了DI）一大部分目的也是为了更好地实现`OCP`，其中`interface`便扮演着及其重要的角色。

因此对于本小节题目中的问题而言，为什么要引入接口，因为它可以在一定程度上帮我们提高java代码的稳定性，在java追求OCP的过程中扮演了很重要的角色。

## 2. 一个例子来看接口如何帮我们提升代码稳定性

如果说我定义了一个接口`A`

``` java
public interface A {
    xxxxx
}
```

在实现自己业务逻辑的时候我有两个类需要传入A类型的对象

``` java
public class MyFunction1 {
    A a;
    
    public setA(A a) {
        this.a = a;
    }
}

public class MyFunction2 {
    A a;
    
    public setA(A a) {
        this.a = a;
    }
}
```
假设`MyFunction1`和`MyFunction2`是我实现具体业务逻辑的代码，它们都依赖了`A`类型的数据，如果`A`是个类，在`MyFunction1`中A有部分`方法`的逻辑需要重新修改，而`MyFunction2`中需要保持不变，这改如何做，修改类`A`中的代码势必导致`MyFunction1`或者`MyFunction2`至少有一个代码也需要改变，一处改变会导致另外一处代码也跟着改变，这些修改有写不符合OCP原则了，使得多处代码都变得不够稳定。

如果此时就像我们上面定义的A是个接口`interface`，那我可以对应实现一个`impA`,在后面升级时`MyFunction1`中有部分业务逻辑需要修改，我可以直接在写一个`impA1`,在`setA`方法中传入`impA1`的实例就可以了，`MyFunction1`中的代码的方法调用也不用更改，此时也正满足了`OCP`原则`扩展开放，修改封闭`，对`impA`类也没有修改。此时代码由于修改而导致新bug的可能性也降低很多，便保证了代码的稳定性，在未来进行代码的维护上也很方便。

接口可以看作是类的进一步抽象，java编程最吸引人的其中一点便是面向抽象编程，其中接口大部分情况就就是这里的抽象。

## 3. 接口的使用场景
尽管我自己是个没学过java的人，但想想去年毕设时候我的项目后端也是java写的，写的一脸懵逼，不过大致知道了项目中写java类之前需要定义接口，然后再写实现接口的类，为什么要这样写，如果直接写类可以吗？当然可以，但是此时我便可以清楚地知道为什么要有这么一层接口。

为了之后项目升级维护的方便性，我们需要使用接口，然后写类，尽管可能这个接口只会在一处被用到，如果写代码时候我们确定某些代码一定不会有更改，还是建议使用接口，编程这种事，谁能确定些什么啊，先写接口再写实现类也是一种防御性编程，是一种优秀的习惯，不是你觉得不会变就要按照类的方式写。

## 4. 接口语法层面的细节
前面啰啰嗦嗦地说了很多，但其实并没有什么关于`interface`语法层面的东西，但是有了这些啰里啰嗦的铺垫，相信基于对`interface`的这个认识现在来学习它应该会有不同的感觉（懂了原理，我们就缺语法层面的用法了）。

### 4.1 接口的定义、使用、实现与继承

上面举例子的时候其实我们已经看到过一次接口的定义，在接口定义中，我们需要提供方法名与方法类型，但是不需要去实现它。

**接口的定义**
例如定义一个`IntSequence`接口(参考《写给大忙人的JavaSE9核心技术》示例)
``` java
public interface IntSequence {
    boolean hasNext();
    int next();
}
```
可以看出来这个接口定义了一个判断是否拥有下一个元素的方法`boolean hasNext();`和获取下一个元素的方法`int next();`,并且这里仅仅是定义了方法名和类型，并没有进行具体的实现。这就是通常情况下定义接口的方式。

**接口的使用**
上面在标题2的内容中提到过面向抽象编程，这里的抽象大多数情况下就是接口。下面看看接口的使用来理解一下这句话的意义。

借助上面定义的接口来实现一个算平均数的方法。(参考《写给大忙人的JavaSE9核心技术》示例)

``` java
public static double average(IntSequence seq, int n) {
    int count = 0;
    double sum = 0;
    while(seq.hasNext() && count < n) {
        count++;
        sum += seq.next();
    }
    return count == 0 ? 0 : sum / count;
}
```
代码很简单，此时我们并没有定义具体的实现类，仅仅靠接口的定义我们就可以依靠它来进行业务逻辑的编写，通常这一部分需要是比较稳定的，而具体的类我们可以基于接口实现多个类，hasNext()和next()里的业务逻辑每个实现上面接口的类中可以是不相同的。我们在这里也仅仅是依赖了比类更抽象的接口的，而不是具体的类，这就是面向抽象来进行编程。


**接口的实现**
实现接口要使用`implements`关键字。如(参考《写给大忙人的JavaSE9核心技术》示例)

``` java
public class SqureSequence implements IntSequence {
    private int i;
    
    public boolean hasNext() {
        return true
    }
    
    public int next() {
        i++;
        return i * i;
    }
}
```
从例子中可以看出`SqureSequence`中`hasNext()`结果总为`true`，`next()`获取到的结果为每个数的平方。

我们也可以基于上面的接口实现其他不同的类，满足不同的需求，在调用`average`的时候，传入对接口`IntSequence`不同的实现类的实例结果也会不同。我们可以通过具体的实现类，来确定`average`最后不同的表现，但是`average`我们并不用每次都修改，这就保证了主要业务逻辑部分的代码的稳定性。

当然接口还可以当做数据的类型来用：

``` java
IntSequence seq = new SqureSequence();
```

**接口的继承**

接口可以通过`extends`关键字来继承

例：

``` java
public interface Closeable {
    void close();
}

public interface Channel extends Closeable {
    boolean isOpen();
}
```
如果此时要实现`Channel`接口，那就必须提供两个方法，一个是`Closeable`中定义的`void close();`一个为`Channel`中定义的`boolean isOpen();`

并且此时这个实现类新建的实例对象可以转换为两个接口的类型。

例如：

```java
public class MyClass implements Channel {
    ......
}
MyClass m = new MyClass();
Channel c = m;
Closeable c1 = m;

```

**实现多个接口**
一个类也可以同时实现多个接口

```java
public class FileSequence implements IntSequence, Closeable {
    ......
}
```
此时`IntSequence`和`Closeable`都是`FileSequence`类的父类型。

**关于父类型的补充**
对于一个类来说，如果实现了一个接口，这个接口就是这个类的父类型，如果实现了多个接口，那每个接口都是这个类的父类型。接口的父类型也算这个父类型。

对于这个类产生的实例对象，在强制类型转换时，只能被强制转换为它的实际类型或者它的父类型之一，否则就会报错或者类型转换异常。

通过`instanceof`关键字可以预先判断某个类型是否是一个对象所属类的父类型。用法为`obj instanceof Type`，为`true`则说明是的。

这里简单测试了一下，附上代码：

``` java
public static void main(String[] args) {
    Fimpl f = new Fimpl();
    FF f1 = f;
    F f2 = f;
    
    System.out.println(f instanceof Fimpl);
    System.out.println(f instanceof FF);
    System.out.println(f instanceof F);
}

public interface F extends FF{
    void sayHello();
}

public interface FF {
    void sayHi();
}

public class Fimpl implements F{
    public void sayHi() {
        System.out.println("Hi");
    }

    public void sayHello() {
        System.out.println("Hello");
    }
}

```
结果为三个`true`.

**接口中的常量**

在接口中除了定义函数，还可以定义常量，在接口中定义的常量会被自动转为`public static final`

``` java
public interface SwingConstants {
    int NORTH = 1;
    int NORTH_FAST = 2;
    int EAST = 3;
    ...
}
```
在接口中定义的常量可以通过`接口名.常量`的方式引用，例如`SwingConstants.NORTH`。在定义时仅需要写`类型 名称 = 值;`即可。

### 4.2 接口的静态方法、默认方法与私有方法

按照正常思维，接口中的定义就是方法名，在早期版本java中接口里也确实不能定义完整的方法，但是现在（这里我学习的是java 9，就只保证9+里面肯定有），有3中方式可以在接口中实现具体的方法。

**静态方法**
在接口中定义的静态方法可以通过`接口名.方法名`的方式调用。
定义方法为：

``` java
public interface IntSequence {
    public static boolean hasNext() {
        return true;
    }
}
```
接口的静态方法在工厂方法的应用中比较有意义。

**默认方法**
``` java
public interface IntSequence {
    default boolean hasNext() {
        return true;
    }
}
```
接口中的方法加上方法体与default修饰符就可以改造成默认方法，实现该接口的类可以选择覆盖或使用默认方法。

如果一个类继承多个接口，而多个接口中有默认方法名称和参数类型相同的默认方法则会产生冲突。(参考《写给大忙人的JavaSE9核心技术》示例)

``` java
public interface Person {
    String getName();
    default int getId(){
        return 0;
    }
}

public interface Identified {
    default int getId() {
        return Math.abs(hashCode());
    }
}

public class Employee implements Person, Identified {
    ...
}
```

解决方法,通过确定父类型调用其方法（super可以调用父类型的方法）。

``` java
public class Employee implements Person, Identified {
    public int getId() {
        return Identified.super.getId();
    }
    ...
}
```

**私有方法**

从java9开始接口中可以拥有私有方法，可以是static方法也可以是实例方法，但只能用于接口自身的方法中。

## 小结

接口相关的语法并没有难度，但是其思想确是值得探究的。我一直在想自己可以写一些java程序，但是还是觉得它比较难，可能也正是因为java并不是简单的语法堆积，它内部有很多值得探究的地方，我却从来没去研究过吧。