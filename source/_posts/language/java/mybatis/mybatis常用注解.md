---
    title: mybatis常用注解
    categories: java-mybatis
    tags:
    creator: cjq
    create_time: 2021/01/28

---

[mybatis注解](https://km.sankuai.com/page/625212960)

![](https://img-blog.csdnimg.cn/20210220193437502.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ExMjQ1OA==,size_16,color_FFFFFF,t_70)

### 1 介绍 

   mybatis 最初配置信息是基于 XML ,映射语句(SQL)也是定义在 XML 中的。而到了 MyBatis 3提供了新的基于注解的配置。mybatis提供的注解有很多，大致可以分为以下几类： 

- **增删改查：**@Insert、@Update、@Delete、@Select、@MapKey、@Options、@SelelctKey、@Param、@InsertProvider、@UpdateProvider、@DeleteProvider、@SelectProvider 
- **结果集映射：**@Results、@Result、@ResultMap、@ResultType、@ConstructorArgs、@Arg、@One、@Many、@TypeDiscriminator、@Case 

- **缓存：**@CacheNamespace、@Property、@CacheNamespaceRef、@Flush 

   绝大部分注解，在xml映射文件中都有元素与之对应，但是不是所有。此外在mybatis-spring中提供了@Mapper注解和@MapperScan注解，用于和spring进行整合。 



### 2 增删改查相关注解

| **注解**                                                     | **使用对象** | **相对应的 XML**                    | **描述**                                                     |
| ------------------------------------------------------------ | ------------ | ----------------------------------- | ------------------------------------------------------------ |
| @Insert @Update @Delete @Select                              | 方法         | <insert> <update> <delete> <select> | 这四个注解分别代表将会被执行的 SQL 语句。它们用字符串数组（或单个字符串）作为参数。如果传递的是字符串数组，字符串之间先会被填充一个空格再连接成单个完整的字符串。这有效避免了以 Java 代码构建 SQL 语句时的“丢失空格”的问题。然而，你也可以提前手动连接好字符串。属性有：value，填入的值是用来组成单个 SQL 语句的字符串数组。 |
| @Options                                                     | 方法         | 映射语句的属性                      | 这个注解提供访问大范围的交换和配置选项的入口，它们通常在映射语句上作为属性出现。Options 注解提供了通俗易懂的方式来访问它们，而不是让每条语句注解变复杂。属性有：useCache=true, flushCache=FlushCachePolicy.DEFAULT, resultSetType=FORWARD_ONLY, statementType=PREPARED, fetchSize=-1, timeout=-1, useGeneratedKeys=false, keyProperty="id", keyColumn="", resultSets=""。值得一提的是， Java 注解无法指定 null 值。因此，一旦你使用了 Options 注解，你的语句就会被上述属性的默认值所影响。要注意避免默认值带来的预期以外的行为。     注意： keyColumn 属性只在某些数据库中有效（如 Oracle、PostgreSQL等）。请在插入语句一节查看更多关于 keyColumn 和 keyProperty 两者的有效值详情。 |
| @MapKey                                                      | 方法         |                                     | 这是一个用在返回值为 Map 的方法上的注解。它能够将存放对象的 List 转化为 key 值为对象的某一属性的 Map。属性有： value，填入的是对象的属性名，作为 Map 的 key 值。 |
| @SelectKey                                                   | 方法         | <selectKey>                         | 这个注解的功能与 <selectKey> 标签完全一致，用在已经被 @Insert 或 @InsertProvider 或 @Update 或 @UpdateProvider 注解了的方法上。若在未被上述四个注解的方法上作 @SelectKey 注解则视为无效。如果你指定了 @SelectKey 注解，那么 MyBatis 就会忽略掉由 @Options 注解所设置的生成主键或设置（configuration）属性。属性有：statement 填入将会被执行的 SQL 字符串数组，keyProperty 填入将会被更新的参数对象的属性的值，before 填入 true 或 false 以指明 SQL 语句应被在插入语句的之前还是之后执行。resultType 填入 keyProperty 的 Java 类型和用 Statement、 PreparedStatement 和 CallableStatement 中的 STATEMENT、 PREPARED 或 CALLABLE 中任一值填入 statementType。默认值是 PREPARED。 |
| @Param                                                       | 参数         | N/A                                 | 如果你的映射方法的形参有多个，这个注解使用在映射方法的参数上就能为它们取自定义名字。若不给出自定义名字，多参数（不包括 RowBounds 参数）则先以 "param" 作前缀，再加上它们的参数位置作为参数别名。例如 #{param1}, #{param2}，这个是默认值。如果注解是 @Param("person")，那么参数就会被命名为 #{person}。 |
| @InsertProvider @UpdateProvider @DeleteProvider @SelectProvider | 方法         | <insert> <update> <delete> <select> | 允许构建动态 SQL。这些备选的 SQL 注解允许你指定类名和返回在运行时执行的 SQL 语句的方法。（自从MyBatis 3.4.6开始，你可以用 CharSequence 代替 String 来返回类型返回值了。）当执行映射语句的时候，MyBatis 会实例化类并执行方法，类和方法就是填入了注解的值。你可以把已经传递给映射方法了的对象作为参数，"Mapper interface type" 和 "Mapper method" 会经过 ProviderContext （仅在MyBatis 3.4.5及以上支持）作为参数值。（MyBatis 3.4及以上的版本，支持多参数传入）属性有： type, method。type 属性需填入类。method 需填入该类定义了的方法名。注意 接下来的小节将会讨论类，能帮助你更轻松地构建动态 SQL。 |



映射器接口示例，假设有以下UserMapper接口： 

```java
public interface UserMapper {

   @Insert("INSERT INTO user(id,name) VALUES (#{id},#{name})”)
   @Options(useGeneratedKeys = true, keyColumn = "id", keyProperty = "id") 
   public int insert(User user);

   @Update(" UPDATE user SET name=#{name} WHERE id=#{id}")
   public int update(User user);

   @Delete(“ DELETE FROM user WHERE id=#{id}")
   public int delete(int id);

   @Select("SELECT id,name FROM user WHERE id= #{id}")
   public User selectById(int id);

   @Select("SELECT id,name FROM user")
   public List<User> selectAll();

   @Select("SELECT id,name FROM user")
   @MapKey("id")
   public Map<Integer,User> selectMap();

   @Select({ "<script>",
      "SELECT id,name " + "FROM user " + "WHERE id IN "
            + "<foreach item='id' index='index' collection=‘array' open='(' separator=',' close=')'> " 
                 + "#{id}"
            + "</foreach> ",
      "</script>" })
   public List<User> selectByIds(int... ids);

   @Select("SELECT id,name FROM user LIMIT #{offset},#{limit}")
   public List<User> selectPage(@Param("offset") int offset, @Param("limit") int limit);
}
```



**说明：**

#### 2.1 @Insert、@Update、@Delete、@Options、@SelectKey注解

   mybatis会根据接口方法上的@Insert、@Update、@Delete注解，分别去调用SqlSession的insert、update、delete方法。这个几个方法返回的都是一个int，表示影响的记录行数。 

  特别的，在UserMapper接口的insert方法上，除了添加了@Insert注解，还添加了@Options注解。在上面的案例中，@Options注解用于获取自动生成主键，并设置到User实体中。此外，@SelectKey注解也可以用于获取自动生成的主键，使用方式如下： 

```java
@Insert("INSERT INTO user(id,name) VALUES (#{id},#{name})")
@SelectKey(statement = "SELECT LAST_INSERT_ID()", keyProperty = "id", before = false, resultType = Integer.class) 
public int insert(User user); 
```



#### 2.2 @Select注解

   在上面的案例中，UserMapper的selectById、selectAll、selectMap、selectByIds、selectPage方法都添加了@Select注解。mybatis会根据方法的返回值类型User、List<User>、Map<Integer,User>判断是调用SqlSession的selectOne、selectList还是selectMap方法。 

#### 2.3 @MapKey注解

特别的，对于返回值是Map的情况，UserMapper的selectMap方法上额外添加了一个@MapKey("id")注解，表示将User实例的id属性当做Map的key。 

#### 2.4  动态sql与<script>标签

  在UserMapper的selectByIds方法中，可以看到@Select注解里填写的SQL前后分别添加了<script>、</script>，这是因为SQL中使用了动态sql标签<foreach>。不管是@Insert、@Update、@Delete、@Select注解，只要SQL里使用了mybatis的动态sql标签(包括：if、choose …when ...otherwise、trim 、where、 set、foreach、bind)等，都建议在sql前后分别加上<script>、</script>，否则可能会出现一些参数找不到的情况。 

#### 2.5 @Param注解

   @Param注解用于给方法参数起一个名字。以下是笔者总结的使用原则：

- 在方法只接受一个参数的情况下，可以不使用@Param。
- 在方法接受多个参数的情况下，建议一定要使用@Param注解给参数命名。

   例如上述案例的selectPage方法接受2个参数，所以其两个参数都使用了@Param注解。@Param注解看起来配置最简单，实际上理解确实最复杂，下面进行详细的介绍。

   前面已经提到，当映射器接口定义的方法被调用时，mybatis内部根据方法上注解：@Insert、@Update、@Delete、@Select来选择调用SqlSession的insert、update、delete、selectXXX方法。以下是SqlSession接口的相关方法定义(部分省略)：

```java
public interface SqlSession extends Closeable {
  ...
  <T> T selectOne(String statement);
  <T> T selectOne(String statement, Object parameter);
  <E> List<E> selectList(String statement);
  <E> List<E> selectList(String statement, Object parameter);
  <K, V> Map<K, V> selectMap(String statement, String mapKey);
  <K, V> Map<K, V> selectMap(String statement, Object parameter, String mapKey);
  ...
  int insert(String statement);
  int insert(String statement, Object parameter);
  int update(String statement);
  int update(String statement, Object parameter);
  int delete(String statement);
  int delete(String statement, Object parameter);
  ...
}
```



   可以看到部分方法接受一个Object类型的parameter参数，另外一些方法则没有此参数。mybatis在执行前，除了会根据方法上注解：@Insert、@Update、@Delete、@Select来选择调用SqlSession的insert、update、delete、selectXXX方法；还会提前对传入映射器接口方法中的参数进行一些处理，然后再调用SqlSession的相应方法。逻辑如下： 

   1、如果映射器接口方法不接受参数，mybatis在执行时会调用相应无parameter参数的方法重载形式。例如，如UserMapper接口的selectAll方法，其不接受参数，返回值类型为List，因此调用SqlSession不接受paramter参数的selectList方法：

```java
<E> List<E> selectList(String statement); 
```

   2、如果映射器方法只有一个参数，并且这个参数没有使用@Param注解，则直接用这个参数来调用SqlSession相应接受Object类型parameter方法参数的重载形式。例如，如UserMapper接口的selectByIds方法，其接受1个int[]数组类型参数作为查询条件，返回值类型为List<User>，因此调用SqlSession接受paramter参数的selectList方法：

```java
<E> List<E> selectList(String statement, Object parameter); 
```

   3、如果映射器方法只接受一个参数，但是使用了@Param注解，也会先封装到Map中；活着映射器方法总是接受多个参数，不管有没有使用@Param注解指定参数，总是会先封装到一个Map中。之后，调用SqlSession的相应方法把这个Map当做parameter参数传入。

   例如：UserMapper接口的selectPage方法， 通过@Param("offset”)和@Param("limit”)为2个int参数指定了名字。 

```java
@Select("SELECT id,name FROM user LIMIT #{offset},#{limit}")
public List<User> selectPage(@Param("offset") int offset, @Param("limit") int limit); 
```

假设我们传入0和10，那么参数封装后的Map结构如下： 

 key     value 

\-------------------- 

 offset    0     //1 

 limit    10    //2 

 param1    0     //3 

 param2    10    //4 

关于Map中1、2两个key-value，比较好理解，是我们通过@Param注解指定的映射关系。而3、4两个key-value，实际上是参数位置(param1、param2…，下标从1开始)和参数值的映射关系，不管我们有没有使用@Param注解，存在多个参数的情况下，我们总是可以按照位置进行引用。 

   因此，将UserMapper的selectPage方法定义改成以下形式也是正确的： 

```java
//使用了@Param注解的情况下，依然根据参数位置进行引用(param1，param2…，下标从1开始)
@Select("SELECT id,name FROM user LIMIT #{param1},#{param2}")
public List<User> selectPage(@Param("offset") int offset, @Param("limit") int limit); 

//不使用@Param注解，直接根据参数位置进行引用
@Select("SELECT id,name FROM user LIMIT #{param1},#{param2}")
public List<User> selectPage(int offset, int limit); 
```

   显然，根据参数位置进行引用不太直观，因此建议在存在多个参数的情况，总是通过@Param注解显式的指定参数名。 



#### 2.6  @InsertProvider、@UpdateProvider、@DeleteProvider、@SelectProvider注解

这几个注解主要用于动态sql构建。

```java
public interface UserBuilderMapper {

   @SelectProvider(type = UserSqlBuilder.class, method = "buildSelectByIdSql")
   public User selectById(@Param("id") int id);

   @InsertProvider(type = UserSqlBuilder.class, method = "buildInsertSql")
   @Options(useGeneratedKeys = true, keyColumn = "id", keyProperty = "id")
   public int insert(User user);

   @UpdateProvider(type = UserSqlBuilder.class, method = "buildUpdateSql")
   public int update(User user);

   @DeleteProvider(type = UserSqlBuilder.class, method = "buildDeleteSql")
   public int delete(@Param("id") int id);


    //建议将sql builder以映射器接口内部类的形式进行定义
    public static class UserSqlBuilder {
      public static String buildSelectByIdSql(@Param("id") int id) {
         return new SQL() {
            {
               SELECT("id, name");
               FROM("user");
               WHERE("id=#{id}");
            }
         }.toString();
      }

      public static String buildInsertSql(User user) {
         return new SQL() {
            {
               INSERT_INTO("user");
               VALUES("name", "#{name}");
            }
         }.toString();
      }

      public static String buildUpdateSql(User user) {
         return new SQL() {
            {
               UPDATE("user");
               SET("name=#{name}");
               WHERE("id=#{id}");
            }
         }.toString();
      }

      public static String buildDeleteSql(@Param("id") int id) {
         return new SQL() {
            {
               DELETE_FROM("user");
                    WHERE("id=#{id}");
            }
         }.toString();
      }
   }
 }

}
```



### 3 结果集映射相关注解

| **注解**           | **使用对象** | **相对应的 XML** | **描述**                                                     |
| ------------------ | ------------ | ---------------- | ------------------------------------------------------------ |
| @Results           | 方法         | <resultMap>      | 结果映射的列表，包含了一个特别结果列如何被映射到属性或字段的详情。属性有：value, id。value 属性是 Result 注解的数组。这个 id 的属性是结果映射的名称。 |
| @Result            | 方法         | <result> <id>    | 在列和属性或字段之间的单独结果映射。属性有：id, column, javaType, jdbcType, typeHandler, one, many。id 属性是一个布尔值，来标识应该被用于比较（和在 XML 映射中的<id>相似）的属性。one 属性是单独的联系，和 <association> 相似，而 many 属性是对集合而言的，和<collection>相似。它们这样命名是为了避免名称冲突。 |
| @ResultMap         | 方法         | N/A              | 这个注解给 @Select 或者 @SelectProvider 提供在 XML 映射中的 <resultMap> 的id。这使得注解的 select 可以复用那些定义在 XML 中的 ResultMap。如果同一 select 注解中还存在 @Results 或者 @ConstructorArgs，那么这两个注解将被此注解覆盖。 |
| @ResultType        | 方法         | N/A              | 此注解在使用了结果处理器的情况下使用。在这种情况下，返回类型为 void，所以 Mybatis 必须有一种方式决定对象的类型，用于构造每行数据。如果有 XML 的结果映射，请使用 @ResultMap 注解。如果结果类型在 XML 的 <select> 节点中指定了，就不需要其他的注解了。其他情况下则使用此注解。比如，如果 @Select 注解在一个将使用结果处理器的方法上，那么返回类型必须是 void 并且这个注解（或者@ResultMap）必选。这个注解仅在方法返回类型是 void 的情况下生效。 |
| @ConstructorArgs   | 方法         | <constructor>    | 收集一组结果传递给一个结果对象的构造方法。属性有：value，它是形式参数数组。 |
| @Arg               | N/A          | <arg> <idArg>    | 单参数构造方法，是 ConstructorArgs 集合的一部分。属性有：id, column, javaType, jdbcType, typeHandler, select和 resultMap。id 属性是布尔值，来标识用于比较的属性，和<idArg> XML 元素相似。 |
| @One               | N/A          | <association>    | 复杂类型的单独属性值映射。属性有：select，已映射语句（也就是映射器方法）的全限定名，它可以加载合适类型的实例。fetchType会覆盖全局的配置参数 lazyLoadingEnabled。**注意** 联合映射在注解 API中是不支持的。这是因为 Java 注解的限制,不允许循环引用。 |
| @Many              | N/A          | <collection>     | 映射到复杂类型的集合属性。属性有：select，已映射语句（也就是映射器方法）的全限定名，它可以加载合适类型的实例的集合，fetchType 会覆盖全局的配置参数 lazyLoadingEnabled。**注意** 联合映射在注解 API中是不支持的。这是因为 Java 注解的限制，不允许循环引用 |
| @TypeDiscriminator | 方法         | <discriminator>  | 一组实例值被用来决定结果映射的表现。属性有：column, javaType, jdbcType, typeHandler 和 cases。cases 属性是实例数组。 |
| @Case              | N/A          | <case>           | 单独实例的值和它对应的映射。属性有：value, type, results。results 属性是结果数组，因此这个注解和实际的 ResultMap 很相似，由下面的 Results 注解指定。 |

#### 3.1 @ConstructorArgs、@Arg注解

  如果实体类没有无参的构造方法，那么我们就必须通过@ConstructorArgs与@Arg注解来提供实体类的构造方法参数信息。例如上述UserMapper的selectById方法，其返回一个User对象。假设User对象只有以下构造方法： 

```java
public User(Integer id, String name) { 
this.id = id; 
this.name = name; 
}
```



  那么需要在selectById方法上，添加@ConstructorArgs注解，提供构造方法信息，如下： 

```java
@Select("SELECT id,name FROM user where id= #{id}")
@ConstructorArgs({
@Arg(column = "id",javaType = Integer.class,id = true), 
@Arg(column = "name",javaType = String.class)}) 
public User selectById(int id); 
```



  注意，通常情况下，我们都建议为实体类提供无参的构造方法的，这是最佳实践的总结，因此@ConstructorArgs注解和@Arg注解基本使用不到。 



#### 3.2 @Results、@Result注解

如果实体字段的名称与数据库表字段名称不一致时，我们就需要显式的指定映射关系。这是通过@Results、@Result注解来指定的，例如为UserMapper的selectById指定映射关系： 

```java
@Select("SELECT id,name FROM user where id= #{id}")
@Results(id = "userMap", value = { 
       @Result(property = "id", column = "id", javaType = Integer.class,jdbcType = JdbcType.INTEGER,id = true), 
       @Result(property = "name", column = "name",javaType = String.class,jdbcType = JdbcType.VARCHAR)}) 
public User selectById(int id); 
```

其中： 

@Results注解：id属性用于给这个映射关系起一个名字(这里指定的为userMap)，其内部还包含了一个@Result[]来表示实体属性和数据库表字段的映射关系 

@Result注解：property属性是java实体属性的名称，column表示对应的数据库字段的名称。javaType和JdbcType属性可以不指定。 

#### 3.3 @ResultMap注解

   上述selectById方法已经通过@Results注解指定了结果映射关系，通过@ConstructorArgs指定了构造方法(必要的情况下才使用)，那么在其他的查询方法中，我们不需要重复定义，可以通过@ResultMap来引用@Results的id属性值进行复用。如我们在UserMapper的selectAll方法进行复用： 

```java
@Select("SELECT id,name FROM user")
@ResultMap("userMap")
public List<User> selectAll(); 
```



#### 3.4 @TypeDiscriminator、@Case注解

鉴别器注解基本没用，有空的时候再补充



#### 3.5 @Many、@One注解

**主要用于关联关系映射**

假设有作者(author)和文章(article)两张数据库表，一个author可以有多个article，一个article只能属于一个author。相关表结构以及初始数据如下所示： 

```sql
--author表
CREATE TABLE `author` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

INSERT INTO `author` (`id`, `name`) 
VALUES 
    (1, 'tianshouzhi');


--article表
CREATE TABLE `article` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `title` varchar(255) NOT NULL,
  `content` longtext NOT NULL,
  `author_id` int(11) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

INSERT INTO `article` (`id`, `title`, `content`, `author_id`)
VALUES
    (1, 'title1', 'content1', 1),
    (2, 'title2', 'content2', 1);
```



相应的Java实体类如下所示： 

```java
public class Article {

   private Integer id;

   private String title;

   private String content;

   private Author author;

   //...setters getters and toString...
}

public class Author {
   private Integer id;

   private String name;

   private List<Article> articles;
  
   //...setters getters and toString...
}
```



映射器接口定义分别如下： 

ArticleMapper 

```java
public interface ArticleMapper {
    //1、根据文章id查询文章Article对象，同时通过One注解关联查询出作者Author信息
    @Select("SELECT id,title,content,author_id FROM article where id= #{articleId}")
    @Results(id = "articleWithAuthor", value = {
            @Result(property = "id", column = "id"),
            @Result(property = "title", column = "title"),
            @Result(property = "content", column = "content"),
            //property属性：指定将关联查询的结果封装到Article对象的author属性上
            //column属性指定：指定在执行@One注解中定义的select语句时，把article表的author_id字段当做参数传入
            //one属性：通过@One注解定义关联查询的语句是AuthorMapper中的findAuthorByAuthorId方法
            @Result(property = "author",column = "author_id”,
                    one = @One(select = "com.tianshouzhi.mapper.AuthorMapper.findAuthorByAuthorId"))})
    public Article findArticleWithAuthorByArticleId(@Param("articleId") int articleId);
    
    //2、根据作者(Author)的id查询其所有的文章(Article)
   @Select("SELECT id,title,content,author_id FROM article WHERE author_id=#{authorId}")
   @Results(id = "articlesWithoutAuthor", value = {
           @Result(property = "id", column = "id"),
            @Result(property = "title", column = "title"),
            @Result(property = "content", column = "content")})
   List<Article> findArticlesByAuthorId(@Param("authorId") int authorId);
}
```

AuthorMapper 

```java
public interface AuthorMapper {
    //根据作者id查询Author信息
    @Select("SELECT id,name FROM author WHERE id=#{authorId}")
    Author findAuthorByAuthorId(int authorId);

    //根据作者id查询Author信息，通过@Many注解关联查询出所有的文章信息
    @Select("SELECT id,name FROM author WHERE id=#{authorId}")
    @Results(id = "authorWithArticles", value = {
            @Result(property = "id", column = "id"),
            @Result(property = "name", column = "name”),
           //property属性：指定将关联查询的结果封装到Author对象的articles属性上
            //column属性指定：指定在执行@Many注解中定义的select语句时，把author表的id字段当做参数传入
            //many属性：指定通过@Many注解定义关联查询的语句是ArticleMapper中的findArticlesByAuthorId方法
            @Result(property = "articles",column = "id”,
                    many = @Many(select = "com.tianshouzhi.mapper.ArticleMapper.findArticlesByAuthorId"))})
    Author findAuthorWithArticlesByAuthorId(int authorId);
}
```



测试如下： 

```java
@Test
public void testOneAndMany(){
       System.out.println("===========通过@One注解查询出Article关联的Auhtor===========");
       Article article = articleMapper.findArticleWithAuthorByArticleId(1);
       System.out.println(article);

       System.out.println("===========通过@Many注解查询出Auhtor关联的Article==========");
       Author author = authorMapper.findAuthorWithArticlesByAuthorId(1);
       System.out.println(author);
   }
```



控制台输出结果为： 

===========通过@One注解查询出Article关联的Auhtor===========

Article{id=1, title='title1', content='content1', author=Author{id=1, name='tianshouzhi', articles=null}}

===========通过@Many注解查询出Auhtor关联的Article==========

Author{id=1, name='tianshouzhi', articles=[Article{id=1, title='title1', content='content1', author=null}, Article{id=2, title='title2', content='content2', author=null}]}



### 4 缓存相关注解



| **注解**           | **使用对象** | **相对应的 XML** | **描述**                                                     |
| ------------------ | ------------ | ---------------- | ------------------------------------------------------------ |
| @CacheNamespace    | 类           | <cache>          | 为给定的命名空间（比如类）配置缓存。属性有：implemetation, eviction, flushInterval, size, readWrite, blocking 和properties。 |
| @Property          | N/A          | <property>       | 指定参数值或占位值（placeholder）（能被 mybatis-config.xml内的配置属性覆盖）。属性有：name, value。（仅在MyBatis 3.4.2以上版本生效） |
| @CacheNamespaceRef | 类           | <cacheRef>       | 参照另外一个命名空间的缓存来使用。属性有：value, name。如果你使用了这个注解，你应设置 value 或者 name 属性的其中一个。value 属性用于指定 Java 类型而指定命名空间（命名空间名就是指定的 Java 类型的全限定名），name 属性（这个属性仅在MyBatis 3.4.2以上版本生效）直接指定了命名空间的名字。 |
| @Flush             | 方法         | N/A              | 如果使用了这个注解，定义在 Mapper 接口中的方法能够调用 SqlSession#flushStatements() 方法。（Mybatis 3.3及以上） |