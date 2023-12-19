---
title : MYBATIS基本使用总结
categories : [java]
tags : [mysql]
cover: https://source.wjwsm.top/crown-4HWiHMDFN4c-unsplash.jpg
swiper_index: 4
date: 2023-03-03
---



# _MYBATIS_基本使用总结

最近在学java,主要他们都说springboot实在太好用了,就想着来体验下spring生态>_<



## 1. 使用maven导入mybatis相关依赖

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- plugins在配置文件中的位置必须符合要求，否则会报错，顺序如下: properties?, settings?, typeAliases?,
        typeHandlers?, objectFactory?,objectWrapperFactory?, plugins?, environments?,
        databaseIdProvider?, mappers? -->
    <!-- 配置mybatis的缓存，延迟加载等等一系列属性 -->

    <settings>
        <!-- 全局映射器启用缓存 -->
        <setting name="cacheEnabled" value="false"/>
        <!-- 对于未知的SQL查询，允许返回不同的结果集以达到通用的效果 -->
        <setting name="multipleResultSetsEnabled" value="true"/>
        <!-- 是否开启自动驼峰命名规则映射，数据库的A_COLUMN映射为Java中的aColumn -->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <!-- MyBatis利用本地缓存机制（Local Cache）防止循环引用（circular references）和加速重复嵌套查询 -->
        <setting name="localCacheScope" value="STATEMENT"/>
    </settings>
    <!-- 指定路径下的实体类支持别名(默认实体类的名称,首字母小写), @Alias注解可设置别名 -->
    <typeAliases>
        <package name="com.wxg.mapper"/>
    </typeAliases>
    <!-- 配置当前环境信息 -->
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC">
                <property name="" value=""/>
            </transactionManager>
            <!-- 配置数据源 -->
            <dataSource type=" POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
<!--                <property name="driverClassLoader" value="com.mysql.cj.jdbc.Driver"/>-->
            </dataSource>
        </environment>
    </environments>
    <!-- 指定Mapper接口的路径 -->
    <mappers>
    <package name="com.wxg"/>
    </mappers>
</configuration>

```

> 支持多数据库设置,对environments标签下的default进行设置,数据库相关连接设置也可以通过读取properties文件

感觉这种写xml的方式有点古老 ,下面是导入properties文件的xml

```xml
<properties resource="db.properties">

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

```xml
driver=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://127.0.0.1:3306/mybatis?useSSL=true&useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&failOverReadOnly=false&useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC
username=root
password=fzl-2000528
```

> 注意mapper标签下,可以使用package导入你要是用mapper,mybatis会自信扫描包下的mapper接口

## 2. 创建数据库实体类

比如我的数据库当中有张user表,下面在某个路径下创建JavaBean,比如com.java下

```java
 
public class User implements Serializable{
    private int userid;
    private String username;
    private String realname;
 
    public User() {
    }
 
    public User(String username, String realname) {
        this.username = username;
        this.realname = realname;
    }
 
    public User(int userid, String username, String realname) {
        this.userid = userid;
        this.username = username;
        this.realname = realname;
    }
 
    @Override
    public String toString() {
        return "User{" +
                "userid=" + userid +
                ", username='" + username + '\'' +
                ", realname='" + realname + '\'' +
                '}';
    }
 
    public int getUserid() {
        return userid;
    }
 
    public void setUserid(int userid) {
        this.userid = userid;
    }
 
    public String getUsername() {
        return username;
    }
 
    public void setUsername(String username) {
        this.username = username;
    }
 
    public String getRealname() {
        return realname;
    }
 
    public void setRealname(String realname) {
        this.realname = realname;
    }
}
```

## 3. 创建usermapper接口配置文件

在项目文件的resources下创建相同com.java,切记要使用/作为分隔符,即`com/java`,因为respurces不属于java package

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<!--
    mapper为映射的根节点，用来管理DAO接口
    namespace指定DAO接口的完整类名，表示mapper配置文件管理哪个DAO接口(包.接口名)
    mybatis会依据这个接口动态创建一个实现类去实现这个接口，而这个实现类是一个Mapper对象
 -->
<mapper namespace="包名.类名">
    <!--
        id = "接口中的方法名"
        #{} :表示占位符，等价于 ？ 这是mybatis框架的语法 
        parameterType = "接口中传入方法的参数类型"
        resultType = "返回实体类对象：包.类名"  处理结果集 自动封装
        注意:sql语句后不要出现";"号
    -->
    <!-- 查询  根据id查询用户信息
            select标签用于查询的标签
                id:标签的唯一标识
                resultType:定义返回的类型 把sql查询的结果封装到哪个实体类中
           #{} :表示占位符，等价于 ？ 这是mybatis框架的语法    
     -->
   <select id="selectOne" resultType="User">
        select * from tb_user where userid= #{userid}
    </select>
