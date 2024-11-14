# Spring介绍

## Spring是一个IOC（DI）和AOP框架
###  Sprng的优良特性
  ·非侵入式：基于Spring开发的应用中的对象可以不依赖于Spring的API
  ·依赖注入：DI是控制反转（IOC）最经典的实现
  ·面向切面编程：AOP
  ·组件化：Spring通过众多简单的组件配置组合成一个复杂应用
  ·一站化：Spring提供了一系列框架，解决了应用开发中的众多问题

## Spring模块划分
![image](https://github.com/user-attachments/assets/a427b517-8fca-4204-a3b7-cf7e87c29937)



![image](https://github.com/user-attachments/assets/455d1c44-9933-4e6f-a011-fcae8305ffc4)

# Spring-IOC容器

## 组件和容器
  ·组件：具有一定功能的对象。
  ·容器：管理组件（创建，获取，保存，销毁）

  可以将组件和容器的关系比喻成**“房间”与“家具”**的关系：

  组件：可以看作是各种“家具”，比如桌子、椅子、灯、书架等。它们是构成界面或应用功能的基本元素，完成特定的功能任务，比如显示文本、输入数据等。
  容器：则是“房间”或“空间”，用于装下各种家具。容器负责管理组件的布局、位置和相互之间的关系，同时也可能控制组件的生命周期和事件传递。

 常见的容器：**Servlet 容器**（如 Tomcat、Jetty）：用于管理和运行 Java Web 应用，处理 HTTP 请求和响应。
​                      **Docker 容器**：用于封装应用及其依赖的环境，确保应用能够在不同平台上运行一致。

![image](https://github.com/user-attachments/assets/8387f0b9-cd20-44c9-84ad-7ce77f8208ee)
## IOC和DI
### **IOC：Inversion of Control（控制反转）**
**控制**：资源的控制权（资源的创建、获取、销毁等）
**反转**：传统上，对象的创建和依赖的管理是由对象本身控制的，而在控制反转的设计中，这种控制权被“反转”到了外部容器或框架中

### **DI：Dependency Injection（依赖注入）**
**依赖**：组件的依赖关系，如 NewsController 依赖 NewsServices
**注入**：通过setter方法、构造器、等方式自动的注入（赋值）

**简单示例**
假设我们有一个 UserService 类，它依赖于 UserRepository：

```java
//无控制反转
public class Userservice{
   private UserReposity userRepository;

   public UserService(){
       this.userRepository=new UserRepository();//自行创建依赖
    }
}
````
在 IoC 的情况下，UserService 不再自行创建 UserRepository，而是通过依赖注入的方式，由外部容器将 UserRepository 注入到 UserService 中：
```java
//使用控制反转
public class Userservice{
 private UserRepository userRepository;

    // 通过构造函数注入
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}

public class UserService {
    @Autowired
    private UserRepository userRepository;  // 直接在属性上注入
}

public class UserService {
    private UserRepository userRepository;

    @Autowired
    public void setUserRepository(UserRepository userRepository) {  // 通过setter方法注入
        this.userRepository = userRepository;
    }
}
```

## 注册组建的各种方式

```java
	 public static void main(String[] args) {
        //跑起spring应用 返回的是一个ioc容器
        ConfigurableApplicationContext ioc = SpringApplication.run(DemoApplication.class, args);
        System.out.println("ioc="+ioc);
        }
```

### **1.通过@Bean**
多应用在方法上
Spring 4后推荐我们使用Java Config的方式来注册组件。**@Configuration**是 Spring 框架中的一个注解，用于标记一个类为 配置类，相当于 Spring XML 配置文件的替代方式。被 **@Configuration** 标记的类可以用来定义 Bean，并将它们注册到 Spring 的 IoC 容器中。
告诉 Spring 该类包含了 一个或多个 @Bean 方法，这些方法会生成所需的 Bean，并注册到容器中以便在整个应用中共享

```java
@Configuration
public class PersonConfig{
  @Bean
  public Person person(){
     return new Person("zhang",20);
  }
}
```

### **2.通过@Component**
多应用在类上
MVC分层多用@Controller，@Service，@Component

```java
//@Service源代码
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component   //一样
public @interface Service {
    @AliasFor(
        annotation = Component.class
    )
    String value() default "";
}
```

```java
@Service
public class UserServiceImpl implements Userservice{
  
}
```
### **3.使用@ComponentScan扫描**
```java
@Configuration
@ComponentScan(value = {"com.example.springdemo.controller","com.example.springdemo.entity","com.example.springdemo.dao","com.example.springdemo.service"})
public class WebConfig {

//    @Bean("myUser")
//    public User user() {
//        return new User("cc", 18);
//    }

}
```
excludeFilters来排除一些组件的扫描：
```java
@Configuration
@ComponentScan(value = {"com.example.springdemo.controller",
        "com.example.springdemo.entity",
        "com.example.springdemo.dao",
        "com.example.springdemo.service"},
        excludeFilters = {
                @ComponentScan.Filter(type = FilterType.ANNOTATION,
                        classes = {RestController.class}),
                @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = User.class)
        })
public class WebConfig {

//    @Bean("myUser")
//    public User user() {
//        return new User("cc", 18);
//    }

}
```
includeFilters的作用和excludeFilters相反，其指定的是哪些组件需要被扫描：
```java
@ComponentScan(value = {"com.example.springdemo.controller",
        "com.example.springdemo.entity",
        "com.example.springdemo.dao",
        "com.example.springdemo.service"},
        includeFilters = {
                @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Service.class)
        }, useDefaultFilters = false)
```

上面配置了只将Service纳入IOC容器，并且需要用useDefaultFilters = false来关闭Spring默认的扫描策略才能让我们的配置生效（Spring Boot入口类的@SpringBootApplication注解包含了一些默认的扫描策略）。

### **4.@Import导入**
可以使用@Import来快速地往IOC容器中添加组件。
创建一个新的类Hello：
```java
public class Hello {
}
```
然后在配置类中导入这个组件：
```java
@Configuration
@Import({Hello.class})
public class WebConfig {
	...
}
```

### **5.组件作用域@Scope**
默认情况下，在Spring的IOC容器中每个组件都是单例的，即无论在任何地方注入多少次，这些对象都是同一个
在Spring中我们可以使用@Scope注解来改变组件的作用域：
```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Scope {
    @AliasFor("scopeName")
    String value() default "";

    @AliasFor("value")
    String scopeName() default "";

    ScopedProxyMode proxyMode() default ScopedProxyMode.DEFAULT;
}
```
有如下几个选项：

**singleton**：单实例（默认）,在Spring IOC容器启动的时候会调用方法创建对象然后纳入到IOC容器中，以后每次获取都是直接从IOC容器中获取（map.get()）；容器启动的时候，会创建单实例组件的对象，容器启动完成之前就会创建
**prototype**：多实例，IOC容器启动的时候并不会去创建对象，而是在每次获取的时候才会去调用方法创建对象；容器启动的时候，不会创建非单实例组件的对象，什么时候获取，什么时候创建
**request**：一个请求对应一个实例；
**session**：同一个session对应一个实例。
我们可以使用多实例测试一下，在配置文件中给User添加Scope注解。
```java
@Configuration
public class WebConfig {

        @Bean("myUser")
        @Scope(value = "prototype")
        public User user() {
                return new User("cc", 18);
        }

}
```
### **6.懒加载@Lazy**
容器启动之前不会创建懒加载组件的对象
什么时候获取，什么时候创建
懒加载是针对单例模式而言的，正如前面所说，IOC容器中的组件默认是单例的，容器启动的时候会调用方法创建对象然后纳入到IOC容器中。
@Configuration
public class WebConfig {
```java
    @Bean("myUser")
//        @Scope(value = "prototype")
    @Lazy
    public User user() {
        System.out.println("往IOC容器中注册user bean");
        return new User("cc", 18);
    }

}
```

## 注入组件的各种方式
### **1. 构造器注入**
构造器注入是最常见和推荐的依赖注入方式之一。通过构造器注入，我们可以在创建一个Bean实例时，将其所需的依赖项作为构造函数的参数进行传递。Spring容器会负责解析依赖关系并创建Bean的实例。示例代码如下：
```java
public class ExampleService {
    private Dependency dependency;

