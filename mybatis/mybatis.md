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

**mappers：**

- 映射器：定义映射sql语句的文件

- 既然 MyBatis 的行为其他元素已经配置完了，我们现在就要定义 SQL 映射语句了。但是首先我们

  需要告诉 MyBatis 到哪里去找到这些语句。 Java 在自动查找这方面没有提供一个很好的方法，所

  以最佳的方式是告诉 MyBatis 到哪里去找映射文件。你可以使用相对于类路径的资源引用， 或完

  全限定资源定位符（包括 file:/// 的 URL），或类名和包名等。映射器是MyBatis中最核心

  的组件之一，在MyBatis 3之前，只支持xml映射器，即：所有的SQL语句都必须在xml文件中配

  置。而从MyBatis 3开始，还支持接口映射器，这种映射器方式允许以Java代码的方式注解定义SQL

  语句，非常简洁。

**引入资源方式：**

```xml
<!-- 使用相对于类路径的资源引用 -->
<mappers>
  <mapper resource="com/xin/dao/UserMapper.xml"/>
</mappers>
```

```xml
<!-- 使用完全限定资源定位符（URL） -->
<mappers>
    <mapper url="file:///Users/aixin/Desktop/tijcode/springStudy/springProject1.0/mybatisTest/src/main/java/com/xin/dao/UserMapper.xml"/>
  </mappers>
```

```xml
<!-- 使用映射器接口实现类的完全限定类名 需要配置文件名称和接口名称一致，并且位于同一目录下 -->
<mappers>
    <mapper class = "com.xin.dao.UserMapper"/>
</mappers>
```

```xml
<!-- 将包内的映射器接口实现全部注册为映射器 但是需要配置文件名称和接口名称一致，并且位于同一目录下 -->
<mappers>
  <package name="com.xin.dao"/>
</mappers>
```

**mapper文件：**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
        
<mapper namespace="com.xin.dao.UserMapper">
  
  <select id="getAllUsers" resultType="user">
    select * from user;
  </select>
  
</mapper>
```

- namespace:命名空间。绑定接口类。
- sql方法标签中的ID要与绑定的接口的方法保持一致。

### 3.2、其他配置

完整的setting配置：

```xml
<settings>
  <setting name="cacheEnabled" value="true"/>
  <setting name="lazyLoadingEnabled" value="true"/>
  <setting name="multipleResultSetsEnabled" value="true"/>
  <setting name="useColumnLabel" value="true"/>
  <setting name="useGeneratedKeys" value="false"/>
  <setting name="autoMappingBehavior" value="PARTIAL"/>
  <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
  <setting name="defaultExecutorType" value="SIMPLE"/>
  <setting name="defaultStatementTimeout" value="25"/>
  <setting name="defaultFetchSize" value="100"/>
  <setting name="safeRowBoundsEnabled" value="false"/>
  <setting name="mapUnderscoreToCamelCase" value="false"/>
  <setting name="localCacheScope" value="SESSION"/>
  <setting name="jdbcTypeForNull" value="OTHER"/>
  <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
</settings>
```

### 3.3、作用域与生命周期

![image-20210302163312890](/Users/aixin/Library/Application Support/typora-user-images/image-20210302163312890.png)

作用域理解：

- SqlSessionFactoryBuilder的作用在于创建SqlSessionFactory，创建成功后SqlSessionFactoryBuilder 就失去了作用，所以它就只应该存在与创建SqlSessionFactory的方法中，而不要长期存在。因此**SqlSessionFactoryBuilder实例的最佳作用域是方法作用域（局部方法变量）。**
- SqlSessionFactory 可以被认为是一个数据库连接池，它的作用是创建 SqlSession 接口对象。因为MyBatis 的本质就是 Java 对数据库的操作，所以 SqlSessionFactory 的生命周期存在于整个MyBatis 的应用之中，所以一旦创建了 SqlSessionFactory，就要长期保存它，直至不再使用MyBatis 应用，所以可以认为SqlSessionFactory 的生命周期就等同于 MyBatis 的应用周期。
- 由于 SqlSessionFactory 是一个对数据库的连接池，所以它占据着数据库的连接资源。如果创建多个 SqlSessionFactory，那么就存在多个数据库连接池，这样不利于对数据库资源的控制，也会导致数据库连接资源被消耗光，出现系统宕机等情况，所以尽量避免发生这样的情况。因此在一般的应用中我们往往希望 SqlSessionFactory 作为一个单例，让它在应用中被共享。所以说 **SqlSessionFactory** **的最佳作用域是应用作用域。**
- 如果说 SqlSessionFactory 相当于数据库连接池，那么 SqlSession 就相当于一个数据库连接（Connection 对象），你可以在一个事务里面执行多条 SQL，然后通过它的 commit、rollback等方法，提交或者回滚事务。所以它应该存活在一个业务请求中，处理完整个请求后，应该关闭这条连接，让它归还给SqlSessionFactory，否则数据库资源就很快被耗费精光，系统就会瘫痪，所以用 try...catch...finally... 语句来保证其正确关闭。**所以 SqlSession 的最佳的作用域是请求或方法作用域**。

![image-20210302164729870](/Users/aixin/Library/Application Support/typora-user-images/image-20210302164729870.png)

## 4.ResultMap

解决的问题：类字段与数据库表的字段名不一致。

搭建环境：把之前的环境复制过来，然后把实体类的字段修改为和数据库表不一致

查询后的接口该字段数据为null

<img src="/Users/aixin/Library/Application Support/typora-user-images/image-20210302173913346.png" alt="image-20210302173913346" style="zoom:50%;" />

分析原因：

- select * from user where id = #{id} 可以看做
  select id,name,pwd from user where id = #{id}
- mybatis会根据这些查询的列名(会将列名转化为小写,数据库不区分大小写) , 去对应的实体类中查找
  相应列名的set方法设值 , 由于找不到setPwd() , 所以password返回null ; 【自动映射】

解决方法：

方法一：为列名指定别名 , 别名和java实体类的属性名一致 。

```xml
<select id="getAllUsers" resultType="user">
  select id,name,pwd as password from user;
