---
title: Swagger简单入门
date: 2020-10-16 21:36:13
tags: 
  - Swagger
categories: java
---



## Swagger简介

## **前后端分离**

- 前端 -> 前端控制层、视图层
- 后端 -> 后端控制层、服务层、数据访问层
- 前后端通过API进行交互
- 前后端相对独立且松耦合

## **产生的问题**

- 前后端集成，前端或者后端无法做到“及时协商，尽早解决”，最终导致问题集中爆发

## **解决方案**

- 首先定义schema [ 计划的提纲 ]，并实时跟踪最新的API，降低集成风险

## **Swagger**

- 号称世界上最流行的API框架
- Restful Api 文档在线自动生成器 => **API 文档 与API 定义同步更新**
- 直接运行，在线测试API
- 支持多种语言 （如：Java，PHP等）
- 官网：https://swagger.io/

## SpringBoot集成Swagger

jdk 1.8 +

### 方式一:使用官方依赖

#### Maven依赖

```
<!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger2 -->
<dependency>
   <groupId>io.springfox</groupId>
   <artifactId>springfox-swagger2</artifactId>
   <version>2.9.2</version>
</dependency>
<!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger-ui -->
<dependency>
   <groupId>io.springfox</groupId>
   <artifactId>springfox-swagger-ui</artifactId>
   <version>2.9.2</version>
</dependency>
```

#### 编写一个配置类-SwaggerConfig来配置 Swagger

```
@Configuration //配置类
@EnableSwagger2// 开启Swagger2的自动配置
public class SwaggerConfig {  
}
```

#### 访问测试 ：http://localhost:8080/swagger-ui.html ，可以看到swagger的界面；

![image-20201016202312084](image-20201016202312084.png)

#### 配置Swagger

```java
package com.czm.swagger.config;

import org.springframework.boot.SpringBootConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.env.Environment;
import org.springframework.core.env.Profiles;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.service.VendorExtension;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

import java.util.ArrayList;


/**
 * @Author CZM
 * @create 2020/10/16 16:44
 */
@Configuration
@EnableSwagger2 // 开启Swagger2的自动配置
public class SwaggerConfig {



    @Bean
    public Docket docket(ApiInfo apiInfo, Environment environment) {
        // 设置要显示swagger的环境
        Profiles profiles = Profiles.of("dev", "test");
        // 判断当前是否处于该环境，环境配置：spring.profiles.active=dev
        // 通过 enable() 接收此参数判断是否要显示
        boolean flag = environment.acceptsProfiles(profiles);
        return new Docket(DocumentationType.SWAGGER_2).apiInfo(apiInfo)
                .groupName("czm") //配置API分组名
                .enable(flag)//配置是否启用Swagger，如果是false，在浏览器将无法访问
                .select()// 通过.select()方法，去配置扫描接口,RequestHandlerSelectors配置如何扫描接口
                /**
                 *  // 扫描所有，项目中的所有接口都会被扫描到
                 *  RequestHandlerSelectors.any()
                 *  // 不扫描接口
                 *  RequestHandlerSelectors.none()
                 *  // 通过方法上的注解扫描，如withMethodAnnotation(GetMapping.class)只扫描get请求
                 *  RequestHandlerSelectors.withMethodAnnotation(final Class<? extends Annotation> annotation)
                 *  // 通过类上的注解扫描，如.withClassAnnotation(Controller.class)只扫描有controller注解的类中的接口
                 *  RequestHandlerSelectors.withClassAnnotation(final Class<? extends Annotation> annotation)
                 *  RequestHandlerSelectors.basePackage(final String basePackage) // 根据包路径扫描接口
                 */
                .apis(RequestHandlerSelectors.basePackage("com.czm.swagger.controller"))
                // 配置如何通过path过滤,即这里只扫描请求以/czm开头的接口
                .paths(PathSelectors.ant("/czm/**"))
                /***其他参数
                 * PathSelectors.any() // 任何请求都扫描
                 * PathSelectors.none() // 任何请求都不扫描
                 * PathSelectors.regex(final String pathRegex) // 通过正则表达式控制
                 * PathSelectors.ant(final String antPattern) // 通过ant()控制
                 */
                .build();
    }

    @Bean
    //配置文档信息
    public ApiInfo apiInfo() {
        /***
         *    Contact contact = new Contact("联系人名字", "http://xxx.xxx.com/联系人访问链接", "联系人邮箱");
         *    return new ApiInfo(
         *            "Swagger学习", // 标题
         *            "学习演示如何配置Swagger", // 描述
         *            "v1.0", // 版本
         *            "http://terms.service.url/组织链接", // 组织链接
         *            contact, // 联系人信息
         *            "Apach 2.0 许可", // 许可
         *            "许可链接", // 许可连接
         *            new ArrayList<>()// 扩展
         *   );
         */
        Contact contact = new Contact("czm", "baidu.com", "1233@qq.com");
        return new ApiInfo("swagger的测试API", "Api Documentation的描述", "1.0版", "urn:tos",
                contact, "Apache 2.0", "http://www.apache.org/licenses/LICENSE-2.0", new ArrayList<VendorExtension>());
    }

    /***
     * 其他多个API组
     * @return
     */
    @Bean
    public Docket docket1() {
        return new Docket(DocumentationType.SWAGGER_2).groupName("1");
    }
    @Bean
    public Docket docket2() {
        return new Docket(DocumentationType.SWAGGER_2).groupName("2");
    }
}

```



