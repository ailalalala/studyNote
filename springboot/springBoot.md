# springBoot

## 1.了解springBoot

springBoot就是一个基于spring但是简化了spring各种复杂配置的一个框架。简化开发，约定大于配置。

### 1.1、搭建springBoot环境

1.新建一个maven项目，再pom.xml文件中添加依赖

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.4</version>
    <relativePath/>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

### 1.2、创建运行主程序

```java
@SpringBootApplication
public class HelloWorld {
    public static void main(String[] args) {
        SpringApplication.run(HelloWorld.class);
    }
}
```

然后直接启动main方法就ok。相当简单了。

### 1.3、创建controller

```java
@RestController
@RequestMapping("/hello")
public class HelloController {
    @RequestMapping("/hello")
    public String hello(){
        return "hello";
    }
}
```

运行主程序访问localhost:8080/hello/hello可以读取hello字符串。

### 1.4、打包

再pom.xml文件中添加打包插件

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <fork>true</fork>
            </configuration>
        </plugin>
    </plugins>
</build>
```

运行 mvn clean package就可以生成可运行jar包。运行java -jar jar包名。就可以直接运行与访问了

## 2.了解自动装配原理

### 2.1、springboot特点

1.依赖管理

- 父项目做依赖管理

  ```xml
  <parent>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-parent</artifactId>
      <version>2.4.4</version>
      <relativePath/>
  </parent>
  他的父项目
  <parent>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-dependencies</artifactId>
      <version>2.4.4</version>
  </parent>
  该父项目几乎声明了所有开发中常用的依赖版本号，自动版本仲裁。该父项目中使用的是dependencyManagement来管理依赖包，只声明依赖不实现导入。
  ```

- 开发导入starter场景启动器

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  
  spring-boot-starter-*，*就代表了某种场景，只要引入了starter，这个场景所有的常会需要的依赖就会自动导入。
  springBoot支持的所有场景：
  https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-starter
  支持第三方提供的场景启动器，命名规则：*-spring-boot-starter。
  所有场景启动最底层的依赖
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <version>2.4.4</version>
    <scope>compile</scope>
  </dependency>
  ```

- 无需关注版本号，自动版本仲裁，可以修改版本号。

  ```xml
  如果引入了某个springboot已经管理的依赖可以不写版本号。如果要修改版本号首先要看spring-boot-dependencies里规定当前依赖版本号的命名。
  然后再当前项目的pom.xml中进行修改.例如修改mysql的依赖版本
  <properties>
      <mysql.version>5.1.43</mysql.version>
  </properties>
  ```

2.自动装配

查看当前环境容器中的依赖

```java
public static void main(String[] args) {
    //1.返回我们的IOC容器
    ConfigurableApplicationContext run = SpringApplication.run(MainApplication.class);
    //2.查看当前容器里的组件
    String[] beanDefinitionNames = run.getBeanDefinitionNames();
    for (String beanDefinitionName : beanDefinitionNames) {
        System.out.println(beanDefinitionName);
    }
}
```

springBoot将所有我们需要配置的东西自动配置进来，tomcat、springMVC、web常见功能。==原理后面会细研究。==

**默认的包结构**：主程序所在的包以及下面的子包都会被扫描。

如果修改包扫描路径：

```java
@SpringBootApplication(scanBasePackages = "com.xin")
```

也可以：

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentSca("com.xin")
这三个注解就相当于一个@SpringBootApplication(scanBasePackages = "com.xin")注解
```

**各种配置的默认值：**

默认配置最终都是要映射到某个类上，这个类会在容器中创建对象

**按需加载所有自动配置项：**

引入了哪些场景，这个场景才会自动配置。springBoot所有的自动配置功能都是在

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-autoconfigure</artifactId>
  <version>2.4.4</version>
  <scope>compile</scope>
</dependency>
```

它管理了所有依赖的自动导入。==后面会讲到按照条件进行装配==

### 2.2、容器功能

#### 2.2.1、组件添加

