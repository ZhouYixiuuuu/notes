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

![image-20231209172631996](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202312091726374.png)

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

```java
@RequestMapping(value = "/user/{id}", method = RequestMethod.GET)
public String getUserById(@PathVariable int id){
    return "根据ID获取用户";
}
```

## Swagger 获取接口文档

http://localhost:8080/swagger-ui/index.html#/

```
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.1.0</version>
</dependency>
```

```java
@Configuration
public class SpringDocConfig {
    @Bean
    public OpenAPI apiInfo() {
        return new OpenAPI().info(new Info()
                        .title("后端接口文档")
                        .version("1.0.0")
                );
    }

    @Bean
    public GroupedOpenApi httpApi() {
        return GroupedOpenApi.builder()
                .group("http")
                .pathsToMatch("/**")
                .build();
    }
}
```

`@Operation(summary = "输入用户id获取用户信息", description = "输入用户id获取用户信息")`

## MybatisPlus

![image-20231204145601412](C:/Users/z1382/AppData/Roaming/Typora/typora-user-images/image-20231204145601412.png)

ORM：Java对象和数据库表映射

在建项目的时候，勾选`mybatis frame`和`mysql driver`

[Maven仓库地址]: https://mvnrepository.com/
[Mybatis-Plus官网]: https://baomidou.com/pages/24112f/

在pom.xml文件中添加依赖：

- Spring Boot Starter JDBC
- Project Lombok
- MySQL Connector/J
- mybatis-plus-boot-starter
- mybatis-plus-generator

配置数据库

```properties
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/user
//
spring.datasource.url=jdbc:mysql://localhost:3306/user?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf-8
spring.datasource.username=root
spring.datasource.password=12345678
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
mybatis-plus.configuration.logImpl=org.apache.ibatis.logging.stdout.StdOutImpl
mybatis-plus.configuration.mapUnderscoreToCamelCase=true 
```

```java
//Mapper
@Mapper
public interface UserMapper extends BaseMapper<User> {
}
```

```java
//Controller
@RestController
public class UserController {
    @Autowired
    private UserMapper userMapper;

    @GetMapping("/user")
    public List<User> query(){
        List<User> list = userMapper.selectList(null);
        return list;
    }

    @PostMapping("/user")
    public String insert(User user) {
        int res = userMapper.insert(user);
        if (res > 0) return "插入成功";
        else return "插入失败";
    }
}
```

```java
//entity
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    private Integer id;
    private String nickname;
    private String password;
}
```



可以查看官网

里面提供了一些注解，可以看看

例如 提供表名`@TableName`

```
@TableId(type = IdType.AUTO)
private int id;

举个例子
@PostMapping("/user")
public String insert(User user) {
    int res = userMapper.insert(user);
    if (res > 0) return "插入成功";
    else return "插入失败";
}

加上上面那个注解以后
除了数据库的id会自增，user的id也会变化，特别是如果插入数据库之后还需要用user做些其他操作的话。
```

```java
//多表查询
@Select("select * from usertable")
    @Results(
            {
                    @Result(column = "id", property = "id"),
                    @Result(column = "nickname", property = "nickname"),
                    @Result(column = "password", property = "password"),
                    @Result(column = "id", property = "orders",javaType = List.class,
                    many = @Many(select = "com.example.sqltest.mapper.OrderMapper.selectOrderByUid"))
            }
    )
    List<User> selectAllUsersAndOrders();
```

条件查询可以使用`QueryWrapper`在官网查询使用

分页查询 

1. 配置一下

```java
@Configuration
public class MyBatisPlusConfig {
    @Bean
    MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return interceptor;
    }
}
```

2. 进行查询

```java
@GetMapping("/user/findByPage")
public IPage findByPage() {
    Page<User> page = new Page<>(0, 2);
    IPage ipage = userMapper.selectPage(page, null);
    return ipage;
}
```

## JWT验证

![image-20231211113713169](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202312111137095.png)