</mapper>
```

> 这个文件主要就是要记录这张表要对应的增删改查操作,后期可以直接通过注解开发,不过可以了解一下

## 4. 创建usermapper接口

在user类同目录下创建user mapper.interface,源码貌似使用代理 ,来实现的

```java
public interface usermapper {
    // @Select("SELECT * FROM tb_user WHERE id = 1")
    //@MapKey("id")
    List<Map<String,Object>> selectOne(@Param("ids") Integer id);
}
```

> 原始就是这样的,感觉写xml就会写废掉

之后就可以创建一个test类来验证

```java
public class test {
    @Test
    public void test2() throws IOException {
        InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        usermapper sql = sqlSession.getMapper(usermapper.class);
        System.out.println(sql.getUser(3));
    }}
```

![Snipaste_2023-03-02_22-34-18](https://source.wjwsm.top/mybatis-1.png)!](C:\Users\wxg\Pictures\Snipaste_2023-03-02_22-34-18.png)

mybatis初步入门就是这样,xml的一些语法,我这边就不介绍,我都觉得烦,下面来聊一聊如何通过注解开发,来避免写mapper.xml文件把

## 5. mybatis常用注解

### 1. 常用注解

| 注解            | 说明                                   |
| --------------- | -------------------------------------- |
| @Insert         | 实现新增                               |
| @Delete         | 实现删除                               |
| @Update         | 实现更新                               |
| @Select         | 实现查询                               |
| @Result         | 实现结果集封装                         |
| @Results        | 可以与@Result 一起使用，封装多个结果集 |
| @ResultMap      | 实现引用@Results 定义的封装            |
| @One            | 实现一对一结果集封装                   |
| @Many           | 实现一对多结果集封装                   |
| @SelectProvider | 实现动态 SQL 映射                      |
| @CacheNamespace | 实现注解二级缓存的使用                 |

那之前的接口就可以使用注解进行修改了

```java
public interface usermapper {
    @Select("SELECT * FROM tb_user WHERE id = 1")
    //@MapKey("id")
    List<Map<String,Object>> selectOne(@Param("ids") Integer id);
}
```

就可以省去很多麻烦,简直是起飞了

### 2. 实现连表,子查询的注解

1. **@Results 注解**

    > **代替的是标签`<resultMap>`
    > 该注解中可以使用单个@Result 注解，也可以使用@Result 集合
    > @Results（{@Result（），@Result（）}）或@Results（@Result（））**

```java
public class User implements Serializable{

    private Integer userId;
    private String userName;
    private String userAddress;
    private String userSex;
    private Date userBirthday;

    //一对多关系映射：一个用户对应多个账户
    private List<Account> accounts;

    public List<Account> getAccounts() {
        return accounts;
    }

    public void setAccounts(List<Account> accounts) {
        this.accounts = accounts;
    }

    public Integer getUserId() {
        return userId;
    }

    public void setUserId(Integer userId) {
        this.userId = userId;
    }

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public String getUserAddress() {
        return userAddress;
    }

    public void setUserAddress(String userAddress) {
        this.userAddress = userAddress;
    }

    public String getUserSex() {
        return userSex;
    }

    public void setUserSex(String userSex) {
        this.userSex = userSex;
    }

    public Date getUserBirthday() {
        return userBirthday;
    }

    public void setUserBirthday(Date userBirthday) {
        this.userBirthday = userBirthday;
    }

    @Override
    public String toString() {
        return "User{" +
                "userId=" + userId +
                ", userName='" + userName + '\'' +
                ", userAddress='" + userAddress + '\'' +
                ", userSex='" + userSex + '\'' +
                ", userBirthday=" + userBirthday +
                '}';
    }
}
```



```java
public class Account implements Serializable {

    private Integer id;
    private Integer uid;
    private Double money;

    //多对一（mybatis中称之为一对一）的映射：一个账户只能属于一个用户
    private User user;

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public Integer getUid() {
        return uid;
    }

    public void setUid(Integer uid) {
        this.uid = uid;
    }

    public Double getMoney() {
        return money;
    }

    public void setMoney(Double money) {
        this.money = money;
    }

    @Override
    public String toString() {
        return "Account{" +
                "id=" + id +
                ", uid=" + uid +
                ", money=" + money +
                '}';
    }
}

```

假设这里有两张表,一张用户表,一张账号表,一个用户可以拥有多个账号,但是一个账号只有一个用户,这就是一对多和多对一的关系.

如果我现在要想在查询账户的时候,顺便查出该账号所属的用户信息,因为这是两张表,一般我们可以用连表查询

```java
public interface IAccountDao {