使用注解进行bean的装配。（该内容为spring的知识点，请见spring笔记中的使用注解开发）

```java
public class Person {
    private String name;
    private Pet pet;

    public Person(String name) {
        this.name = name;
    }
    public Person() {
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public Pet getPet() {
        return pet;
    }
    public void setPet(Pet pet) {
        this.pet = pet;
    }
    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", pet=" + pet +
                '}';
    }
}
```

```java
public class Pet {
    private String name;

    public Pet() {
    }
    public Pet(String name) {
        this.name = name;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    @Override
    public String toString() {
        return "Pet{" +
                "name='" + name + '\'' +
                '}';
    }
}
```

```java
@Configuration
public class MyConfig {
    @Bean
    public Person user01(){
        return new Person("aixin");
    }
    @Bean("tom01")
    public Pet tomcat(){
        return new Pet("tomcat");
    }
}
```

在主程序导出容器中的组件名称中就可以看到user01与tom01了。

```java
public static void main(String[] args) {
    //1.返回我们的IOC容器
    ConfigurableApplicationContext run = SpringApplication.run(MainApplication.class);
    MyConfig myConfig = run.getBean("myConfig", MyConfig.class);
    System.out.println(myConfig);
}
输出结果：
com.xin.config.MyConfig$$EnhancerBySpringCGLIB$$cffbd24a@781a9412
```

可以看到Myconfig是一个被spring增强了的代理对象。因为@Configuration中的proxyBeanMethods默认为true代表了该配置类为一个代理对象。springboot就总会检查容器中是否有user01与tom01等组件，如果有就直接使用。就默认都是单例的。

```java
public static void main(String[] args) {
    //1.返回我们的IOC容器
    ConfigurableApplicationContext run = SpringApplication.run(MainApplication.class);
    MyConfig myConfig = run.getBean("myConfig", MyConfig.class);
    System.out.println(myConfig);
    Person user01 = run.getBean("user01", Person.class);
    Person user02 = run.getBean("user01", Person.class);
    System.out.println(user01 == user02);
    Person user03 = myConfig.user01();
    Person user04 = myConfig.user01();
    System.out.println(user03 == user04);
}
结果都为true
```

无论是直接从容器中去获取还是直接通过myconfig的方法调用，两个对象都是一样的。就说明只要调用了这个方法springBoot就会去容器中查找是否有该组件，如果有就使用这个组件。

如果改为false：

```java
@Configuration(proxyBeanMethods = false)//默认不写为true
public class MyConfig {
    @Bean
    public Person user01(){
        return new Person("aixin");
    }
    @Bean("tom01")
    public Pet tomcat(){
        return new Pet("tomcat");
    }
}
```

```java
public static void main(String[] args) {
    //1.返回我们的IOC容器
    ConfigurableApplicationContext run = SpringApplication.run(MainApplication.class);
    MyConfig myConfig = run.getBean("myConfig", MyConfig.class);
    System.out.println(myConfig);
    Person user01 = run.getBean("user01", Person.class);
    Person user02 = run.getBean("user01", Person.class);
    System.out.println(user01 == user02);
    Person user03 = myConfig.user01();
    Person user04 = myConfig.user01();
    System.out.println(user03 == user04);
}
结果都为
com.xin.config.MyConfig@6b7d1df8
true
false
```



总结：

配置的两个模式

- Full（全模式）：@Configuration(proxyBeanMethods = true)或默认不写
- Lite（轻量级模式）：@Configuration(proxyBeanMethods = false)。该模式优点，springBoot就不会去检查配置方法中的返回值是否在容器中存在组件，启动模式快，加载起来也会快。

#### 2.2.2、@Import

@Import给容器中自动创建出该类型的组件，默认组件的名字就是全类名

```java
@Import({Person.class})
@Configuration(proxyBeanMethods = true)
public class MyConfig {
    @Bean
    public Person user01(){
        return new Person("aixin");
    }
    @Bean("tom01")
    public Pet tomcat(){
        return new Pet("tomcat");
    }
}
```