#### 实体配置

1、新建一个实体类

```
@ApiModel("用户实体类")
public class User {

    @ApiModelProperty("用户账号")
    private String account;

    @ApiModelProperty("用户密码")
    private String password;

    public User(){

    }
    public User(String account, String password) {
        this.account = account;
        this.password = password;
    }

    public String getAccount() {
        return account;
    }

    public void setAccount(String account) {
        this.account = account;
    }
}

```

2、只要这个实体在**请求接口**的返回值上（即使是泛型），都能映射到实体项中：

```
@RequestMapping("/getUser")
public User getUser(){
   return new User();
}
```

![image-20201016210951882](image-20201016210951882.png)

注：并不是因为@ApiModel这个注解让实体显示在这里了，而是只要出现在接口方法的返回值上的实体都会显示在这里，而@ApiModel和@ApiModelProperty这两个注解只是为实体添加注释的。

@ApiModel为类添加注释

@ApiModelProperty为类属性添加注释



### 常用注解

Swagger的所有注解定义在io.swagger.annotations包下

下面列一些经常用到的，未列举出来的可以另行查阅说明：

| Swagger注解                                            | 简单说明                                             |
| ------------------------------------------------------ | ---------------------------------------------------- |
| @Api(tags = "xxx模块说明")                             | 作用在模块类上                                       |
| @ApiOperation("xxx接口说明")                           | 作用在接口方法上                                     |
| @ApiModel("xxxPOJO说明")                               | 作用在模型类上：如VO、BO                             |
| @ApiModelProperty(value = "xxx属性说明",hidden = true) | 作用在类方法和属性上，hidden设置为true可以隐藏该属性 |
| @ApiParam("xxx参数说明")                               | 作用在参数、方法和字段上，类似@ApiModelProperty      |
| @ApiImplicitParam()                                    | 作用在方法上，表示单独的请求参数                     |
| @ApiImplicitParams()                                   | 作用于方法，包含多个 @ApiImplicitParam               |

注：@ApiImplicitParam(name–参数ming，value–参数说明 ，dataType–数据类型 ，example–举例说明，required–是否必填,

paramType–参数类型 )

paramType表示参数放在哪个地方