    public ExampleService(Dependency dependency) {
        this.dependency = dependency;
    }

    // ...
}
```
### **2. Setter方法注入**
Setter方法注入是另一种常用的依赖注入方式。通过Setter方法注入，我们在Bean的类中定义对应的Setter方法，Spring容器会通过调用这些Setter方法来设置依赖项。示例代码如下：
```java
public class ExampleService {
    private Dependency dependency;

    public void setDependency(Dependency dependency) {
        this.dependency = dependency;
    }

    // ...
}
```

### **3. 接口注入**
除了构造器注入和Setter方法注入，Spring还支持通过接口注入来实现依赖注入。这种方式要求目标Bean实现特定的接口，并通过接口方法来设置依赖项。示例代码如下：
```java
public interface DependencyInjection {
    void setDependency(Dependency dependency);
}

public class ExampleService implements DependencyInjection {
    private Dependency dependency;

    @Override
    public void setDependency(Dependency dependency) { 
        this.dependency = dependency;
    }

    // ...
}
```

### **4.注解注入**
Spring框架提供了多个注解用于依赖注入，简化了配置和代码的编写。常用的注解包括：

**@Autowired**：自动装配依赖项。
**@Qualifier**：在存在多个候选Bean时，指定要注入的具体Bean。
**@Resource**：指定要注入的Bean，并可以通过名称或类型进行查找。
**@Value**：注入简单的值，如基本类型、字符串等。
**@Inject**：与@Autowired类似，用于依赖注入。
示例代码如下：

```java
public class ExampleService {
    @Autowired
    @Qualifier("dependency")
    private Dependency dependency;
    // ...
}
```
------

***@Autowired 和 @Resource 有什么区别?***
**1.来源不同**
@Autowired 和 @Resource 来自不同的“父类”，其中 @Autowired 是 Spring 定义的注解，而 @Resource 是 Java 定义的注解，它来自于 JSR-250（Java 250 规范提案）
**2.依赖查找顺序不同**
依赖注入的功能，是通过先在 Spring IoC 容器中查找对象，再将对象注入引入到当前类中。而查找有分为两种实现：按名称（byName）查找或按类型（byType）查找，其中 @Autowired 和 @Resource 都是既使用了名称查找又使用了类型查找，但二者进行查找的顺序却截然相反。
  2.1 @Autowired 查找顺序
@Autowired 是先根据类型（byType）查找，如果存在多个 Bean 再根据名称（byName）进行查找，它的具体查找流程如下：

![image.png](https://ucc.alicdn.com/464o66bnbw5zc/developer-article1003903/20241026/ee57439399df487b9eb40a0f50387f32.png)

  2.2 @Resource 查找顺序
@Resource 是先根据名称查找，如果（根据名称）查找不到，再根据类型进行查找，它的具体流程如下图所示:

![image.png](https://ucc.alicdn.com/464o66bnbw5zc/developer-article1003903/20241026/594e0365fda84e37867ebfb98ec738e8.png)
  2.3 查找顺序小结
由上面的分析可以得出：
@Autowired 先根据类型（byType）查找，如果存在多个（Bean）再根据名称（byName）进行查找；
@Resource 先根据名称（byName）查找，如果（根据名称）查找不到，再根据类型（byType)进行查找。
**3.支持的参数不同**
@Autowired 和 @Resource 在使用时都可以设置参数，比如给 @Resource 注解设置 name 和 type 参数，实现代码如下：

```java
@Resource(name = "userinfo", type = UserInfo.class)
private UserInfo user;
```
但二者支持的参数以及参数的个数完全不同，其中 @Autowired 只支持设置一个 required 的参数，而 @Resource 支持 7 个参数![image.png](https://ucc.alicdn.com/464o66bnbw5zc/developer-article1003903/20241026/b021209425294ad9845a765b0f16fa9d.png)

![image.png](https://ucc.alicdn.com/464o66bnbw5zc/developer-article1003903/20241026/272dd61a8fa949fa95c6826f2558d3fe.png)
**4.依赖注入的支持不同**
其中， @Autowired 支持属性注入、构造方法注入和 Setter 注入，而 @Resource 只支持属性注入和 Setter 注入
**5.编译器提示不同**
当使用 IDEA 专业版在编写依赖注入的代码时，如果注入的是 Mapper 对象，那么使用 @Autowired 编译器会提示报错信息

## @Bean的生命周期

![image](https://github.com/user-attachments/assets/cf715aae-db48-4238-9e43-9c6d2c98d335)