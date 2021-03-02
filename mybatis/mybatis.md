# mybatis

## 1.环境搭建

1. 导入相关依赖

   ```xml
   <!-- https://mvnrepository.com/artifact/junit/junit -->
   <dependency>
     <groupId>junit</groupId>
     <artifactId>junit</artifactId>
     <version>4.12</version>
     <scope>test</scope>
   </dependency>
   
   <!--mysqlq驱动-->
   <dependency>
       <groupId>mysql</groupId>
       <artifactId>mysql-connector-java</artifactId>
       <version>8.0.12</version>
   </dependency>
   
   <!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
   <dependency>
     <groupId>org.mybatis</groupId>
     <artifactId>mybatis</artifactId>
     <version>3.4.6</version>
   </dependency>
   ```

2. 创建mybatis核心配置文件

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE configuration
     PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
     "http://mybatis.org/dtd/mybatis-3-config.dtd">
   <configuration>
     <environments default="development">
       <environment id="development">
         <transactionManager type="JDBC"/>
         <dataSource type="POOLED">
           <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
           <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=utf8"/>
           <property name="username" value="root"/>
           <property name="password" value="root"/>
         </dataSource>
       </environment>
     </environments>
     <mappers>
       <mapper resource="com/xin/dao/UserMapper.xml"/>
     </mappers>
   </configuration>
   ```

   注意点：

   - mysql的jar包版本不同 driver的驱动不同，版本5的为原来的驱动，5以上的为····mysql.cj.jdbc.Diver

3. 创建sqlsession生成工具类与相关类
   sqlsession生成工具类

   ```java
   public class MybatisUtils {
   	private static SqlSessionFactory sqlSessionFactory;
   	 
       static {
           try {
               String resource = "mybatis-config.xml";
               InputStream inputStream = Resources.getResourceAsStream(resource);
               sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
           } catch (IOException e) {
               e.printStackTrace();
           }
       }
    
       //获取SqlSession连接
       public static SqlSession getSession(){
           return sqlSessionFactory.openSession();
       }
   
   }
   ```

4. 创建mapper接口与mapper.xml配置文件

   ```java
   public interface UserMapper {
   
   	List<User> getAllUsers();
   	
   }
   ```

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper
           PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
           
   <mapper namespace="com.xin.dao.UserMapper">
     <select id="getAllUsers" resultType="com.xin.beans.User">
       select * from user;
     </select>
   </mapper>
   ```

5. 测试

   ```java
   public class MyTest {
   	@Test
   	public void selectTest() {
   		SqlSession sqlSession = MybatisUtils.getSession();
   		
   		UserMapper mapper = sqlSession.getMapper(UserMapper.class);
   		
   		List<User> users = mapper.getAllUsers();
   		
   		for (User user : users) {
   			System.out.println(user);
   		}
   		
   		sqlSession.close();
   	}
   }
   ```

注意点：

- 如果接口方法中有多个参数的时候必须使用注解来标明参数

  ```java
  //如果多个参数则必须添加参数注解
  User getUserByParam(@Param("id") int id,@Param("name")String name);
  ```

- 如果修改了项目中依赖包的版本后子项目依赖包的版本还是原来版本，则进入子项目进行 mvn clean。

  

## 2.增删改

增

```java
//增
int create(User user);
```

```xml
<insert id="create" parameterType="com.xin.beans.User">
  insert into user (id,name,pwd) values (#{id},#{name},#{pwd});
</insert>
```

改

```java
//改
int update(User user);
```

```xml
<update id="update" parameterType="com.xin.beans.User">
  update user set name=#{name},pwd=#{pwd} where id = #{id};
</update>
```

删

```java
//删
int deleteById(int id);
```

```xml
<delete id="deleteById">
  delete from user where id = #{id};
</delete>
```

测试代码：

```java
@Test
public void createTest() {

  SqlSession session = MybatisUtils.getSession();

  UserMapper mapper = session.getMapper(UserMapper.class);
  User user = new User();
  user.setId(4);
  user.setName("哈哈");
  user.setPwd("666sss");
  mapper.create(user);
  session.commit();

  session.close();

}

@Test
public void updateTest() {

  SqlSession session = MybatisUtils.getSession();

  UserMapper mapper = session.getMapper(UserMapper.class);
  List<User> users = mapper.getAllUsers();
  User user = new User();
  System.out.println("==============update前================");
  for (User user1 : users) {
    System.out.println(user1);
    if(user1.getId() == 1) {
      user = user1;
    }
  }
  user.setPwd("aaabbb");
  mapper.update(user);
  session.commit();
  users = mapper.getAllUsers();
  System.out.println("==============update后================");
  for (User user1 : users) {
    System.out.println(user1);
  }

  session.close();

}

@Test
public void deleteTest() {
  SqlSession session = MybatisUtils.getSession();

  UserMapper mapper = session.getMapper(UserMapper.class);
  List<User> users = mapper.getAllUsers();
  System.out.println("==============delete前================");
  for (User user1 : users) {
    System.out.println(user1);
  }
  mapper.deleteById(4);
  session.commit();
  users = mapper.getAllUsers();
  System.out.println("==============delete后================");
  for (User user1 : users) {
    System.out.println(user1);
  }

  session.close();
}
```