    /**
     * 查询所有账户，并且获取每个账户所属的用户信息
     * @return
     */
    @Select("select * from account")
    @Results(id="accountMap",value = {
            @Result(id=true,column = "id",property = "id"),
            @Result(column = "uid",property = "uid"),
            @Result(column = "money",property = "money"),
            @Result(property = "user",column = "uid",one=@One(select="com.itheima.dao.IUserDao.findById",fetchType= FetchType.EAGER))
    })
    List<Account> findAll();

    /**
     * 根据用户id查询账户信息
     * @param userId
     * @return
     */
    @Select("select * from account where uid = #{userId}")
    List<Account> findAccountByUid(Integer userId);
}
3.3.2.3 添加用户的持久层接口并使用注解配置

@CacheNamespace(blocking = true)
public interface IUserDao {

    /**
     * 查询所有用户
     * @return
     */
    @Select("select * from user")
    @Results(id="userMap",value={
            @Result(id=true,column = "id",property = "userId"),
            @Result(column = "username",property = "userName"),
            @Result(column = "address",property = "userAddress"),
            @Result(column = "sex",property = "userSex"),
            @Result(column = "birthday",property = "userBirthday"),
            @Result(property = "accounts",column = "id",
                    many = @Many(select = "com.itheima.dao.IAccountDao.findAccountByUid",
                                fetchType = FetchType.LAZY))
    })
    List<User> findAll();

    /**
     * 根据id查询用户
     * @param userId
     * @return
     */
    @Select("select * from user  where id=#{id} ")
    @ResultMap("userMap")
    User findById(Integer userId);

    /**
     * 根据用户名称模糊查询
     * @param username
     * @return
     */
    @Select("select * from user where username like #{username} ")
    @ResultMap("userMap")
    List<User> findUserByName(String username);


}
```

>@Result(property = "user",column = "uid",one=@One(select="com.itheima.dao.IUserDao.findById",fetchType= FetchType.EAGER)

这个@Result注解中 user对应javabean中对应的user类,column表示连接的外键,@one表示一对一关系,fetchType表示如果是EAGER，那么表示取出这条数据时，它关联的数据也同时取出放入内存中如果是LAZY，那么取出这条数据时，它关联的数据并不取出来，在同一个session中，什么时候要用，就什么时候取(再次访问数据库)。但是，在session外，就不能再取了。用EAGER时，因为在内存里，所以在session外也可以取。

<font color ='red'>一般简单的sql可以直接通过注解,但是对于复杂的动态sql,都是写在xml文件当中,方便阅读和管理</font>

## 6.动态*SQL*

1. 注解开发

    ```java
    package Intefaceproxy.Dyno;
    
    import java.util.List;
    import java.util.Map;
    import org.apache.ibatis.annotations.SelectProvider;
    import model.Employee;
    
    public interface EmployeeMapper {
        //动态查询  type:指定一个类    method:使用这个类中的selectWhitParamSql方法返回的sql字符串  作为查询的语句
        @SelectProvider(type=Intefaceproxy.Dyno.EmployeeDynaSqlProvider.class,method="selectWhitParamSql")
        List<Employee> selectWithParam(Map<String,Object> param);
    }
    ```

    

```java
import java.util.Map;

import org.apache.ibatis.jdbc.SQL;

public class EmployeeDynaSqlProvider {
    //方法中的关键字是区分大小写的  SQL SELECT WHERE
    //该方法会根据传递过来的map中的参数内容  动态构建sql语句
    public String selectWhitParamSql(Map<String, Object> param) {
        return new SQL() {
            {
                SELECT("*");
                FROM("tb_employee");
                if (param.get("id")!=null) {
                    WHERE("id=#{id}");
                }
                if(param.get("loginname")!=null) {
                    WHERE("loginname=#{loginname}");
                }
                if(param.get("password")!=null) {
                    WHERE("password=#{password}");
                }
                if(param.get("name")!=null) {
                    WHERE("name=#{name}");
                }
                if(param.get("sex")!=null) {
                    WHERE("sex=#{sex}");
                }
                if(param.get("age")!=null) {
                    WHERE("age=#{age}");
                }
                if(param.get("phone")!=null) {
                    WHERE("phone=#{phone}");
                }
                if(param.get("sal")!=null) {
                    WHERE("sal=#{sal}");
                }
                if(param.get("state")!=null) {
                    WHERE("state=#{state}");
                }
            }
            
        }.toString();
    }
}
```

这边也可以换成xml,在mapper配置文件中使用

```xml
<select id="selectWhitParamSql" resultType="Employee">
    
     SELECT *  FROM  tb_employee;
    <where>
        <if test="sex != null">
            sex = #{sex} 
        </if>
        <if test="phone!= null">
           phone = #{phone}
        </if>
    </where>
</select>
```

也可以通过<trim>标签来进行前缀和后缀的设置

> 我自己写过的业务 也做多涉及三四张表,没写过十几张表的联合查询,我想哪个xml就emo了
