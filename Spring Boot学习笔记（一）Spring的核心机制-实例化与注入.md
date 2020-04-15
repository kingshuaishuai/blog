---
title: Spring Boot学习笔记（一）Spring的核心机制-实例化与注入
---

## stereotype annotations模式注解
1. `@Component` 最基础的模式注解，将组件/类/bean加入到容器中，以下注解以此为基础
    > 目前2-4三个注解与`@Component`目前在功能上没有任何区别，目前它们用于标注类的作用
    
2. `@Service` 标明这个类是一种服务
3. `@Controller` 标明这个类是控制器
4. `@Repository` 标明这个类是个仓储，通常在数据库访问时候，模型上会打这个注解、
5. `@Configuration` 与上面注解不同，它是把一组bean加入到容器中的注解，比上面的容器更灵活

注入时使用`@Autowired`注解进行属性注入，例：

``` java
@RestController
@RequestMapping("/v1/banner")
public class BannerController {
    @Autowired
    private Diana diana;

    @GetMapping("/test")
    public String test() {
        diana.r();
        return "hello yishuai";
    }
}
```

## IOC容器实例化和注入的时机

**默认机制：立即/提前实例化**

在SpringBoot应用启动过程中IOC容器就开始进行对象的实例化，同时将实例化好的对象注入到代码中

**延迟实例化：`@Lazy`**

通过使用`@Lazy`注解可以延迟实例化，在调用时再进行实例化
> `注意`：
> 所有依赖该类的类都要加`@Lazy`注解，否则还是默认机制，无特殊需求不要开启`@Lazy`

## 属性注入与构造注入

**属性注入：最简单方便，但是不规范**
属性注入需要使用`@Autowired`注解，更推荐使用构造注入
``` java
@RestController
@RequestMapping("/v1/banner")
public class BannerController {
    @Autowired
    private Diana diana;

    @GetMapping("/test")
    public String test() {
        diana.r();
        return "hello yishuai";
    }
}
```

**构造注入：推荐**
构造注入可以不加`@Autowired`注解,添加也不会报错

``` java
@RestController
@RequestMapping("/v1/banner")
public class BannerController {
    private Diana diana;
    
    // @Autowired
    public BannerController(Diana diana) {
        this.diana = diana
    }

    @GetMapping("/test")
    public String test() {
        diana.r();
        return "hello yishuai";
    }
}
```

**setter注入**

``` java
@RestController
@RequestMapping("/v1/banner")
public class BannerController {
    private Diana diana;

    @Autowired
    public void setDiana(Diana diana) {
        this.diana = diana;
    }

    @GetMapping("/test")
    public String test() {
        diana.r();
        return "hello yishuai";
    }
}
```

## 一个接口多个实现类的处理

**@Autowried自动注入**
如果一个接口只有一个实现类，默认是按类型注入（bytype），此时变量名可以任意定义，不会报错

![](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1581847686382.png)

如果一个接口有多个实现类，则不是按类型注入（bytype），变成按名称注入（byname），此时变量名不能随便定义，否则会报错
![](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1581847807238.png)
![](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1581847856182.png)
![](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1581847923149.png)

`@Autowried`默认按类型注入
找到0个bean: 报错
找到1个bean: 直接注入
找到多个bean: 不一定报错，变为byname方式注入

**主动注入`@Qualifier("名称")`**
`@Autowried`自动注入是一种被动注入方式，使用`@Autowried`+`@Qualifier("名称")`可以主动注入，解决上述报错问题。
![](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1581848883967.png)

## 面向对象中应对变化的解决方案
1. 制定一个interface,然后用多个类实现同一个interface --- 策略模式
2. 通过一个类的属性配置解决变化，通过配置文件读取具体的属性值。例如使用MySQ时候引入的配置文件

## `@Configuration`的使用

``` java
@Configuration
public class HeroConfiguration {
    @Bean
    public ISkill camille() {
        return new Camille();
    }
}
```
通过`@Configuration`可以手动将类加入到容器，里面定义的Bean要手动实例化，并加上`@Bean`注解。例如上方`Camille`，在`Camille`类中就不需要使用`@Component`了

**`@Configuration`的意义**
`@Configuration`注解的类称为配置类，用来代替xml文件。它可以应对变化，简单的`@Component`如果某个类的实例化需要传入参数是无法满足的，可以通过`@Configuration`和`@Bean`的方式书写配置类来满足这样的需求。方便灵活的管理Bean

## 包扫描机制
通过`@ComponentScan`进行包扫描，`@SpringBootApplication`中包含该注解。

## 策略模式的变化方案
1. byname 切换bean name
2. `@Qualifier`指定bean
3. 有选择地只注入一个bean,注释掉其他bean上的`@Component`
4. 使用`@Primary`注解

## `@Conditional`条件注解
用来白你写自定义条件注解：
`@Conditional + Condition接口`

## 内置成品条件注解
1. `@ConditionalOnProperty`, matchIfMissing为true时，如果没有找到（没定义，如果定义了但没有匹配到任何策略会报错）策略则设置它为默认策略。

``` java
 @Configuration
public class HeroConfiguration {
    @Bean
//    @Conditional(DianaCondition.class)
    @ConditionalOnProperty(value = "hero.condition", havingValue = "diana", matchIfMissing = true)
    public ISkill diana() {
        return new Diana();
    }

    @Bean
//    @Conditional(IreliaCondition.class)
    @ConditionalOnProperty(value = "hero.condition", havingValue = "irelia")
    public ISkill irelia() {
        return new Irelia();
    }
}
```
2. @ConditionalOnBean 当SpringIoc容器内 存在指定Bean的条件
3. @ConditionalOnClass 当SpringIoc容器内 存在指定Class的条件
4. @ConditionalOnException 基于spEL表达式作为判断条件
5. @ConditionalOnJava 基于JVM版本作为条件判断
6. @ConditionalOnJndi 在JNDI存在时查找指定的位置
7. @ConditionalOnMissingBean 当SpringIoc容器内不存在指定Bean的条件
8. @ConditionalOnMissingClass 当SpringIoc容器内不存在指定Class的条件
9. @ConditionalOnNotWebApplication 当前项目不是web项目的条件
10. @ConditionalOnProperty 指定的属性是否有指定的值
11. @ConditionalOnResource 类路径是否有指定的值
12. @ConditionalOnSpringCandidate 当指定Bean在SpringIoc容器内只有一个，或者虽然有多个但是指定首选的Bean
13. @ConditionalOnWebApplication 当前项目是web项目的条件

## 自动配置/自动装配
第三方库的引入：
1. 安装库
2. 引入库

学习目标：
1. 原理是什么
2. 为什么要有自动装配

## SPI机制/思想
SPI全名为 Service Provider Interface

![](https://www.github.com/kingshuaishuai/static_resource/raw/master/assets/1586918964491.png)

为了解决变化  的  模块解决方案，与@Component, @Primary, @Qualifier, @Congiguration的不同在于，他们的粒度不同，SPI的目的为解决一种方案的整体变化，而其他几种则是解决类的变化
