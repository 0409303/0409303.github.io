---
    title: mybatis与spring整合详解
    categories: java-mybatis
    tags:
    creator: cjq
    create_time: 2021/01/28

---

[mybatis与spring整合](https://km.sankuai.com/page/625188235)

mybatis可以单独使用，也可以与spring进行整合。我们在自己的项目中，需要引入相关依赖，如你的项目管理工具使用的是maven，可以在pom.xml中添加以下配置

```xml
<!--spring依赖-->
  <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>4.3.17.RELEASE</version>
  </dependency>
  <dependency>
     <groupId>org.springframework</groupId>
     <artifactId>spring-context</artifactId>
     <version>4.3.17.RELEASE</version>
  </dependency>
  <!--mybatis spring 依赖-->
  <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
      <version>1.3.0</version>
  </dependency>
  <!--mybatis依赖-->
  <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.4.0</version>
  </dependency>
  <!--数据库连接池依赖-->
  <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid</artifactId>
      <version>1.1.9</version>
  </dependency>
  <!--数据库驱动依赖-->
  <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.39</version>
  </dependency>
```

在整合之后，框架的之间的依赖关系如下图所示：

<img src="../../../picture/java-mybatis/spring%E4%B8%ADmybatis%E4%BE%9D%E8%B5%96%E5%85%B3%E7%B3%BB.png" alt="依赖关系" style="zoom:33%;" />

# 整合一：SqlSessionFactoryBean

   在单独使用mybatis时，我们通过SqlSessionFactoryBuilder 来创建SqlSessionFactory。当mybatis与spring进行整合时，我们使用mybatis-spring提供的SqlSessionFactoryBean 来替代，SqlSessionFactoryBean实现了 Spring 的 FactoryBean 接口，用于创建 SqlSessionFactory 对象实例。 

   SqlSessionFactoryBean的配置有2种风格： 

- 保留mybatis的核心配置文件 
- 不保留mybatis的核心配置文件 

## **保留mybatis的核心配置文件**

我们将绝大部分关于mybatis的配置依然保留在mybatis的核心配置文件mybatis-config文件中，以下是一个示例： 

mybatis-config.xml 

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <setting name="cacheEnabled" value="true"/>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <setting name="defaultExecutorType" value="SIMPLE"/>
    </settings>

    <typeAliases>
        <package name="com.tianshouzhi.mybatis.entity"/>
    </typeAliases>

    <mappers>
        <mapper resource="mybatis/mappers/UserMapper.xml"/>
    </mappers>
</configuration>
```

细心的读者注意到，在mybatis-config.xml文件中我们并没有通过<environment>的子元素<dataSource>、<transactionManager>来配置数据源和事务管理器。即使配置了，也会被SqlSessionFactoryBean忽略。我们需要显式的为SqlSessionFactoryBean的dataSource属性引用一个数据源配置，如果不指定，在其初始化时就会抛出异常。 

​    此时SqlSessionFactoryBean配置方式如下： 

```xml
<!—-SqlSessionFactoryBean-—>
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean”>
     <!--数据源配置-->
     <property name="dataSource" ref="dataSource”/>
     <!--通过configLocation属性指定mybatis核心配置文件mybatis-config.xml路径-->
     <property name="configLocation" value="classpath:mybatis/mybatis-config.xml"/>
</bean>

<!--数据源使用druid-->
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="username" value="root"/>
        <property name="password" value=“your password"/>
        <property name="url" value="jdbc:mysql://localhost:3306/mybatis"/>
        <property name="driverClassName" value="com.mysql.jdbc.Driver”/>
        <!--其他配置-->
</bean>
```

## **不保留mybatis的核心配置文件**

   从mybatis-spring 1.3.0之后，我们可以移除mybatis-config.xml文件，将所有关于myabtis的配置都通过SqlSessionFactoryBean来指定。 

以下配置案例演示了与上述等价的配置： 

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="typeAliasesPackage" value="com.tianshouzhi.zebracost.entity”/>
        <!--从类路径下加载在mybatis/mappers包和它的子包中所有的 MyBatis 映射器 XML 文件-->
        <property name="mapperLocations" value="classpath*:mybatis/mappers/**/*.xml"></property>
        <property name="configuration">
            <bean class="org.apache.ibatis.session.Configuration">
                <property name="mapUnderscoreToCamelCase" value="true"/>
                <property name="cacheEnabled" value="true"/>
                <property name="defaultExecutorType" value="SIMPLE"/>
            </bean>
        </property>
 </bean>
```

在mybatis与spring整合后, 通常我们不会再直接使用SqlSessionFactory。mybatis-spring提供了其他更加易于操作的工具类，如SqlSessionTemplate、SqlSessionDaoSupport，当然还有其他更加高级的使用方式，如：MapperFactoryBean，MapperScannerConfigurer。

# **整合二：使用SqlSessionTemplate**

   SqlSessionTemplate 是 mybatis-spring 的核心，其实现了SqlSession接口。在使用了SqlSessionTemplate之后，我们不再需要通过SqlSessionFactory.openSession()方法来创建SqlSession实例；使用完成之后，也不要调用SqlSession.close()方法进行关闭。另外，对于事务，SqlSessionTemplate 将会保证使用的 SqlSession 是和当前 Spring 的事务相关的。 

SqlSessionTemplate依赖于SqlSessionFactory，其配置方式如下所示： 

```xml
<bean id="sqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg index="0" ref="sqlSessionFactory"/>
</bean>
<bean id="userDao" class="com.tianshouzhi.mybatis.dao.UserDao"/>
```

   之后我们可以在UserD类中直接进行注入。SqlSessionTemplate 是线程安全的, 可以被多个 DAO 所共享使用，以下是一个示例：

```java
public class UserDao {
    private static String NAMESPACE = "com.tianshouzhi.zebracost.UserMapper";

    @Autowired
    SqlSessionTemplate sqlSession;

    public User selectById(int id) {
        User user = sqlSession.selectOne(NAMESPACE + ".selectById",id);
        return user;
    }
}
```



  SqlSessionTemplate本质上是一个代理，关于SqlSessionTemplate的源码分析，后期会抽空编写。

# **整合三：继承SqlSessionDaoSupport**

mybatis提供了抽象类SqlSessionDaoSupport，调用其getSqlSession()方法你会得到一个 SqlSessionTemplate,之后可以用于执行 SQL 方法, 就像下面这样:

```java
public class UserDao extends SqlSessionDaoSupport{
    private static String NAMESPACE = "com.tianshouzhi.zebracost.UserMapper";

    public User selectById(int id) {
        User user = getSqlSession().selectOne(NAMESPACE + ".selectById",id);
        return user;
    }
}
```

  SqlSessionDaoSupport 需要一个 sqlSessionFactory 或 sqlSessionTemplate 属性来设置 。如果两者都被设置了 , 那么SqlSessionFactory是被忽略的。由于我们的UserDao类继承了SqlSessionDaoSupport，所以你可以在UserDao类中进行设置：

```xml
<bean id="userDao" class="com.tianshouzhi.zebracost.dao.UserDao">
        <property name="sqlSessionFactory" ref="sqlSessionFactory"/>
 </bean>
```

 或：

```xml
<bean id="userDao" class="com.tianshouzhi.zebracost.dao.UserDao">
     <property name="sqlSessionTemplate" ref="sqlSessionTemplate"/>
 </bean>
```

事实上，如果你提供的是一个SqlSessionFactory，SqlSessionDaoSupport内部也会使用其来构造一个SqlSessionTemplate实例。

# **整合四：MapperFactoryBean**

  无论是使用SqlSessionTemplate，还是继承SqlSessionDaoSupport，我们都需要手工编写DAO类的代码。熟悉mybatis同学知道，SqlSession有一个getMapper()方法，可以让我们通过映射器接口的方式来使用mybatis。 

例如针对有以下UserMapper.xml 

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd”>
<mapper namespace=“com.tianshouzhi.mybatis.mapper.UserMapper”>

  <select id="selectById" resultType="Blog">
    select * from user where id = #{id}
  </select>
  <insert id="insert" paramterType="com.tianshouzhi.mybatis.domain.User”>
    insert into user(name,age) values(#{name},#{age})
  </insert>
</mapper>
```

我们可以定义以下UserMapper接口：

```java
package com.tianshouzhi.mybatis.mapper;
import com.tianshouzhi.mybatis.domain.User;

public interface UserMapper {
    public int insert(User user);
    public User selectById(int id);
}
```

细心的读者注意到： 

1、映射文件的namespace属性值”com.tianshouzhi.mybatis.mapper.UserMapper"就是UserMapper接口的全路径 

2、映射文件中的<select> 、<insert>元素的id属性值”selectById”、"insert"，对应UserMapper接口中的selectById方法和insert方法。 

之后，我们调用UserMapper接口中的方法，就会执行UserMapper.xml中对应的sql。 



你甚至可以通过注解的方式直接将sql写到UserMapper接口的方法上，此时你就不再需要UserMapper.xml，如： 

```java
package com.tianshouzhi.mybatis.mapper;
import com.tianshouzhi.mybatis.quickstart.domain.User;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Select;

public interface UserMapper {
    @Insert("insert into user(name,age) values(#{id},#{name},#{age})")
    public int insert(User user);
    @Select("select * from user where id = #{id}")
    public User selectById(int id);
}
```

此时我们可以修改mybatis-config.xml中<mapper>元素的resource属性为class属性，指定UserMapper接口的全路径，如下

```xml
<mappers>
   <!--<mapper resource="mappers/UserMapper.xml"/>—>
   <!--使用class属性指定UserMapper接口全路径-->
   <mapper class="com.tianshouzhi.mybatis.mapper.UserMapper"/>
</mappers>
```

之后，我们可以通过以下方式来使用UserMapper接口，更加直观，不容易出错。

```java
SqlSession session = sqlSessionFactory.openSession();
try {
  UserMapper mapper = session.getMapper(UserMapper.class);
  User user = mapper.selectById(1);//等价于执行UserMapper中id属性值为”selectById”的<select>元素中的sql，或者selectById方法上注解的sql
} finally {
  session.close();
}
```



但是在与spring进行整合时，是否有更加简单的使用方法呢？能够在一个业务Bean中注入UserMapper接口，不需要通过SqlSession的getMapper来创建。我们期望的使用方式如下：

```java
public class UserService {
    @Autowired
    private UserMapper userMapper;

    public void insert(User user){
        System.out.println("create a new user!"+user);
        userMapper.insert(user);
    }
}
```

 直接这样操作显然是会报错的，因为UserMapper是一个接口，且不是spring管理的bean，因此无法直接注入。 

   这个时候，本节的主角MapperFactoryBean登场了，通过如下配置，MapperFactoryBean会针对UserMapper接口创建一个代理，并将其变成spring的一个bean。MapperFactoryBean继承了SqlSessionDaoSupport类，这也是为什么我们先介绍SqlSessionDaoSupport，再介绍MapperFactoryBean的原因。显然的，你可以想到，我们可以为MapperFactoryBean指定一个SqlSessionTemplate或者SqlSessionFactory，如果两个属性都设置了,那么 SqlSessionFactory 就会被忽略。 

通过以下配置，我们就可以在一个业务bean中直接注入UserMapper接口了：

```xml
<bean id="userMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">
  <property name="mapperInterface" value="com.tianshouzhi.mybatis.mapper.UserMapper" />
  <property name="sqlSessionFactory" ref="sqlSessionFactory" />
</bean>
```

你可能想知道MapperFactoryBean为什么具有这样的魔力，但是这不是本文的重点，本文讲解的是mybatis是如何spring进行整合的。笔者将其会在其他章节分析MapperFactoryBean的源码。 

   上述方式，已经是mybatis与spring进行时理想的方式了。但是如果你的业务很复杂，有许多的XxxMapper接口要配置，针对每个接口都配置一个MapperFactoryBean，会使得我们的配置文件很臃肿。关于这一点，mybatis团队提供了MapperScannerConfigurer来帮助你解决这个问题。 

# **整合五：MapperScannerConfigurer**

   通常我们在开发时，会将同一类型的类放于同一个包下，例如现在我们com.tianshouzhi.mybatis.mappers包下，有两个接口：UserMapper、UserAccountMapper。 

  MapperScannerConfigurer可以指定扫描某个包，将这个包下的所有接口，自动的为每个映射器接口都注册一个MapperFactoryBean。配置如下： 

```xml
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
  <property name="basePackage" value="org.tianshouzhi.mybatis.user.mappers" />
</bean>
```

  其中：basePackage 属性是让你为映射器接口的包路径。如果的映射器接口位于不同的包下，可以使用分号”;”或者逗号”,”进行分割。如：

```xml
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
  <property name="basePackage" value=“org.tianshouzhi.mybatis.user.mappers;org.tianshouzhi.mybatis.product.mappers" />
</bean>
```

 如果你指定的还包含子包，子包中的映射器接口递归地被搜索到。因此对于上述配置，我们可以通过公共的包名"org.tianshouzhi.mybatis”进行简化。如：

```xml
 <property name="basePackage" value="org.tianshouzhi.mybatis" />
```

你可能想到了，如果指定的公共的包名下面还包含了一些其他的接口，这些接口是你作为其他用途使用到的，并不能作为mybatis的映射器接口来使用。此时，你可以通过markerInterface属性或者annotationClass属性来进行过滤。 

  对于markerInterface，首先，你需要定义一个标记接口，接口名随意，这个接口中不需要定义任何方法，如： 

```java
public interface MybatisMapperInterface{}
```

接着，你需要将你的映射器接口继承MybatisMapperInterface，如：

```java
public interface UserMapper implements MybatisMapperInterface{
   //...映射器方法
}
```

  此时你可以指定MapperScannerConfigurer中指定，只有继承了MybatisMapperInterface接口的子接口，才为其自动注册MapperFactoryBean，如：

```xml
<property name="markerInterface" value="com.tianshouzhi.mybatis.MybatisMapperInterface"/>
```

对于annotationClass属性，作用是类似的，根据注解进行过滤。一般我们是映射器接口上添加mybatis提供的@Mapper注解进行过滤，你也可以自定义一个注解。配置如下：

```xml
<property name="annotationClass" value="org.apache.ibatis.annotations.Mapper"/>
```

​    如果同时指定了markerInterface和annotationClass属性，那么只有同时满足这两个条件的接口才会被注册为MapperFactoryBean。 

  细心的读者可能意识到了，到目前，我们还没有为MapperScannerConfigurer指定一个SqlSessionFactory，或者SqlSessionTemplate。前面配置MapperFactoryBean的时候，我们已经看到，我们至少需要为其提供一项。之所以不指定，是因为MapperScannerConfigurer将会spring上下文中自动进行寻找类型为SqlSessionFactory，或者SqlSessionTemplate的bean，然后利用其来创建MapperFactoryBean实例。 

   当然你也可以手工的进行指定任意一个，如：

```xml
<property name="sqlSessionFactory" ref=“sqlSessionFactory"/>
<!--<property name="sqlSessionTemplate" ref=“sqlSessionTemplate"/>-->
```

然而，sqlSessionFactory和sqlSessionTemplate属性已经不建议使用了。原因在于，这两个属性不支持你使用spring提供的PropertyPlaceholderConfigurer的属性替换。例如你配置了SqlSessionFactoryBean来创建SqlSessionFactory实例，前面已经看到必须为其指定一个dataSource属性。很多用户习惯将数据源的配置放到一个独立的配置文件，如jdbc.properties文件中，之后在spring配置中，通过占位符来替换，如：

```xml
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="driverClassName" value="${jdbc.driver}"/>
        <!--其他配置-->
</bean>
```

对于这样的配置，spring在初始化时会报错，因为MapperScannerConfigurer会在PropertyPlaceholderConfigurer初始化之前，就加载dataSource的配置，而此时PropertyPlaceholderConfigurer还未准备好替换的占位符的内容，所以抛出异常。 

  当然，这个问题并不是无解，我们可以使用sqlSessionFactoryBeanName、sqlSessionTemplateBeanName属性来替代sqlSessionFactory和sqlSessionTemplate属性。如下： 

```xml
<property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
<!--<property name="sqlSessionTemplateBeanName" value="sqlSessionTemplate”/>-->
```

此时，你依然可以放心大胆的在你的数据源配置中，使用占位符了。 

  事实上，笔者总是建议你，在MapperScannerConfigurer的配置中，显示的指定sqlSessionFactoryBeanName或sqlSessionTemplateBeanName。如果你不指定，MapperScannerConfigurer就会在spring上下文中自动的寻找类型为SqlSessionFactory或者SqlSessionTemplate的bean。如果你的数据源配置中使用了占位符，还是会报错。 

  最后，你可能想知道，为什么MapperScannerConfigurer指定一个basePackage属性，就可以为包下的每个接口都注册一个MapperFactoryBean？其内部是如何自动在spring上下文中寻找类型为SqlSessionFactory或者SqlSessionTemplate的bean实例的？以及为什么又做了如此多限制，只有指定sqlSessionFactoryBeanName或sqlSessionTemplateBeanName才能在数据源的配置中使用占位符？ 

   关于这些问题，笔者将在MapperScannerConfigurer的源码分析中进行解答。 

# **整合六：@MapperScan**

   如果读者习惯使用注解，而不是xml文件的方式进行配置，mybatis-spring提供了@MapperScan注解，其用于取代MapperScannerConfigurer。以下演示了如何通过注解的方式来配置mybatis。

```java
@Configuration
@MapperScan(basePackages = "com.tianshouzhi.security.mapper”,//等价于MapperScannerConfigurer的basePackage属性
        markerInterface = MybatisMapperInterface.class,//等价于MapperScannerConfigurer的markerInterface属性, 默认为null
        annotationClass = MybatisMapper.class,//等价于MapperScannerConfigurer的annotationClass属性，默认为null
        sqlSessionFactoryRef = "sqlSessionFactory")//等价于MapperScannerConfigurer的sqlSessionFactoryBeanName属性
public class DatabaseConfig {
    //定义数据源
    @Bean
    public DataSource dataSource() {
        SimpleDriverDataSource dataSource = new SimpleDriverDataSource();
        dataSource.setUsername(“your username");
        dataSource.setPassword(“you password");
        dataSource.setUrl("jdbc:mysql://localhost:3306/mybatis?characterEncoding=UTF-8");
        dataSource.setDriverClass(com.mysql.jdbc.Driver.class);
        return dataSource;
    }
    //定义SqlSessionFactoryBean
    @Autowired
    @Bean("sqlSessionFactory")
    public SqlSessionFactoryBean sqlSessionFactory(DataSource dataSource) throws Exception {
        SqlSessionFactoryBean ssfb = new SqlSessionFactoryBean();
        ssfb.setDataSource(dataSource);
        return ssfb;
    }
}
```



# **整合七：事务**

  上述所有的配置，还没有涉及到mybatis与spring进行整合另一个核心要点，即事务。整合后，我们需要将事务委派给spring来管理。 

   spring使用PlatformTransactionManager接口来表示一个事务管理器。其有2个重要的实现类： 

​    DataSourceTransactionManager：用于支持本地事务，简单理解，你可以认为就是操作单个数据库的事务，其内部也是通过操作java.sql.Connection来开启、提交(commit)和回滚(rollback)事务。 

​    JtaTransactionManager：用于支持分布式事务，其实现了JTA规范，使用XA协议进行两阶段提交。 

在本文中，我们主要介绍的是DataSourceTransactionManager，绝大部分情况下， 我们使用的都是这个事务管理器。其配置方式如下： 

```xml
<!--spring 事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--默认事务超时时间-->
        <property name="defaultTimeout" value="30000"/>
        <!--数据源-->
        <property name="dataSource" ref="dataSource" />
        <!--提交失败的话，也进行回滚-->
        <property name="rollbackOnCommitFailure" value="true"/>
    </bean>
```

关于事务管理器使用方式有2种：声明式事务管理、编程式事务管理 

## **声明式事务管理**

**1、基于注解**

首先，你需要配置事务管理是基于注解驱动的，如下： 

```xml
<tx:annotation-driven transaction-manager="transactionManager"/>
```

之后，在业务bean的方法上添加@Transactional注解，此时这个方法就自动具备了事务的功能，如果出现异常，会自动回滚，没有出现异常则自动交。

```java
@Transactional  
public void transfer(final String out,final String in,final Double money) {  
            accountMapper.outMoney(out, money);  
            int i=1/0;  
            accountMapper.inMoney(in, money);  
    }  
```



基于注解的形式的声明式事务管理器，是最为简单的，也是建议使用的方式。 

**2、基于切面**

   如果你有很多方法，都需要有事务管理，你觉得每个方法都添加@Transactional注解比较麻烦，此时你可以使用以下配置取代<tx:annotation-driven>元素。 

```xml
<!-- 1.配置事务通知：（事务的增强） -->  
   <tx:advice id="txAdvice" transaction-manager="transactionManager">  
   <tx:attributes>  
        <tx:method name="transfer"/>  
   </tx:attributes>  
   </tx:advice>  
    
<!-- 2.配置切面 -->  
<aop:config>  
   <!-- 配置切入点 -->  
   <aop:pointcut id="tx_pointcut" expression="execution(*com.tianshouzhi.mybatis.AccountService+.*(..))"/>  
   <!-- 配置切面 -->  
   <aop:advisor pointcut-ref="tx_pointcut" advice-ref="txAdvice" />
</aop:config>  
```

 事实上，基于切面的配置，太麻烦了，即使对于一些老鸟，长时间没有编写过类似的配置，可能也无法立即正确的进行配置，没有@Transactional注解来的直观。 

## **编程式事务管理**

如果上述两种声明式事务的配置都不是你要想要，那么你可以采取编程式事务。你可以在业务bean中注入事务管理器，然后进行编程式事务的管理，如： 

```java
public class AccountService{
  @Autowired
  private DataSourceTransactionManager transactionManager;

  @Autowired
  private AccountMapper accountMapper;

  public void transfer(final String out,final String in,final Double money){
      DefaultTransactionDefinition def = new DefaultTransactionDefinition();
      def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
    
      TransactionStatus status = transactionManager.getTransaction(def);
      try {
         accountMapper.outMoney(out, money);  
         int i=1/0;  
         accountMapper.inMoney(in, money);  
        transactionManager.commit(status);
      }
      catch (MyException ex) {
         transactionManager.rollback(status);
         throw ex;
      }
   }
}
```



另外，Spring还提供了一个TransactionTemplate，用于简化编程式事务代码的编写，配置方式如下：

```xml
<bean id="transactionTemplate" class="org.springframework.transaction.support.TransactionTemplate">  
    <property name="transactionManager"ref="transactionManager"></property>  
</bean>   
```

使用方式如下：

```java
    public void transfer(final String out,final String in,final Double money) {  
        //在事务模板中执行操作  
        transactionTemplate.execute(new TransactionCallbackWithoutResult(){  
        @Override  
        protected void doInTransactionWithoutResult(TransactionStatustransactionstatus){  
            accountMapper.outMoney(out, money);  
            int i=1/0;  
            accountMapper.inMoney(in, money);  
         }  
        });  
    }  
```

总的来说，编程式事务管理还是显得略微麻烦。 



# **整合八：springboot**

   mybatis开发团队为Spring Boot 提供了 mybatis-spring-boot-starter。你需要引入如下依赖： 

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.1.1</version>
</dependency>
```



​    使用了该starter之后，只需要定义一个DataSource即可，它会自动利用该DataSource创建需要使用到的SqlSessionFactoryBean、SqlSessionTemplate、以及ClassPathMapperScanner来自动扫描你的映射器接口，并针对每个接口都创建一个MapperFactoryBean，注册到Spring上下文中。 

​    关于mybatis-spring-boot-starter如何实现自动配置的相关源码，参见：org.mybatis.spring.boot.autoconfigure.MybatisAutoConfiguration类。 

  默认情况下，扫描的basePackage是spring boot的根目录(这里指的是应用启动类Application.java类所在的目录)，且只会对添加了@Mapper注解的映射器接口进行注册。 

   因此，最简单的情况下，你只需要在application.yml中进行数据源的相关配置即可，以下配置依赖于spring-boot-starter-jdbc： 

```properties
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/mybatis
    username: root
    password: your password
```

 之后，你就可以直接在你的业务bean中注入映射器接口来使用了。 

  需要注意的是，一旦你自己提供了MapperScannerConfigurer，或者配置了MapperFactoryBean，那么mybatis-spring-boot-starter的自动配置功能将会失效。此时所有关于mybatis与spring进行整合的配置，都需要由你自行控制。 

# **整合九：多数据源**

有的时候，应用需要访问多个数据库，假设现在有两个mysql库db_user、db_acount。这个时候我们就需要配置2个数据源，来连接不同的库，如下： 

```xml
<bean id="ds_user" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="username" value="root"/>
        <property name="password" value="shxx12151022"/>
        <property name="url" value="jdbc:mysql://192.168.0.1:3306/db_user"/>
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <!--其他配置-->
</bean>

<bean id="ds_account" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="username" value="root"/>
        <property name="password" value=“your password"/>
        <property name="url" value="jdbc:mysql://192.168.0.2:3306/db_account"/>
        <property name="driverClassName" value="com.mysql.jdbc.Driver”/>
        <!--其他配置-->
</bean>
```

针对上述的数据源ds_user,ds_account，我们需要为每个都配置一个SqlSessionFactoryBean，如下：

```xml
<bean id="ssf_user" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="ds_user"/>
        <property name="mapperLocations" value="classpath*:mybatis/mappers/db_user/**/*.xml"></property>
</bean>

<bean id="ssf_account" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="ds_account"/>
        <property name="mapperLocations" value="classpath*:mybatis/mappers/db_account/**/*.xml"></property>
</bean>
```

细心的读者可能已经注意到，这两个SqlSessionFactoryBean分别操作数据源ds_user、ds_account，且mapperLocations属性值指向的是不同的目录： 

- 在ssf_user中，指定关于ds_user的映射器xml文件都位于类路径的mybatis/mappers/db_user/目录下 
- 在ssf_account中，指定关于ds_account的映射器xml文件都位于类路径的mybatis/mappers/db_account/目录下 

  如果你是使用映射接口的方式来操作mybatis，那么还应该针对ssf_user和ssf_account两个SqlSessionFactoryBean，各配置一个MapperScannerConfigurer。如下：

```xml
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
  <property name="basePackage" value="com.tianshouzhi.mybatis.mappers.user" />
  <property name="sqlSessionFactoryBeanName" value="ssf_user"/>
</bean>


<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
  <property name="basePackage" value="com.tianshouzhi.mybatis.mappers.account" />
  <property name="sqlSessionFactoryBeanName" value="ssf_account"/>
</bean>
```



可以看到，两个MapperScannerConfigurer的basePackage属性是不同的，我们将操作db_user库下的映射器XxxMapper接口都放在com.tianshouzhi.mybatis.mappers.user包中，将我们将操作db_account库下的映射器XxxMapper接口都放在com.tianshouzhi.mybatis.mappers.account包中。 

  再次提醒，在使用多数据源时，将操作不同库的xml映射文件、以及对应的映射器接口放到不同的目录下很重要，如果不这样，在使用时你几乎100%会遇到问题。

最后，由于你现在有2个数据源，因此你应该配置两个事务管理器，如： 

```xml
<bean id="userTxManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="ds_user" />
</bean>

<bean id="accountTxManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="ds_account" />
</bean>
```

如果你是使用@Transactional注解来进行声明式事务管理，应该要指定你使用的是哪一个事务管理器，如：

```java
@Transactional("accountTxManager")  
public void transfer(final String out,final String in,final Double money) {  
            accountMapper.outMoney(out, money);  
            int i=1/0;  
            accountMapper.inMoney(in, money);  
    }  
```

一个事务管理器，只能保证操作单个库的事务，例如在这里，accountTxManager只能保证db_account库的事务，证userTxManager只能保证db_user库的事务。 

  如果你在一个方法内，想同时操作两个库，并保证事务，使用普通的DataSourceTransactionManager已经无法满足这种需求，这属于分布式事务的范畴。在笔者的另一篇文章[atomikos JTA/XA全局事务](http://www.tianshouzhi.com/api/tutorials/distributed_transaction/386)演示了如何使用atomikos与mybatis、spring进行整合，来进行分布式事务的管理。 

# 整合十：zebra  

你应该使用：  

1、使用zebra-api中提供了GroupDataSource(读写分离)或者ShardDataSource(分库分表)，而不是直接使用第三方的数据库连接池。  

2、使用zebra-dao提供了ZebraMapperScannerConfigurer替代mybatis-spring原生的MapperScannerConfigurer  

3、此外，你应该使用zebra-ds-monitor-client，只需引入依赖即可，会自动将连接池的监控信息上报到cat中