</select>
```

方法二：修改set方法，不推荐。

```java

//	public void setPassword(String password) {
//		this.password = password;
//	}
	
	public void setPwd(String password) {
			this.password = password;
	}
```

方法三：**使用结果集映射->ResultMap** 【推荐】

```xml
<!--column不做限制，可以为任意表的字段，而property须为type 定义的pojo属性-->
<resultMap id="唯一的标识" type="映射的pojo对象">
  <id column="表的主键字段，或者可以为查询语句中的别名字段" jdbcType="字段类型" property="映射pojo对象的主键属性" />
  <result column="表的一个字段（可以为任意表的一个字段）" jdbcType="字段类型" property="映射到pojo对象的一个属性（须为type定义的pojo对象中的一个属性）"/>
  <association property="pojo的一个对象属性" javaType="pojo关联的pojo对象">
    <id column="关联pojo对象对应表的主键字段" jdbcType="字段类型" property="关联pojo对象的主席属性"/>
    <result  column="任意表的字段" jdbcType="字段类型" property="关联pojo对象的属性"/>
  </association>
  <!-- 集合中的property须为oftype定义的pojo对象的属性-->
  <collection property="pojo的集合属性" ofType="集合中的pojo对象">
    <id column="集合中pojo对象对应的表的主键字段" jdbcType="字段类型" property="集合中pojo对象的主键属性" />
    <result column="可以为任意表的字段" jdbcType="字段类型" property="集合中的pojo对象的属性" />  
  </collection>
</resultMap>
```

```xml
<resultMap type="com.xin.beans.User" id="userMap">
  <id column="id" property="id"/>
  <result column="name" property="name"/>
  <result column="pwd" property="password"/>
</resultMap>

<select id="getAllUsers" resultMap="userMap">
 select * from user;
</select>
```

如果世界总是这么简单就好了。但是肯定不是的，数据库中，存在一对多，多对一的情况，我们之后会使用到一些高级的结果集映射，association，collection这些，我们将在之后讲解，今天你们需要把这些知识都消化掉才是最重要的！理解结果集映射的这个概念！

## 5.分页

### 5.1、日志

Mybatis内置的日志工厂提供日志功能，具体的日志实现有以下几种工具：

- SLF4J
- Apache Commons Logging
- Log4j 2
- Log4j
- JDK logging

具体选择哪个日志实现工具由MyBatis的内置日志工厂确定。它会使用最先找到的（按上文列举的顺序查找）。 如果一个都未找到，日志功能就会被禁用。

**标准日志实现**

```xml
<settings>
    <setting name="logImpl" value="STDOUT_LOGGING"/>
</settings>
```

输出结果：

<img src="C:\Users\aixin\AppData\Roaming\Typora\typora-user-images\image-20210302220000214.png" alt="image-20210302220000214" style="zoom: 80%;" />

### 5.2、Log4J

- Log4j是Apache的一个开源项目
- 通过使用Log4j，我们可以控制日志信息输送的目的地：控制台，文本，GUI组件....
- 我们也可以控制每一条日志的输出格式；
- 通过定义每一条日志信息的级别，我们能够更加细致地控制日志的生成过程。最令人感兴趣的就是，这些可以通过一个配置文件来灵活地进行配置，而不需要修改应用的代码。

**使用方法:**

导入jar包：







