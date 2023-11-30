# SpringBoot

## 热部署

```
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-devtools</artifactId>
<version>3.1.2</version>
</dependency>
```

```
//application.properties
spring.devtools.remote.restart.enabled=true
spring.devtools.restart.additional-paths=src/main/java
```

* Settings -> Build, Execution, Deployment -> Compile -> Build project automatically
* Settings  -> Advanced Settings -> auto-make xxxxxx

## 控制器

* `@Controller` 请求页面和数据
* `@RestController` 请求数据

```java
@Controller
public class HelloController {
    @RequestMapping("/hello")
    public String index(ModelMap map) {
        map.addAttribute("name", "zhangsan");
        return "hello"
    }
}
```

1. 返回了`hello`页面和`name`数据，在前端页面中可以通过`${name}`参数获取后台返回的数据并显示
2. `@Controller` 通常与 `Thymeleaf` 模板引擎一起使用

前后端分离 通常会使用 `@ResController`

默认情况下，`@RestController` 注解会将返回的对象数据转换为JSON格式

## 路由映射

* `@RequestMapping`注解主要负责URL的路由映射，它可以添加在`Controller`或具体的方法上
* `value` 请求URL的路径，支持URL模板、正则表达式
* `method`请求的方法  `@RequestMapping("/hello", method = RequestMethod.GET)`

## 参数传递

* `@RequestParam` 将请求参数绑定到控制器的方法参数上

  ```java
  public String getTest3(@RequetParam("nickname") String name)
  ```

* 也可以封装成一个类，在`entity`包里面新建一个类

* `@RequestBody`接收JSON对象，也是接收类，要一一对应

```java
//http://localhost:8080/hello?nickname=zhangsan
@RequestMapping(value = "/hello", method = RequestMethod.GET)
public String hello(String nickname) {
    return "hello" + nickname;
}
```

