# 一个前端的java后端之旅(二) 面向对象绕不过？那就干掉它(1)

![](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1577506338111.png)
java跟我们用过的（c++/JavaScript）有很大的不同，尽管都支持面向对象，但你仍可以在c++,JavaScript中不使用类和对象，而java就不同了，就连`main`入口方法都是类的方法，它是一个纯粹的面向对象的语言，所以面向对象在java学习中是无法绕过的，那既然绕不过，就先干掉他，在适应了它的规则后再深入了解它。

## java中的类Class

说到类，其实现在做前端的同学也会很熟悉，如果你使用过TypeScript，那应该对类的概念就更清楚不过了，这里不对类和对象的概念再做无意义的描述，让我们直接开始coding，`复习`一下类和对象的基本使用方法。

由于没有很好的想法，那就创造一个接近实际的需求，我们来用`收货地址`来熟悉类和对象基本使用

## 创造需求---收货地址类
根据[上一篇文章](https://github.com/kingshuaishuai/blog/blob/master/%E4%B8%80%E4%B8%AA%E5%89%8D%E7%AB%AF%E7%9A%84java%E5%90%8E%E7%AB%AF%E4%B9%8B%E6%97%85(%E4%B8%80)%20%E4%BB%8EPackage%E5%85%A5%E6%89%8BJava.md)中创建的java项目，我们在src下创建一个包`learning.oop`,然后在里面创建一个Class文件`DiliveryAddress`，注意类名最好大写开头，驼峰结构命名，idea会自动帮我们创建与Class文件名相同的类，文件名与类名保持一致。

```java
package learning.oop;

public class DeliveryAddress {
}

```

好了，现在该想想里面应该设计什么字段了。我就直接提供了：

- id: 收货地址本身需要一个id，来代表具体哪一个收货地址
- userId: 用户Id
- receiverName: 收件人名字
- receiverPhone: 收件人电话
- receiverProvince: 省
- receiverCity: 市
- receiverDistrict: 区
- receiverAddress: 详细地址
- receiverZip: 邮政编码
- isDefault: 是否是默认地址
- createTime: 创建时间
- updateTime: 修改时间

你是否露出了惊讶的表情，长大了嘴巴，心想，不就是个例子嘛，要不要这么认真，举个猫狗动物的例子它不香吗？要搞这么多字段的东西。（请合拢你张大的嘴巴）,我们不是儿戏，我们要通过接近真实的东西快速上手java。

## 初始化属性

在类中，我们可以包含仅表示数据的属性以及操作数据的方法，对于表示数据的属性，尽量使用`private`关键字，不要把属性直接暴露出去，如果你有修改和获取数据的需求，则应该使用`修改器setter`和`获取器getter`的方式来进行。

接下来先根据上述需求，来创建类中的属性，设计过程中想想它应该是哪种数据类型

```java
import java.util.Date;

public class DeliveryAddress {
    private Integer id;
    private Integer userId;
    private String receiverName;
    private String receiverPhone;
    private String receiverProvince;
    private String receiverCity;
    private String receiverDistrict;
    private String receiverAddress;
    private String receiverZip;
    private Boolean isDefault;
    private Date createTime;
    private Date updateTime;
}

```
以上，我们使用了四种数据类型，分别是`Integer`,`String`,`Boolean`,`Date`类型，Date类型引入了一个`java.util.Date`的包，说明它是那个包中所定义的一种类，并不是基础的数据类型。其他类型也很容易认，认真观察这些类型都是大写开头，所以他们本质也是一种类，这种形式叫做包装类。说到包装了类，有些包装类还对应有基础类型(比如int Interger都可以定义整数)。

比如int类型为整型（定义整数用的），他是定义整数变量的基本类型，对应的Interger是它的包装类，定义的是一种对象，纯粹定义的数字可以理解为就是一种数字，但是数字要想使用一些例如`toString()`的方法，就要借助对象，所以就会进行Integer类型的包装，如果用int定义的变量，在使用方法的时候java会自动帮我们进行包装，在java中这种转变叫做自动装箱，说到这里，如果用int变量接收一个Interger定义的变量，java就会自动拆除变量的方法，让他变成一个纯数字，在java中这种转变就是自动装箱（手动拆箱装箱自己理解咯）。

如果想详细了解java中的变量，可以自行查阅资料，但更建议的是边学边记。这里就不再做拓展（好吧，是因为我会的也不多）。

## 添加无参构造函数

既然是前端者的后端之旅，那么大家对JavaScript的构造函数一定不会陌生，它是通过`constructor`进行定义的，java不同于此，是通过与类名相同的方法来进行构造函数的定义的，如果我们不定义构造函数，系统会默认给我们添加一个没有参数且函数体为空的构造函数，当然我们也可以自定义带参数的构造函数。

```java
package learning.oop;

import java.util.Date;

public class DeliveryAddress {
    private Integer id;
    private Integer userId;
    private String receiverName;
    private String receiverPhone;
    private String receiverProvince;
    private String receiverCity;
    private String receiverDistrict;
    private String receiverAddress;
    private String receiverZip;
    private Boolean isDefault;
    private Date createTime;
    private Date updateTime;

    // 无参构造函数
    public DeliveryAddress(){
        this.createTime = new Date();
    }

    public Date getCreateTime() {
        return createTime;
    }
}

```

这里添加了一个无惨构造函数，里面对createTime做了初始化操作，因为这个属性应该是不可以改变的，而且是在创建对象时候就应该拥有值的。下面还定义了一个`public`方法，类型为Date，返回createTime，注意构造函数和getCreateTime中的写法，一个带this,一个不带，两种形式都可以。

> `tips`
> 注意构造函数也要加`public`修饰符

现在我们去src下的Main方法里面实例化几个对象试试看。

```java
import learning.oop.DeliveryAddress;

public class Main {

    public static void main(String[] args) {
        DeliveryAddress address1 = new DeliveryAddress();
        System.out.println(address1);
        System.out.println(address1.toString());
        System.out.println(address1.getClass());
        System.out.println(address1.getCreateTime());
        System.out.println(address1.getCreateTime().getTime());
    }
}

```

![](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1577511803014.png)

我打印了其中的`toString()`和`getClass()`方法的运行结果，`toString()`与JavaScript中的方法类似，在我们呢直接打印这个变量的时候，就会输出它的`toString()`结果，因此我们可以看到直接打印与打印`toString()`结果相同，然后好奇打印了`getClass()`，得到了这个对象对应的类所在具体地址（包名）`learning.oop.DeliveryAddress`



通过`getCreateTime()`方法，我们也成功获得了对象创建的时间，，这个createTime其实是个Date类型的对象，我们可以通过它上面的方法获取自己想要的时间形式，例如想要时间戳，可以通过`getTime()`方法获取。


大家可以通过[文档](https://docs.oracle.com/javase/9/docs/api/index.html?overview-summary.html)看看Date类定义了哪些方法。其实完全不用全部看，自己需要用哪些就看哪些。

## 添加带参构造函数/方法

what？构造函数可以添加多个？使得没错，在C++/Java中都是可以的，这种特点叫做重载，TypeScript中也支持重载，但由于它本身还是由JavaScript写的，所以它的重载并不是真正的重载。

```java
    public DeliveryAddress(
        Integer userId,
        String receiverName,
        String receiverPhone,
        String receiverProvince,
        String receiverCity,
        String receiverDistrict,
        String receiverAddress,
        String receiverZip
    ) {
        this.userId = userId;
        this.receiverName = receiverName;
        this.receiverPhone = receiverPhone;
        this.receiverProvince = receiverProvince;
        this.receiverCity = receiverCity;
        this.receiverDistrict = receiverDistrict;
        this.receiverAddress = receiverAddress;
        this.receiverZip = receiverZip;
        this.isDefault = false;
    }
    
    public DeliveryAddress(
            Integer userId,
            String receiverName,
            String receiverPhone,
            String receiverProvince,
            String receiverCity,
            String receiverDistrict,
            String receiverAddress,
            String receiverZip,
            Boolean isDefault
    ) {
        this.userId = userId;
        this.receiverName = receiverName;
        this.receiverPhone = receiverPhone;
        this.receiverProvince = receiverProvince;
        this.receiverCity = receiverCity;
        this.receiverDistrict = receiverDistrict;
        this.receiverAddress = receiverAddress;
        this.receiverZip = receiverZip;
        this.isDefault = isDefault;
    }
```
这次我添加了两个带参的构造方法，他们的唯一不同就在于`isDefault`属性的设置，因为java不支持设置默认参数，所以只能用重载的形式来进行设置。同时仔细观察这里的所有属性设置都会使用`this.xxx = xxx`，而没有直接使用`xxx = xxx`，后者其实是错误的，这样java分辨不出来你到底要给谁设置值，所以如果属性名跟参数名相同，那就必须加`this`。

我们可以去Main方法中添加两个实例试试

```java
import learning.oop.DeliveryAddress;

public class Main {

    public static void main(String[] args) {
        DeliveryAddress address1 = new DeliveryAddress();
        DeliveryAddress defaultAddress = new DeliveryAddress(
                1,
                "张三",
                "13122888888",
                "上海",
                "上海",
                "浦东新区",
                "XX小区XX楼XX户",
                "20000",
                true
                );
        DeliveryAddress address2 = new DeliveryAddress(
                1,
                "张三",
                "13122888888",
                "上海",
                "上海",
                "浦东新区",
                "XX小区XX楼XX户",
                "20000",
        );

        System.out.println(address1.getCreateTime());
        System.out.println(defaultAddress.getCreateTime());
        System.out.println(address2.getCreateTime());
    }
}

```

![enter description here](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1577513261351.png)

我们打印了三个地址的创建时间，但一看，似乎不对啊，为什么后面两个是null，难道不应该是正常时间？

这其实是正常现象，三个构造方法是独立的，后两个调用的时候并没有去设置时间，所以不会有。

好吧，这真的是故意的，但又一点不是故意的我忘记重写toString()方法了，那么下面我们来解决这些问题，同时两个带参的构造函数似乎很大一部分代码是重复的，我们也顺手将代码重复的问题进行解决。

**改良**
```java
    ...
    private static int counter = 0;
    
    public DeliveryAddress(){
        this.id = DeliveryAddress.counter;
        DeliveryAddress.counter++;
        this.createTime = new Date();
        this.updateTime = this.createTime;
    }

    public DeliveryAddress(
        Integer userId,
        String receiverName,
        String receiverPhone,
        String receiverProvince,
        String receiverCity,
        String receiverDistrict,
        String receiverAddress,
        String receiverZip
    ) {
        this();
        this.userId = userId;
        this.receiverName = receiverName;
        this.receiverPhone = receiverPhone;
        this.receiverProvince = receiverProvince;
        this.receiverCity = receiverCity;
        this.receiverDistrict = receiverDistrict;
        this.receiverAddress = receiverAddress;
        this.receiverZip = receiverZip;
        this.isDefault = false;
    }

    public DeliveryAddress(
            Integer userId,
            String receiverName,
            String receiverPhone,
            String receiverProvince,
            String receiverCity,
            String receiverDistrict,
            String receiverAddress,
            String receiverZip,
            Boolean isDefault
    ) {
        this(
            userId,
            receiverName,
            receiverPhone,
            receiverProvince,
            receiverCity,
            receiverDistrict,
            receiverAddress,
            receiverZip
        );
        this.isDefault = isDefault;
    }

    @Override
    public String toString() {
        String res = "id: " + this.id + "\n" +
                "userId: " + this.userId + "\n" +
                "receiverName: " + this.receiverName + "\n" +
                "receiverPhone: " + this.receiverPhone + "\n" +
                "receiverCity: " + this.receiverCity + "\n" +
                "receiverDistrict: " + this.receiverDistrict + "\n" +
                "receiverAddress: " + this.receiverAddress + "\n" +
                "receiverZip: " + this.receiverZip + "\n" +
                "isDefault: " + this.isDefault + "\n";
        return res;
    }
```

这里我们还是三个构造方法，第一个无惨，后两个有参，第三个比第二个多个isDeafult，这些都没问题，前面提出的问题我们又是怎么解决的呢？我们通过给第二个构造函数添加`this()`，这代表如果要执行第二个构造方法，就会先执行第一个，给第三个构造方法添加了个`this(xxxxx)`里面传进了很多参数，这代表要使用第三个构造方法，就必须调用第二个，同时，也做到了代码的简化，第三个构造函数仅仅是比第二个多了个`isDefault`的参数而已，其他代码一样，那就直接调用第二个就好了。

最后实现了个`publish String toString()`方法,这个方法奇怪的地方是在它头上多了个`@Override`的东西，这个东西叫做注解，可以暂时理解为要重写方法，就必须用这个注解进行标识（其实是因为我也不会注解，后面再研究好了）。

> `tips`
> 这里注意一下又添加了一个属性，`private static int counter = 0`,这个属性与其他不同的在于多了个`static`关键词的描述，并且直接初始化为0，添加了`static`的属性与JavaScript中的作用一样，它就变成了类的属性，而不是某个实例的属性，这里通过它模拟了收货地址id的自增。当然，也可以使用`static`创建静态方法，它也是属于类的方法，既可以通过类名调用，也可以通过实例进行调用。

然后我们可以回去Main方法中完善一下三个实例的输出
```java
import learning.oop.DeliveryAddress;

public class Main {

    public static void main(String[] args) {
        DeliveryAddress address1 = new DeliveryAddress();
        DeliveryAddress defaultAddress = new DeliveryAddress(
                1,
                "张三",
                "13122888888",
                "上海",
                "上海",
                "浦东新区",
                "XX小区XX楼XX户",
                "20000",
                true
                );
        DeliveryAddress address2 = new DeliveryAddress(
                1,
                "张三",
                "13122888888",
                "上海",
                "上海",
                "浦东新区",
                "XX小区XX楼XX户",
                "20000"
        );

        System.out.println(address1.getCreateTime());
        System.out.println(defaultAddress.getCreateTime());
        System.out.println(address2.getCreateTime());
        System.out.println(address1);
        System.out.println(defaultAddress);
        System.out.println(address2);
    }
}

```
![](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1577514365017.png)
![](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1577514430367.png)

这次就成功看到了三个创建时间的输出，并且看到直接打印对象，打印出了我们`toString()`方法返回的值。也证明之前说的直接打印对象其实输出的就是`toString()`。

## 属性默认值

在上面运行结果中我们看到第一个地址的很多属性值都是null，这明显是不行的，如果这些属性没有被赋值，我们需要给他们添加默认值，添加默认值有两种方式：
1. 可以在设置属性的时候直接初始化话，就像上面所说的`private static int counter = 0`一样，只不过普通属性不加static
2. 也可以像`createTime`一样放在无惨构造函数里进行初始化

这里我将其完善一下，使用第一种方式进行初始化。
```java
    private Integer id;
    private Integer userId = -1;
    private String receiverName = "";
    private String receiverPhone = "";
    private String receiverProvince = "";
    private String receiverCity = "";
    private String receiverDistrict = "";
    private String receiverAddress = "";
    private String receiverZip = "";
    private Boolean isDefault = false;
    private Date createTime;
    private Date updateTime;
    private static int counter = 0;
```

回到Main函数重新运行代码结果为
![](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1577515535594.png)

这样就舒服多了，未填的值直接为设置的初始值。