注意：增删改操作后必须使用commit，否则不生效。

## 3.配置解析

### 3.1、核心配置文件

mybatis-config.xml文件的配置项。MyBatis 的配置文件包含了会深深影响 MyBatis 行为的设置和属性信息。

```xml
configuration（配置）
    properties（属性）
    settings（设置）
    typeAliases（类型别名）
    typeHandlers（类型处理器）
    objectFactory（对象工厂）
    plugins（插件）
    environments（环境配置）
        environment（环境变量）
            transactionManager（事务管理器）
            dataSource（数据源）
    databaseIdProvider（数据库厂商标识）
    mappers（映射器）
<!-- 注意元素节点的顺序！顺序不对会报错 -->
```

主要配置项：

#### 3.1.1、properties

使用外部properties配置文件或者在配置文件中使用properties标签进行赋值，为核心配置文件赋值

第一种：创建mysql.properties配置文件

```properties
driver=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/mybatis?useSSL=true&useUnicode=true&characterEncoding=utf8
username=root
password=root
```

第二种：在mybatis核心配置文件中

```xml
<properties>
  <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
  <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=utf8"/>
  <property name="username" value="root"/>
  <property name="password" value="root"/>
</properties>
```

也可以第一种与第二种混合使用

```xml
<properties resource="org/mybatis/example/config.properties">
  <property name="username" value="dev_user"/>
  <property name="password" value="F2Fa3!33TYyg"/>
</properties>
```

核心配置文件修改

```xml
<properties resource="mysql.properties">
</properties>

<environments default="development">
  <environment id="development">
    <transactionManager type="JDBC"/>
    <dataSource type="POOLED">
      <property name="driver" value="${driver}"/>
      <property name="url" value="${url}"/>
      <property name="username" value="${username}"/>
      <property name="password" value="${password}"/>
    </dataSource>
  </environment>
</environments>
```

注意：核心配置文件里value的值需要使用双引号引上。在properties文件中&不需要使用转义字符&amp;

#### 3.1.2、typeAliases（类型别名）

类型别名是为 Java 类型设置一个短的名字。它只和 XML 配置有关，存在的意义仅在于用来减少类完全

限定名的冗余。

第一种写法：

```xml
<typeAliases>
  <typeAlias type="com.xin.beans.User" alias="user"/>
</typeAliases>
```

当这样配置时，user就可以在任何使用com.xin.beans.User的地方。

第二种写法：扫描包

```xml
<typeAliases>
 	<package name="com.xin.beans"/>
</typeAliases>
```

```java
@Alias("user")
public class User 
```

如果包中的bean上没有添加别名注解，那则以此类名的首字母小写作为别名。

#### 3.1.2、environments元素

```xml
<environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
      </dataSource>
    </environment>
  </environments>
```

- mybatis可以配置多个运行环境，将SQL映射到多个不同的数据库上，必须指定其中一个为默认运行

  环境（通过default指定）

  ![image-20210301174311323](/Users/aixin/Library/Application Support/typora-user-images/image-20210301174311323.png)

- 子元素节点：environment

  - 具体的一套环境，通过id区分，id保证唯一。

  - 子元素节点：transactionManager-[事务管理器]

    ```xml
    <!-- 语法 --> 
    <transactionManager type="[ JDBC | MANAGED ]"/>
    ```

  - 子元素节点：dataSource-数据源

    - dataSource 元素使用标准的 JDBC 数据源接口来配置 JDBC 连接对象的资源。

    - 数据源是必须配置的

    - 有三种内建的数据源类型

      ```xml
      type="[UNPOOLED|POOLED|JNDI]"）
      ```

      UNPOOLED:这个数据源的实现只是每次被请求时打开和关闭连接。

      POOLED: 这种数据源的实现利用“池”的概念将 JDBC 连接对象组织起来 , 这是一种使得

      ​                 并发 Web 应用快速响应请求的流行处理方式。

      JNDI：这个数据源的实现是为了能在如 Spring 或应用服务器这类容器中使用，容器可以

      ​			集中或在外部配置数据源，然后放置一个 JNDI 上下文的引用。

      数据源也有很多第三方的实现，比如dbcp，c3p0，druid等等....

#### 3.1.3、mappers元素







