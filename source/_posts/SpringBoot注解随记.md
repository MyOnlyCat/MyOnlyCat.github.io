---
title: SpringBoot注解随记
categories: SpringBoot
tags:
  - 'SpringBoot,'
  - 注解
abbrlink: 17a32c5f
date: 2017-12-15 20:14:52
---


**记录一些工作中遇到的注解方便以后查找,2017年12月19日**
<!--more-->

### 前言
**记录一些工作中遇到的注解方便以后查找,2017年12月19日**


### @RestController

- `@RestController`
4.0之前的版本，SpringMVC的组件都使用`@Controller`来标识当前类是一个控制器servlet,现在`@RestController` 就相当于标识当前类为**Controller**,支持返回xml和json


### @RequestMapping

- `@RequestMapping(value="/users")`
在**Controller**层上配置
```java
@RestController
@RequestMapping(value = "/user")  // 通过这里配置使下面的映射都在/user下
public class UserController {
}
```
- @RequestMapping(value="/", method=RequestMethod.GET)
这里指定了访问的类型`GET`,相似的还有

| Method                | 请求类型 |
| ----------------------| ---------|
| method=RequestMethod.GET | GET请求 |
| method=RequestMethod.POST | POST请求 |
| method=RequestMethod.PUT | PUT请求 |
| method=RequestMethod.DELETE | DELETE请求|

可以用这个方式实现**RESTful API**

**RESTful API设计思想大概:**

| 请求类型 | URL | 功能说明 |
| -------- | ----| -------- |
| GET | /users | 查询用户列表 |
| POST | /users | 创建一个用户 |
| GET | /users?id= | 根据ID查询一个用户|
| PUT | /users?id= | 根据ID更新一个用户 |
| DELETE | /users?id= | 根据ID删除一个用户 |

示例：
```java
 
@RestController 
@RequestMapping(value="/users")     // 通过这里配置使下面的映射都在/users下 
public class UserController { 
 
 
    @RequestMapping(value="/", method=RequestMethod.GET) 
    public List<User> getUserList() { 
        // 处理"/users/"的GET请求，用来获取用户列表 
        // 还可以通过@RequestParam从页面中传递参数来进行查询条件或者翻页信息的传递 
        List<User> r = new ArrayList<User>(users.values()); 
        return r; 
    } 
 
    @RequestMapping(value="/", method=RequestMethod.POST) 
    public String postUser(@ModelAttribute User user) { 
        // 处理"/users/"的POST请求，用来创建User 
        // 除了@ModelAttribute绑定参数之外，还可以通过@RequestParam从页面中传递参数 
        users.put(user.getId(), user); 
        return "success"; 
    } 
 
    @RequestMapping(value="/{id}", method=RequestMethod.GET) 
    public User getUser(@PathVariable Long id) { 
        // 处理"/users/{id}"的GET请求，用来获取url中id值的User信息 
        // url中的id可通过@PathVariable绑定到函数的参数中 
        return users.get(id); 
    } 
 
    @RequestMapping(value="/{id}", method=RequestMethod.PUT) 
    public String putUser(@PathVariable Long id, @ModelAttribute User user) { 
        // 处理"/users/{id}"的PUT请求，用来更新User信息 
        User u = users.get(id); 
        u.setName(user.getName()); 
        u.setAge(user.getAge()); 
        users.put(id, u); 
        return "success"; 
    } 
 
    @RequestMapping(value="/{id}", method=RequestMethod.DELETE) 
    public String deleteUser(@PathVariable Long id) { 
        // 处理"/users/{id}"的DELETE请求，用来删除User 
        users.remove(id); 
        return "success"; 
    } 
 
}
```

### @PathVariable

- `@PathVariable`
作用:映射 URL 绑定的占位符
**示例:**
```java
//@PathVariable可以用来映射URL中的占位符到目标方法的参数中
@RequestMapping("/testPathVariable/{id}")
    public String testPathVariable(@PathVariable("id") Integer id)
    {
        System.out.println("testPathVariable:"+id);
        return SUCCESS;
    }
```

### @ModelAttribute

- @ModelAttribute
```java
 @RequestMapping(value="/", method=RequestMethod.POST) 
    public String postUser(@ModelAttribute User user) { 
        // 处理POST请求，用来创建User
    } 
```
### @RequestParam

- `@RequestParam`
一种是`request.getParameter("name")`，另外一种是用注解`@RequestParam`直接获取
**示例**
```java
@RequestMapping("testRequestParam")
   public String filesUpload(@RequestParam String inputStr, HttpServletRequest request) { 
    System.out.println(inputStr); 
      
    int inputInt = Integer.valueOf(request.getParameter("inputInt")); 
    System.out.println(inputInt); 
      
    // ......省略 
    return "index"; 
   } 
```

### @ControllerAdvice
- `@ControllerAdvice`
作用于 整个 Spring 工程. ControllerAdvice 注解定义了一个全局的异常处理器,详情在**异常处理**的那一篇博客里面
`@ControllerAdvice`呢也有个相似的`@RestControllerAdvice`

### @ExceptionHandler
捕获异常,以及异常的子类,详细用法在在**异常处理**的那一篇博客里面