- header-->请求参数的获取：@RequestHeader(代码中接收注解)
- query-->请求参数的获取：@RequestParam(代码中接收注解)
- path（用于restful接口）-->请求参数的获取：@PathVariable(代码中接收注解)
- body-->请求参数的获取：@RequestBody(代码中接收注解)
- form（不常用）

我们也可以给请求的接口配置一些注释

```
package com.czm.swagger.controller;

import com.czm.swagger.entity.User;
import io.swagger.annotations.*;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.ArrayList;
import java.util.List;

/**
 * @Author CZM
 * @create 2020/10/16 16:49
 */
@RestController
@Api(value="swagger的hello模块",tags={"用户操作接口"})
public class HelloController {

    @GetMapping("/czm/hello")
    public String hello() {
        return "hello";
    }

    @RequestMapping("/czm/getUser")
    @ApiOperation("获取一个默认user对象")
    @ApiResponses({@ApiResponse(code = 200, message = "good"),@ApiResponse(code = 401, message = "no power")})
    public User getUser(@ApiParam(value = "这是个参数",name = "param") String param){
        return new User(param+"123","123456");
    }

    @GetMapping("/czm/getUser2/{account}/{password}")
    @ApiOperation("获取一个指定user对象")
    @ApiImplicitParams({
            /**
             * name–参数ming
             * value–参数说明
             * dataType–数据类型
             * paramType–参数类型
             * example–举例说明
             * required–是否必填
             */
            @ApiImplicitParam(name="account",value="用户名",dataType="String", paramType = "path",
            example = "username",required = true),
            @ApiImplicitParam(name="password",value="用户密码",dataType="String", paramType = 						"path",example = "password")     
    })
    public User getUser2(@PathVariable("account") String account, 
    					@PathVariable("password") String password) {
        System.out.println(account+password);
        return new User(account,password);
    }
}
```

这样的话，可以给一些比较难理解的属性或者接口，增加一些配置信息，让人更容易阅读！

相较于传统的Postman或Curl方式测试接口，使用swagger简直就是傻瓜式操作，不需要额外说明文档(写得好本身就是文档)而且更不容易出错，只需要录入数据然后点击Execute，如果再配合自动化框架，可以说基本就不需要人为操作了。

Swagger是个优秀的工具，现在国内已经有很多的中小型互联网公司都在使用它，相较于传统的要先出Word接口文档再测试的方式，显然这样也更符合现在的快速迭代开发行情。当然了，提醒下大家在正式环境要记得关闭Swagger，一来出于安全考虑二来也可以节省运行时内存。



### 方式二：使用第三方依赖

```xml
<dependency>
    <groupId>com.spring4all</groupId>
    <artifactId>swagger-spring-boot-starter</artifactId>
    <version>1.9.1.RELEASE</version>
</dependency>
```

[github上有配置详情](https://github.com/SpringForAll/spring-boot-starter-swagger)

## 拓展：其他皮肤

我们可以导入不同的包实现不同的皮肤定义：

- bootstrap-ui  **访问 http://localhost:8080/doc.html**


```
<dependency>
   <groupId>com.github.xiaoymin</groupId>
   <artifactId>swagger-bootstrap-ui</artifactId>
   <version>1.9.1</version>
</dependency>
```

![image-20201016210544771](image-20201016210544771.png)

- Layui-ui  **访问 http://localhost:8080/docs.html**

```
<dependency>
   <groupId>com.github.caspar-chen</groupId>
   <artifactId>swagger-ui-layer</artifactId>
   <version>1.1.3</version>
</dependency>
```

注：需注入一个groupName为默认的Docket

![image-20201016210310198](image-20201016210310198.png)

- mg-ui  **访问 http://localhost:8080/document.html**

```
<dependency>
   <groupId>com.zyplayer</groupId>
   <artifactId>swagger-mg-ui</artifactId>
   <version>1.0.6</version>
</dependency>
```

![image-20201016202725022](image-20201016202725022.png)