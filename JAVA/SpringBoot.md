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

![image-20231202151051746](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202312031336284.png)

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

## 静态资源的访问

在static目录下放一张静态图片，访问`localhost:8080/test.jpg`就可以直接访问

## 文件上传

```java
spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=10MB
```

当表单的`enctype="multipart/form-data"`时可以使用`MultipartFile`获取上传的文件数据，再通过`transferTo`方法将其写入到磁盘中

```java
@RequestMapping(value = "/upload", method = RequestMethod.POST)
public String up(String nickname, MultipartFile photo, HttpServletRequest request) throws IOException{
    System.out.println(nickname);
    System.out.println(photo.getOriginalFilename());
    System.out.println(photo.getContentType());

    //获取当前服务器的目录
    //创建一个upload目录
    String path = request.getServletContext().getRealPath("/upload/");
    System.out.println(path);
    saveFile(photo, path);
    return "上传成功";
}

private void saveFile(MultipartFile photo, String path) throws IOException {
    //判断存储的目录是否存在
    File dir = new File(path);
    //如果不存在
    if (!dir.exists())
    {
        //则创建目录
        dir.mkdir();
    }

    File file = new File(path+photo.getOriginalFilename());
    photo.transferTo(file);
}
```

`spring.web.resources.static-locations=/upload/`修改static的默认路径

## 拦截器

1. 权限检查：如登录检测
2. 性能监控：
3. 通用行为：读取cookie得到用户信息并对用户对象放入请求，从而方便后续流程使用

`HandlerInterceptor`接口定义了`preHandle`、`postHandle`、`afterCompletion`三种方法，通过重写这三种方法实现请求前、请求后等操作。

`Ctrl` 点击查看父类接口

request获取前端的数据，response给前端返回数据

```java
public class LoginInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("LoginInterceptor");
        return true;
    }
}
```

配置一下拦截器，定义拦截的地址

如果没有addPathPatterns，默认拦截所有请求

```java
@Configuretion
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginInterceptor()).addPathPatterns("/user/**");
    }
}
```

## RESTful

一组客户端和服务端交互时的架构理念和设计原则

* 客户端使用GET、POST、PUT、DELETE四种表示操作方式的动词对服务端资源进行操作：GET获取资源，POST新建资源，PUT更新资源，DELETE删除资源
* 资源的表现形式是JSON或HTML

| HTTP方法 | 操作   | 返回值                                 | 特定返回值                                 |
| -------- | ------ | -------------------------------------- | ------------------------------------------ |
| POST     | Create | 201（Create）或200（OK）提交或保存资源 | 404（NotFound），409（资源已经存在）       |
| GET      | Read   | 200（OK）                              | 404（NotFound）                            |
| PUT      | Update | 200（OK）或204（NoContent）修改资源    | 404（NotFound），405（禁止使用改方法调用） |
| PATCH    | Update | 200（OK）或204（NoContent）部分修改    | 404（NotFound）                            |
| DELETE   | Delete | 200（OK）                              | 404（NotFound），405（禁止使用改方法调用） |

![image-20231202163013052](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202312031337612.png)



## 状态码

1xx：信息，通信传输协议

2xx：成功

3xx：重定向，客户端必须执行一些其他操作才能完成器请求

4xx：客户端错误

5xx：服务器错误