### @RunWith
- `@RunWith`
使用RunWith注解改变JUnit的默认执行类，并实现自已的Listener在平时的单元测试，如果不使用RunWith注解，那么JUnit将会采用默认的执行类Suite执行，如下类：
```java
public class SimpleJunitTest {
    
    @Test
    public void testSayHi() {
        System.out.println("Hi Junit.");
    }
    
}
```
- `@RunWith(SpringJUnit4ClassRunner.class)`，这里就指定的是SpringJUnit4ClassRunner.class,如下类:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = MockServletContext.class)
public class HttpTest {

    private MockMvc mvc;

    @Before
    public void setUp() throws Exception {
        //绑定需要测试的Controller到MockMvc上
        mvc = MockMvcBuilders.standaloneSetup(new HelloWordTest()).build();
    }

    @Test
    public void getHello() throws Exception {
        //发出请求，在请求中可以设置一个http request可设置的所有参数
        mvc.perform(MockMvcRequestBuilders.get("/basic/hello").accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(content().string(equalTo("hello")));
    }
} 
```

### @SpringApplicationConfiguration
- `@SpringApplicationConfiguration`
**废弃**

### @SpringBootTest
- `@SpringBootTest`
`@SpringBootTest`替代了`@SpringApplicationConfiguration`
具体用法规定启动容器?如下类:
```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = DemoApplication.class)
public class RedisDemo {
    @Test
    public void test(){
        //测试方法
    }

    @After
    public void testGet(){
       //测试方法
    }

}
```

### @DataJpaTest
- `@DataJpaTest`
目前我已知的可用在测试JPA中(毕竟我还是个菜鸟)
`@DataJpaTest`注解它只扫描`@Entity` 和装配 Spring Data JPA 存储库，其他常规的`@Component`（包括`@Service`、`@Repository`等）Bean 则不会被加载到 Spring 测试环境上下文。如下类:
```java
@RunWith(SpringRunner.class)
@DataJpaTest
public class UserRepositoryInMemoryTest {
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    public void testSave() {
        User user = new User();
        user.setName("fanlychie");
        userRepository.save(user);
        System.out.println("====================================");
        System.out.println(userRepository.findAll());
        System.out.println("====================================");
    }
    
}
```

### @Entity,@Table,@Id,@GeneratedValue,@Column
- `@Entity`,`@Entity`,`@Table`,`@Id`,`@GeneratedValue`,`@Column`
- 这里把这几个注解一起记录,因为一般都是在一起用的
具体使用的方法如下类(**结合JPA使用**):
```java
@Entity //表明为实体类
@Table(name="user") //指定对应的表
public class SystemUser implements Serializable{
	@Id //表明主键
	@GeneratedValue(strategy = GenerationType.IDENTITY) //表明id为自增
	@Column(name = "id") //在表中对应的字段
	private Integer id;
	
	@Column(name="user_name")
	private String name;
	
	@Column(name="user_password")
	private String password;
	
	//....省略Geter,Seter
}
```

### @Mapper,@Results,@Result,@Select,@Update,@Delete,@Insert,@Options
- `@Mapper`将UserDao声明为一个Mapper接口
- `@Results` 字段与数据库的映射列表
- `@Result` 进行详细的映射,其中property是User类的属性名，colomn是数据库表的字段名 
- `@Select` 写入查询
- `@Update` 写入更新语句
- `@Delete` 写入删除语句
- `@Insert` 写入插入语句
- `@Options` 设置主键,其中useGeneratedKeys是使用主键,keyProperty实体类中主键的名字,keyColumn数据

**详细的用法可以参照....**

### @GetMapping,@PostMapping,@PutMapping,@DeleteMapping,@PatchMapping

- 这些注解是spring4.3引进的,自己感觉更便于开发RESTful风格的接口
- 拿  以@GetMapping为例，Spring官方文档说：
`@GetMapping`是一个组合注解，是`@RequestMapping(method = RequestMethod.GET)`的缩写。该注解将HTTP Get 映射到 特定的处理方法上。
- 其它的同理

### @SpringBootApplication

- `@SpringBootApplication`
- 在springboot的启动类上,具体如下:

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
@SpringBootApplication //开启组件扫描和自动配置
public class ReadingListApplication {
    public static void main(String[] args) {
    SpringApplication.run(ReadingListApplication.class, args); //负责启动引导应用程序
    }
}
```

- `@SpringBootApplication` 开启了Spring的组件扫描和Spring Boot的自动配置功能。实际
上， `@SpringBootApplication` 将三个有用的注解组合在了一起
    -   Spring的 `@Configuration` ：标明该类使用Spring基于Java的配置。
    -   Spring的 `@ComponentScan` ：启用组件扫描，这样你写的Web控制器类和其他组件才能被
自动发现并注册为Spring应用程序上下文里的Bean
    - Spring Boot 的 `@EnableAutoConfiguration` ： 这 个 不 起 眼 的 小 注 解 也 可 以 称 为
`@Abracadabra`

### @SpringApplicationConfiguration

- @SpringApplicationConfiguration

```jav
@SpringApplicationConfiguration(classes = ReadingListApplication.class)
```
通过 Spring Boot加载上下文


