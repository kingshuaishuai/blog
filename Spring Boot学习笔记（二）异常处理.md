---
title: Spring Boot学习笔记（二）异常处理
---

## Spring异常处理机制

给类加`@ControllerAdvice`注解，标明异常处理类
给方法添加`@ExceptionHandler`注解，标明异常处理器，需要接受一个参数`@ExceptionHandler(value=Exception.class)`，指定处理的Exception类型

## 异常分类
Throwable

Error     错误
Exception 异常

异常又分为：
1. CheckedException （程序中没有这个类，使用Exception类即可） 在程序中必须处理的异常，不处理编译过程中会报错
2. RuntimeException （有这个类） 程序运行过程中抛出的异常

CheckedException  是真正的bug
RuntimeException  程序运行中的未知情况（比如查询数据库没找到记录）

异常也分为：
1. 已知异常   考虑到的异常
2. 未知异常   编程中没有考虑到的bug，如5/0，分母为0，但是程序员没有考虑到这种情况

## 已知异常、未知异常
未知异常： 对于前端开发者和用户都是无意义的。服务端开发者代码逻辑有问题，需要模糊地返回给前端，具体信息可以打印日志

HttpStatus状态码的设置
1.`@ResponseStatus(code = HttpStatus.xxxx)`通过注解进行设置，缺点：不够灵活，无法通过代码动态修改Http错误码
2. 手动通过`ResponseEntity<T>`类进行设置
   

``` java
 @ExceptionHandler(HttpException.class)
    @ResponseBody
    public ResponseEntity<UnifyResponse> handleHttpException(HttpServletRequest req, HttpException e) {
        String requestUrl = req.getRequestURI();
        String method = req.getMethod();
        System.out.println(e);
        UnifyResponse message = new UnifyResponse(e.getCode(), "message", method + " " + requestUrl);
//        设置HTTP头
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
//        设置HTTP状态
        HttpStatus httpStatus = HttpStatus.resolve(e.getHttpStatusCode());
//        通过ResponseEntity进行返回
        ResponseEntity<UnifyResponse> r = new ResponseEntity(message, headers, httpStatus);
        return r;
    }
```

## 读取properties配置文件
1. @PropertySource 设置配置文件路径，以`classpath:`开头
2. @ConfigurationProperties
``` java
@ConfigurationProperties(prefix = "lin")  //设置前缀
@PropertySource(value = "classpath:config/exception-code.properties")
@Component
public class ExceptionCodeConfiguration {
    private Map<Integer, String> codes = new HashMap<>();

    public String getMessage(Integer code) {
        String message = this.codes.get(code);
        return message;
    }

    public Map<Integer, String> getCodes() {
        return codes;
    }

    public void setCodes(Map<Integer, String> codes) {
        this.codes = codes;
    }
}

```

## 根据目录自动生成路由前缀

``` java
public class AutoPrefixUrlMapping extends RequestMappingHandlerMapping { // 继承RequestMappingHandlerMapping类
    @Value("${missyou.api-package}")
    private String apiPackagePath;

// 重写getMappingForMethod方法
    @Override
    protected RequestMappingInfo getMappingForMethod(Method method, Class<?> handlerType) {
        RequestMappingInfo mappingInfo = super.getMappingForMethod(method, handlerType);

        if(mappingInfo != null) {
            String prefix = this.getPrefix(handlerType);
            return RequestMappingInfo.paths(prefix).build().combine(mappingInfo);
        }
        return mappingInfo;
    }

    private String getPrefix(Class<?> handlerType) {
        String packageName = handlerType.getPackage().getName();
        String dotPath = packageName.replace(apiPackagePath, "").replace(".", "/");
        return dotPath;
    }
}
```

实现WebMvcRegistrations接口并加入容器 

``` java
@Component
public class AutoPrefixConfiguration implements WebMvcRegistrations {
    public RequestMappingHandlerMapping getRequestMappingHandlerMapping() {
        return new AutoPrefixUrlMapping();
    }
}
```

## 参数校验
**参数识别：**
1. path参数`@PathVariable`
2. query参数`@RequestParam`
3. requst body: `@RequestBody`

DTO: Data Transfer Object 数据传输对象

**Lombok:**
1. `@Getter` 编译阶段生成getter
2. `@Setter` 编译阶段生成setter
3. `@Data` 综合注解
4. `@AllArgsConstructor` 生成全部参数的构造函数
5. `@NoArgsConstructor` 生成无参构造函数
6. `@NonNull` 字段不为空
7. `@RequiredArgsConstructor` 生成部分（必填字段）构造函数
8. `@Builder` 

``` java
PersonDTO.builder().name("yishuai").age(18).build()
```

**JSR规范提案（没有实现，各厂商可以自己实现）**
LomBok  遵循JSR-269规范
JSR-303 是Bean Validation的规范提案

Hibernate-Validator 实现了JSR-303规范提案
使用方式：
1. 类上使用`@Validated`,参数校验使用详细注解比如`@Max(10, message="不可以超过10哦")`
2. 级联校验，在一个DTO中的一个字段是另外一个DTO对象（包含校验），需要在该字段添加`@Valid`注解

``` java
@Builder
@Getter
public class PersonDTO {
    @Length(max = 10, min = 2)
    private String name;
    private Integer age;
    @Valid
    private SchoolDTO school;
}
```
3. `@Valid`与`@Validated`区别：`@Valid`是java提供的标准，`@Validated`是Spring对`@Valid`的扩展，在一定程度上`@Valid`可以代替`@Validated`。级联校验建议使用`@Valid` 其他情况（开启校验）使用`@Validated`

**自定义校验注解**
注解：
``` java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Constraint(validatedBy = PasswordValidator.class) // 用来关联校验的类
public @interface PasswordEqual {
    String message() default "passwords are not equal";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```
注解解释器：

``` java
public class PasswordValidator implements ConstraintValidator<PasswordEqual, PersonDTO> {
//第二个类型参数为要自定义注解修饰的目标类型
    @Override
    public boolean isValid(PersonDTO value, ConstraintValidatorContext context) {
        String password1 = personDTO.getPassword1();
        String password2 = personDTO.getPassword2();
        boolean match = password1.equals(password2);
        return match;
    }
}
```