**注意：如果使用import注解去导入进行创建组件，那么person类必须有无参构造器，否则启动报错。**

```java
在主程序中：
String[] beanNamesForType = run.getBeanNamesForType(Person.class);
for (String s : beanNamesForType) {
	System.out.println(s);
}
结果：
com.xin.beans.Person
user01
```

#### 2.2.3、@Condition

条件装配：满足condition指定的条件才会进行组件注入。

![image-20210411225518995](springBoot.assets/image-20210411225518995.png)

例子：

```java
@Configuration(proxyBeanMethods = false)
public class MyConfig {

    public Person user01(){
        return new Person("aixin");
    }
    @Bean("tom01")
    @ConditionalOnBean(name="user01")
    public Pet tomcat(){
        return new Pet("tomcat");
    }
}
```

测试：

```java
boolean user01 = run.containsBean("user01");
System.out.println(user01);
boolean tom01 = run.containsBean("tom01");
System.out.println(tom01);
结果：都为false
```

**注意：请注意容器注入装配顺序问题，有的时候放在上面可能下面的装配还没有进行所以条件判断才触发。**

```java
@Bean
@ConditionalOnBean(name="tom01")
public Person user01(){
    return new Person("aixin");
}

@Bean("tom01")
public Pet tomcat(){
    return new Pet("tomcat");
}
```

比如上述代码其实已经写明了要注入tom01的组件，但是因为顺序问题所以在@ConditionalOnBean(name="tom01")判断的时候容器还没有将tom01组件装配到容器中所以判断触发。请多注意这个问题。

#### 2.2.4、@ImportResource

可能存在项目中或第三方的spring的xml配置文件，可以使用该注解进行导入装配。

```java
@ImportResource("classpath:beans.xml")
```

在Myconfig类的上面加上这行代码就会将beans.xml中的内容注入到容器中。

```java
boolean haha = run.containsBean("haha");
System.out.println(haha);

结果为true
```

#### 2.2.5、配置绑定

将properties配置文件里的内容与javabean中的字段进行绑定

```properties
mycar.brand=audi
mycar.price=200000
```

方法1：@Component+@ConfigurationProperties(prefix = "mycar")

```java
@Component
@ConfigurationProperties(prefix = "mycar")
public class Car {
    private String brand;
    private Integer price;

    public String getBrand() {
        return brand;
    }
    public void setBrand(String brand) {
        this.brand = brand;
    }
    public Integer getPrice() {
        return price;
    }
    public void setPrice(Integer price) {
        this.price = price;
    }
}
```

**注意：只有在容器中的组件才会拥有springBoot提供的功能。**

测试：

```java
@RestController
public class HelloController {

    @Autowired
    private Car car;

    @RequestMapping("/car")
    public Car mycar(){
        return car;
    }
}
```

访问成功，可以显示出正确的内容。

方法二：@EnableConfigurationProperties(Car.class)+@ConfigurationProperties(prefix = "mycar")

```java
@ConfigurationProperties(prefix = "mycar")
public class Car {
    private String brand;
    private Integer price;

    public String getBrand() {
        return brand;
    }
    public void setBrand(String brand) {
        this.brand = brand;
    }
    public Integer getPrice() {
        return price;
    }
    public void setPrice(Integer price) {
        this.price = price;
    }
}
```

在配置文件中添加注解：

```java
@EnableConfigurationProperties(Car.class)
public class MyConfig
```

@EnableConfigurationProperties(Car.class)该注解实现了

1.开启Car的配置绑定功能。

2.把这个Car这个组件自动注册到容器中

测试成功。

经过我的测试：感觉只要在类中加入@ConfigurationProperties(prefix = "mycar")，然后将Car注入到容器中就会生效。使用@Import({Car.class})也可以。或者直接在MyConfig类中：

```java
@Bean
public Car car(){
    return new Car();
}
```

也可以生效。