![image-20231212091128562](https://raw.githubusercontent.com/ZhouYixiuuuu/picture/master/imgs/202312120911284.png)

添加依赖

- spring-boot-starter-security
- jjwt-api
- jjwt-impl
- jjwt-jackson

工具类

`utils.JwtUtil`类

```java
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.JwtBuilder;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import org.springframework.stereotype.Component;

import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;
import java.util.Base64;
import java.util.Date;
import java.util.UUID;

@Component
public class JwtUtil {
    public static final long JWT_TTL = 60 * 60 * 1000L * 24 * 14;  // 有效期14天
    public static final String JWT_KEY = "SDFGjhdsfalshdfHFdsjkdsfds121232131afasdfac";

    public static String getUUID() {
        return UUID.randomUUID().toString().replaceAll("-", "");
    }

    public static String createJWT(String subject) {
        JwtBuilder builder = getJwtBuilder(subject, null, getUUID());
        return builder.compact();
    }

    private static JwtBuilder getJwtBuilder(String subject, Long ttlMillis, String uuid) {
        SignatureAlgorithm signatureAlgorithm = SignatureAlgorithm.HS256;
        SecretKey secretKey = generalKey();
        long nowMillis = System.currentTimeMillis();
        Date now = new Date(nowMillis);
        if (ttlMillis == null) {
            ttlMillis = JwtUtil.JWT_TTL;
        }

        long expMillis = nowMillis + ttlMillis;
        Date expDate = new Date(expMillis);
        return Jwts.builder()
                .setId(uuid)
                .setSubject(subject)
                .setIssuer("sg")
                .setIssuedAt(now)
                .signWith(signatureAlgorithm, secretKey)
                .setExpiration(expDate);
    }

    public static SecretKey generalKey() {
        byte[] encodeKey = Base64.getDecoder().decode(JwtUtil.JWT_KEY);
        return new SecretKeySpec(encodeKey, 0, encodeKey.length, "HmacSHA256");
    }

    public static Claims parseJWT(String jwt) throws Exception {
        SecretKey secretKey = generalKey();
        return Jwts.parserBuilder()
                .setSigningKey(secretKey)
                .build()
                .parseClaimsJws(jwt)
                .getBody();
    }
}
```



## 部署到服务器

1. 组一台服务器

2. 设置安全组，把端口开了，包括后端项目的端口/http/https

3.  `git bash` 设置ssh远程登录服务器，将ip放在`.ssh/config`里面

   通过`ssh-copy-id myserver ` 设置免密登录

1. 安装mysql：`sudo apt-get install mysql-server`

   启动mysql：`sudo service mysql start`

   进入mysql：`sudo mysql -u root`

   设置mysql密码：`ALTER USER 'root'@'%' IDENTIFIED WITH caching_sha2_password BY '12345678';`

2. 安装jdk：`sudo apt-get install openjdk-17-jdk`

3. 打包后端

   * 在`pom.xml`中添加配置

     ```
     <build>
         <plugins>
             <plugin>
                 <groupId>org.springframework.boot</groupId>
                 <artifactId>spring-boot-maven-plugin</artifactId>
                 <!--这里写上main方法所在类的路径-->
                 <configuration>
                     <mainClass>com.kob.backend.BackendApplication</mainClass>
                 </configuration>
                 <executions>
                     <execution>
                         <goals>
                             <goal>repackage</goal>
                         </goals>
                     </execution>
                 </executions>
             </plugin>
         </plugins>
     </build>
     ```

   * 添加多一句 `<packaging>jar</packaging>`

4. 在maven里面，点开项目，点击`clean`，点击`package`，在target目录下找到jar包，放到服务器里面去 `scp xxxxx.jar Ali-server:`

5. `java -jar xxxxx.jar`运行后端项目 记得使用tmux

## 状态码

1xx：信息，通信传输协议

2xx：成功

3xx：重定向，客户端必须执行一些其他操作才能完成器请求

4xx：客户端错误

5xx：服务器错误



select * from article where authorID in (select followedId from follow where userId = #{userId})
