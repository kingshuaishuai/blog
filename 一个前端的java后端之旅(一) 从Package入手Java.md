# 一个前端的java后端之旅(一) 从Package入手Java

![](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1576812813818.png)

作为一个想要进阶java后端但又恐惧java的前端工程师，你只是缺少适合自己的教程，大家都是编程的人，完全没必要从int,String,Boolean,if,else,witch,for,while....这种东西学起了。那就让我开始，先行寻找java后端之旅的路线，然后跟大家一起快速进击java后端吧。

## 首先学会创建Java项目

了解Java包之前先看看如何使用它。使用IDEA工具创建一个普通的java项目。

这里对于IDEA工具的下载安装以及jdk的下载就不介绍了（如果真的不会安装jdk,[点击查看详细安装步骤](https://www.runoob.com/java/java-environment-setup.html)），但是你必须保证已经安装了IDEA工具以及jdk才能继续看下去:

**step1 点击 Create New Project 创建一个工程**
![](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1576801869087.png)

**step2 这里必须保证的一点就是蓝色框中jdk必须选中一个，如果没有，就点击右边的New将你下载的jdk添加进来，其余选框均不用勾选，点击next**
![](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1576802312836.png)

**step3 出现选择模板的界面，默认未勾选，为了方便可以勾选并选择Java Hello Word模板，点击next**
![](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1576802387507.png)

**step4 第一行输入项目名称，第二行选择一下项目存储路径，点击Finish完成项目创建**
![](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1576806980018.png)

**step5 进入Main.java后右键菜单里点击Run 'Main'就可以体验自己的第一个java程序了**
![](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1576807239574.png)

## Java中的文件管理

作为一个职业的前端选手，你可能经常游走于vue/react项目，对项目src目录有着深刻的见解，自己也会设计符合自己风格的目录存放相应的代码，那么在java中也是一样的，我们可以在src中创建不同的目录来管理不同的Java代码。

**来个简单的例子：**
当你准备创建Directory（WebStorm里是Directory）的时候，你突然发现，WDMY 竟然没有这个选项，但是有个你在WebStorm里没见过的东西，那就是`Package`，没错，Java就是通过这个叫`Package`的东西来管理文件目录的。

![](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1576807632779.png)

好了，那就创建一个`learning.base`的`Package(包)`吧，然后你会发现src目录下多了个`learning.base`的包。
![](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1576808134945.png)

你可以点击右上角红色框中的齿轮按钮，将`Compact Middle Packages`选项的勾去掉，就可以看到`learning.base`已经变成了一个`learning/base`的级联目录。

![](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1576808308962.png)
![](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1576808393902.png)

此时就可以清楚的解释java中的包到底是什么东西，他就是用来管理和组织java项目文件的一种目录。

## 包和文件夹到底有什么区别？
当了解到包的时候，我们知道包的本质就是个文件夹或者说是目录，但是你（其实是我）可能会有些许困惑，为什么java把目录叫做包，为什么还需要包这个概念，我们写前端也有目录啊，也没见人家把目录抽象成什么包的概念。

那么既然叫做包，它肯定不是一种普通目录，那么它跟普通目录到底有什么区别呢？

在寻找答案的时候，发现网上的回答都说没有区别，但我不能放弃啊，没有区别那凭什么会有包的概念而不是叫目录。

个人的理解是这样的：java的包是一种接受java规范的目录结构。它与普通的目录却别就在于它受制于某种规定，而普通目录你可以随意创建。

这种规范包含：
1. 命名规范：包名只能是字符串、数字、下划线、$且不能以数字开头，一般是小写英文字（大写不报错）。
    - 这个规则非常类似变量的命名，其实这也是有原因的。你可以在learning.base下新建一个Hello的 Java Class
    - ```java
        package learning.base;

        public class Hello {
        
        }
      ```
    - 你可以看到`package learning.base`包名声明在java代码中，如果包名中出现什么加减乘除符号，是不是不合理，所以这一规范非常重要。
    
2. 包结构的组织规范：包的结构一般以域名倒写来组织，包名之间以点号`.`来连接。比如我有个域名叫做`geekrole.com`我就可以这样组织包名`com.geekrole.user`。
    - 刚开始你可能会有疑问为什么不正着写，但如果你把包拆成目录结构`com/geekrole/user`，你就知道了，目录的命名是个总分结构的，所以包名也是。

## 包的用途

我们已经知道，java中的包是来管理和组织java项目文件的一种目录，所以我们可以将某一类代码封装到同一个包中，java提供的很多标准库文件也是通过包的形式进行组织，只不过它是全局的，并以java开头，比如`java.lang`,`java.util`等。

**打破畏惧心理**

此时你（其实是我）可能会有种面对未知的忧虑和害怕，我怎么知道它需要有些什么包，它的包有些什么方法，每次调用导入包怎么导入...

好吧，说实话，这种担心应该是多余的，Idea可以帮我们自动导入一些用到的包, 你可以试试下面的代码，当你写到Scanner回车后会自动导入Scanner包。如果是自定义包，idea也会有相应提示，以帮助我们自动导入。

```java
package learning.base;

import java.util.Scanner;

public class Hello {
    public static void main(String[] args) {
        // 输入两个数字比较大小，输出大数
        Scanner sc = new Scanner(System.in);
        int num1 = sc.nextInt();
        int num2 = sc.nextInt();
        int res = Math.max(num1, num2);
        System.out.println(res);
    }
}
```

解决了包导入的问题，那么如何知道有哪些包，这就是个积累的问题了，如果开发过程中需要用到时间相关，那么就查一查时间相关的包，看看有哪些方法可以供我们使用，一下是java9官方文档，可以找到你想要用的各种包[(java9文档)](https://docs.oracle.com/javase/9/docs/api/index.html?overview-summary.html)

> `tips`
> 可以注意一下包的声明方式与导入方式。package关键字用于声明包，import关键字用于导入包

## 什么是jar包和war包？

既然说到了包，那就不妨把jar包和war包也简单说明，避免后续它们给我们编程带来未知的恐惧。

### jar包是个啥玩意

你可能不了解jar包，但一定知道并用过zip压缩工具，比如2345好压，通过它可以把很多文件甚至文件夹打包到一起，方便传输给别人，同时它也拥有压缩算法，可以把文件体积变小。

jar包的本质就是java中的zip包,本质相同，但既然他是jar，就一定跟zip包有区别，jar与zip的区别是什么：jar文件不仅用于压缩和发布，而且还用于部署和封装库、组件和插件程序，并可被像编译器和 JVM 这样的工具直接使用（[此句来自百度](https://zhidao.baidu.com/question/2208202223415308508.html)）。

也就是说jar包可以进一步将我们封装的各种类文件（可能通过包形式阻止起来的类）进一步做个打包文件，并且这个文件在使用的时候完全不用解压缩，可以直接使用。除此之外还拥有一些zip包的优势，比如压缩。

到此，jar包到底有什么优点自己也可以大致推断出一二了，如果实在强迫症，百度搜一搜，然后你会发现自己也能根据zip包和以上jar与zip的区别猜出来。

### war包又是个啥玩意

说了jar包，怎么又有个war包，直白一点，war包就是把一个web应用打包起来的包，并且跟jar包类似，它也可以直接运行。

我们可以这样理解，jar包是对各种工具的打包，war则是以整个网站，或者一个网站的一个模块的打包，它可以直接运行。

## 小结

虽然本文并没有什么实质性的编程相关的东西，但它可以从另一个角度来帮助一些没有java的基础的朋友快速对java有个大致的认识。

**以下为闲谈**
说到为什么要写java相关的文章，就想起了作为一个计科专业的学生，竟然只学过c/c++，都没认真玩过java，然后兴趣和机会让我接触了前端，但java始终是块心病啊。

可能java的内容真的有点多，一本正常点的java SE语法书也要四五百页，看起来真的像本字典。就说说我最近看的一本java书《写给大忙人的JavaSE9核心技术》，500多页也像字典一样，`大忙人`也没有空看字典啊。

在网上找了找可以快速学习的资料，但是没找到想要的，要么就是介绍int,Integer,String,Boolean, if, else, switch要么开始介绍一堆面向对象的特点，jdk、jre是什么。这种教程内容是很全，可不适合我快速学习，作为一个学过c系语言（c,c++,java,javascript都算）的人，一些基础语法，if,else,switch,while,do while,for都一模一样啊，重复学习并没有多大的必要。

所以想着那就自己开始挑着学吧，然后记录下来，也为一些前端朋友进阶java认真做点事，大家都是编程的，很多不用说的东西那就别说了，快速熟悉java，尽快过度到spring boot等框架，开始真正的后端编程才是重要的，而不是追求内容够多，多了学不完要他何用？
