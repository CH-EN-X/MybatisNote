

# 1. Mybatis基础操作

学习完mybatis入门后，我们继续学习mybatis基础操作。

## 1.1 需求

需求说明：

- 根据资料中提供的《tlias智能学习辅助系统》页面原型及需求，完成员工管理的需求开发。

![image-20221210180155700](assets/image-20221210180155700.png) 

![image-20221210180343288](assets/image-20221210180343288.png)

![image-20221210180515206](assets/image-20221210180515206.png)

通过分析以上的页面原型和需求，我们确定了功能列表：

1. 查询
   - 根据主键ID查询
   - 条件查询

2. 新增
3. 更新
4. 删除
   - 根据主键ID删除
   - 根据主键ID批量删除





## 1.2 准备

实施前的准备工作：

1. 准备数据库表
2. 创建一个新的springboot工程，选择引入对应的起步依赖（mybatis、mysql驱动、lombok）
3. application.properties中引入数据库连接信息
4. 创建对应的实体类 Emp（实体类属性采用驼峰命名）
5. 准备Mapper接口 EmpMapper



**准备数据库表**

~~~mysql
-- 部门管理
create table dept
(
    id          int unsigned primary key auto_increment comment '主键ID',
    name        varchar(10) not null unique comment '部门名称',
    create_time datetime    not null comment '创建时间',
    update_time datetime    not null comment '修改时间'
) comment '部门表';
-- 部门表测试数据
insert into dept (id, name, create_time, update_time)
values (1, '学工部', now(), now()),
       (2, '教研部', now(), now()),
       (3, '咨询部', now(), now()),
       (4, '就业部', now(), now()),
       (5, '人事部', now(), now());


-- 员工管理
create table emp
(
    id          int unsigned primary key auto_increment comment 'ID',
    username    varchar(20)      not null unique comment '用户名',
    password    varchar(32) default '123456' comment '密码',
    name        varchar(10)      not null comment '姓名',
    gender      tinyint unsigned not null comment '性别, 说明: 1 男, 2 女',
    image       varchar(300) comment '图像',
    job         tinyint unsigned comment '职位, 说明: 1 班主任,2 讲师, 3 学工主管, 4 教研主管, 5 咨询师',
    entrydate   date comment '入职时间',
    dept_id     int unsigned comment '部门ID',
    create_time datetime         not null comment '创建时间',
    update_time datetime         not null comment '修改时间'
) comment '员工表';
-- 员工表测试数据
INSERT INTO emp (id, username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time)
VALUES 
(1, 'jinyong', '123456', '金庸', 1, '1.jpg', 4, '2000-01-01', 2, now(), now()),
(2, 'zhangwuji', '123456', '张无忌', 1, '2.jpg', 2, '2015-01-01', 2, now(), now()),
(3, 'yangxiao', '123456', '杨逍', 1, '3.jpg', 2, '2008-05-01', 2, now(), now()),
(4, 'weiyixiao', '123456', '韦一笑', 1, '4.jpg', 2, '2007-01-01', 2, now(), now()),
(5, 'changyuchun', '123456', '常遇春', 1, '5.jpg', 2, '2012-12-05', 2, now(), now()),
(6, 'xiaozhao', '123456', '小昭', 2, '6.jpg', 3, '2013-09-05', 1, now(), now()),
(7, 'jixiaofu', '123456', '纪晓芙', 2, '7.jpg', 1, '2005-08-01', 1, now(), now()),
(8, 'zhouzhiruo', '123456', '周芷若', 2, '8.jpg', 1, '2014-11-09', 1, now(), now()),
(9, 'dingminjun', '123456', '丁敏君', 2, '9.jpg', 1, '2011-03-11', 1, now(), now()),
(10, 'zhaomin', '123456', '赵敏', 2, '10.jpg', 1, '2013-09-05', 1, now(), now()),
(11, 'luzhangke', '123456', '鹿杖客', 1, '11.jpg', 5, '2007-02-01', 3, now(), now()),
(12, 'hebiweng', '123456', '鹤笔翁', 1, '12.jpg', 5, '2008-08-18', 3, now(), now()),
(13, 'fangdongbai', '123456', '方东白', 1, '13.jpg', 5, '2012-11-01', 3, now(), now()),
(14, 'zhangsanfeng', '123456', '张三丰', 1, '14.jpg', 2, '2002-08-01', 2, now(), now()),
(15, 'yulianzhou', '123456', '俞莲舟', 1, '15.jpg', 2, '2011-05-01', 2, now(), now()),
(16, 'songyuanqiao', '123456', '宋远桥', 1, '16.jpg', 2, '2010-01-01', 2, now(), now()),
(17, 'chenyouliang', '123456', '陈友谅', 1, '17.jpg', NULL, '2015-03-21', NULL, now(), now());
~~~



**创建一个新的springboot工程，选择引入对应的起步依赖（mybatis、mysql驱动、lombok）**

![image-20221210182008131](assets/image-20221210182008131.png)



**application.properties中引入数据库连接信息**

> 提示：可以把之前项目中已有的配置信息复制过来即可

~~~properties
#驱动类名称
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
#数据库连接的url
spring.datasource.url=jdbc:mysql://localhost:3306/mybatis
#连接数据库的用户名
spring.datasource.username=root
#连接数据库的密码
spring.datasource.password=1234
~~~



**创建对应的实体类Emp（实体类属性采用驼峰命名）**

~~~java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Emp {
    private Integer id;
    private String username;
    private String password;
    private String name;
    private Short gender;
    private String image;
    private Short job;
    private LocalDate entrydate;     //LocalDate类型对应数据表中的date类型
    private Integer deptId;
    private LocalDateTime createTime;//LocalDateTime类型对应数据表中的datetime类型
    private LocalDateTime updateTime;
}
~~~



**准备Mapper接口：EmpMapper**

~~~java
/*@Mapper注解：表示当前接口为mybatis中的Mapper接口
  程序运行时会自动创建接口的实现类对象(代理对象)，并交给Spring的IOC容器管理
*/
@Mapper
public interface EmpMapper {

}
~~~

完成以上操作后，项目工程结构目录如下：

![image-20221210182500817](assets/image-20221210182500817.png)





## 1.3 删除

### 1.3.1 功能实现

页面原型：

![image-20221210183336095](assets/image-20221210183336095.png)

> 当我们点击后面的"删除"按钮时，前端页面会给服务端传递一个参数，也就是该行数据的ID。 我们接收到ID后，根据ID删除数据即可。



**功能：根据主键删除数据**

- SQL语句

~~~mysql
-- 删除id=17的数据
delete from emp where id = 17;
~~~

> Mybatis框架让程序员更关注于SQL语句

- 接口方法

~~~java
@Mapper
public interface EmpMapper {
    
    //@Delete("delete from emp where id = 17")
    //public void delete();
    //以上delete操作的SQL语句中的id值写成固定的17，就表示只能删除id=17的用户数据
    //SQL语句中的id值不能写成固定数值，需要变为动态的数值
    //解决方案：在delete方法中添加一个参数(用户id)，将方法中的参数，传给SQL语句
    
    /**
     * 根据id删除数据
     * @param id    用户id
     */
    @Delete("delete from emp where id = #{id}")//使用#{key}方式获取方法中的参数值
    public void delete(Integer id);
    
}
~~~

> @Delete注解：用于编写delete操作的SQL语句

> 如果mapper接口方法形参只有一个普通类型的参数，#{…} 里面的属性名可以随便写，如：#{id}、#{value}。但是建议保持名字一致。

- 测试
  - 在单元测试类中通过@Autowired注解注入EmpMapper类型对象

~~~java
@SpringBootTest
class SpringbootMybatisCrudApplicationTests {
    @Autowired //从Spring的IOC容器中，获取类型是EmpMapper的对象并注入
    private EmpMapper empMapper;

    @Test
    public void testDel(){
        //调用删除方法
        empMapper.delete(16);
    }

}
~~~





### 1.3.2 日志输入

在Mybatis当中我们可以借助日志，查看到sql语句的执行、执行传递的参数以及执行结果。具体操作如下：

1. 打开application.properties文件

2. 开启mybatis的日志，并指定输出到控制台

```properties
#指定mybatis输出日志的位置, 输出控制台
mybatis.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
```

开启日志之后，我们再次运行单元测试，可以看到在控制台中，输出了以下的SQL语句信息：

![image-20220901164225644](assets/image-20220901164225644.png) 

> 但是我们发现输出的SQL语句：delete from emp where id = ?，我们输入的参数16并没有在后面拼接，id的值是使用?进行占位。那这种SQL语句我们称为预编译SQL。





### 1.3.3 预编译SQL

#### 1.3.3.1 介绍

预编译SQL有两个优势：

1. 性能更高
2. 更安全(防止SQL注入)

![image-20221210202222206](assets/image-20221210202222206.png)

> 性能更高：预编译SQL，编译一次之后会将编译后的SQL语句缓存起来，后面再次执行这条语句时，不会再次编译。（只是输入的参数不同）
>
> 更安全(防止SQL注入)：将敏感字进行转义，保障SQL的安全性。



#### 1.3.3.2 SQL注入

SQL注入：是通过操作输入的数据来修改事先定义好的SQL语句，以达到执行代码对服务器进行攻击的方法。

> 由于没有对用户输入进行充分检查，而SQL又是拼接而成，在用户输入参数时，在参数中添加一些SQL关键字，达到改变SQL运行结果的目的，也可以完成恶意攻击。

**测试1：使用资料中提供的程序，来验证SQL注入问题**

![image-20221210205419634](assets/image-20221210205419634.png)

第1步：进入到DOS

![image-20221211124744203](assets/image-20221211124744203.png)

![image-20221211124840720](assets/image-20221211124840720.png)

第2步：执行以下命令，启动程序

~~~powershell
#启动存在SQL注入的程序
java -jar sql_Injection_demo-0.0.1-SNAPSHOT.jar 
~~~

![image-20221210211605231](assets/image-20221210211605231.png)

第3步：打开浏览器输入`http://localhost:9090/login.html`

![image-20221210212406527](assets/image-20221210212406527.png)

发现竟然能够登录成功：

![image-20221210212511915](assets/image-20221210212511915.png)



以上操作为什么能够登录成功呢？

- 由于没有对用户输入内容进行充分检查，而SQL又是字符串拼接方式而成，在用户输入参数时，在参数中添加一些SQL关键字，达到改变SQL运行结果的目的，从而完成恶意攻击。

![image-20221210213311518](assets/image-20221210213311518.png)

> ![image-20221210214431228](assets/image-20221210214431228.png)
>
> 用户在页面提交数据的时候人为的添加一些特殊字符，使得sql语句的结构发生了变化，最终可以在没有用户名或者密码的情况下进行登录。



**测试2：使用资料中提供的程序，来验证SQL注入问题**

第1步：进入到DOS

第2步：执行以下命令，启动程序：

~~~powershell
#启动解决了SQL注入的程序
java -jar sql_prepared_demo-0.0.1-SNAPSHOT.jar
~~~

第3步：打开浏览器输入`http://localhost:9090/login.html`

![image-20221210212406527](assets/image-20221210212406527.png)

发现无法登录：

![image-20221211125751981](assets/image-20221211125751981.png)

以上操作SQL语句的执行：

![image-20221211130011973](assets/image-20221211130011973.png)

> 把整个`' or '1'='1`作为一个完整的参数，赋值给第2个问号（`' or '1'='1`进行了转义，只当做字符串使用）



#### 1.3.3.3 参数占位符

在Mybatis中提供的参数占位符有两种：${...} 、#{...}

- #{...}
  - 执行SQL时，会将#{…}替换为?，生成预编译SQL，会自动设置参数值
  - 使用时机：参数传递，都使用#{…}

- ${...}
  - 拼接SQL。直接将参数拼接在SQL语句中，存在SQL注入问题
  - 使用时机：如果对表名、列表进行动态设置时使用

> 注意事项：在项目开发中，建议使用#{...}，生成预编译SQL，防止SQL注入安全。



 



## 1.4 新增

功能：新增员工信息

![image-20221211134239610](assets/image-20221211134239610.png)

### 1.4.1 基本新增

员工表结构：

![image-20221211134746319](assets/image-20221211134746319.png)

SQL语句：

```sql
insert into emp(username, name, gender, image, job, entrydate, dept_id, create_time, update_time) values ('songyuanqiao','宋远桥',1,'1.jpg',2,'2012-10-09',2,'2022-10-01 10:00:00','2022-10-01 10:00:00');
```

接口方法：

```java
@Mapper
public interface EmpMapper {

    @Insert("insert into emp(username, name, gender, image, job, entrydate, dept_id, create_time, update_time) values (#{username}, #{name}, #{gender}, #{image}, #{job}, #{entrydate}, #{deptId}, #{createTime}, #{updateTime})")
    public void insert(Emp emp);

}
```

> 说明：#{...} 里面写的名称是对象的属性名

测试类：

```java
import com.itheima.mapper.EmpMapper;
import com.itheima.pojo.Emp;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import java.time.LocalDate;
import java.time.LocalDateTime;

@SpringBootTest
class SpringbootMybatisCrudApplicationTests {
    @Autowired
    private EmpMapper empMapper;

    @Test
    public void testInsert(){
        //创建员工对象
        Emp emp = new Emp();
        emp.setUsername("tom");
        emp.setName("汤姆");
        emp.setImage("1.jpg");
        emp.setGender((short)1);
        emp.setJob((short)1);
        emp.setEntrydate(LocalDate.of(2000,1,1));
        emp.setCreateTime(LocalDateTime.now());
        emp.setUpdateTime(LocalDateTime.now());
        emp.setDeptId(1);
        //调用添加方法
        empMapper.insert(emp);
    }
}

```

> 日志输出：
>
> ![image-20221211140222240](assets/image-20221211140222240.png)







### 1.4.2 主键返回

概念：在数据添加成功后，需要获取插入数据库数据的主键。

> 如：添加套餐数据时，还需要维护套餐菜品关系表数据。
>
> ![image-20221211150353385](assets/image-20221211150353385.png)
>
> 业务场景：在前面讲解到的苍穹外卖菜品与套餐模块的表结构，菜品与套餐是多对多的关系，一个套餐对应多个菜品。既然是多对多的关系，是不是有一张套餐菜品中间表来维护它们之间的关系。
>
> ![image-20221212093655389](assets/image-20221212093655389.png)
>
> 在添加套餐的时候，我们需要在界面当中来录入套餐的基本信息，还需要来录入套餐与菜品的关联信息。这些信息录入完毕之后，我们一点保存，就需要将套餐的信息以及套餐与菜品的关联信息都需要保存到数据库当中。其实具体的过程包括两步，首先第一步先需要将套餐的基本信息保存了，接下来第二步再来保存套餐与菜品的关联信息。套餐与菜品的关联信息就是往中间表当中来插入数据，来维护它们之间的关系。而中间表当中有两个外键字段，一个是菜品的ID，就是当前菜品的ID，还有一个就是套餐的ID，而这个套餐的 ID 指的就是此次我所添加的套餐的ID，所以我们在第一步保存完套餐的基本信息之后，就需要将套餐的主键值返回来供第二步进行使用。这个时候就需要用到主键返回功能。

那要如何实现在插入数据之后返回所插入行的主键值呢？

- 默认情况下，执行插入操作时，是不会主键值返回的。如果我们想要拿到主键值，需要在Mapper接口中的方法上添加一个Options注解，并在注解中指定属性useGeneratedKeys=true和keyProperty="实体类属性名"



主键返回代码实现：

~~~java
@Mapper
public interface EmpMapper {
    
    //会自动将生成的主键值，赋值给emp对象的id属性
    @Options(useGeneratedKeys = true,keyProperty = "id")
    @Insert("insert into emp(username, name, gender, image, job, entrydate, dept_id, create_time, update_time) values (#{username}, #{name}, #{gender}, #{image}, #{job}, #{entrydate}, #{deptId}, #{createTime}, #{updateTime})")
    public void insert(Emp emp);

}
~~~

测试：

~~~java
@SpringBootTest
class SpringbootMybatisCrudApplicationTests {
    @Autowired
    private EmpMapper empMapper;

    @Test
    public void testInsert(){
        //创建员工对象
        Emp emp = new Emp();
        emp.setUsername("jack");
        emp.setName("杰克");
        emp.setImage("1.jpg");
        emp.setGender((short)1);
        emp.setJob((short)1);
        emp.setEntrydate(LocalDate.of(2000,1,1));
        emp.setCreateTime(LocalDateTime.now());
        emp.setUpdateTime(LocalDateTime.now());
        emp.setDeptId(1);
        //调用添加方法
        empMapper.insert(emp);

        System.out.println(emp.getDeptId());
    }
}
~~~







## 1.5 更新

功能：修改员工信息

![image-20221212095605863](assets/image-20221212095605863.png)

> 点击"编辑"按钮后，会查询所在行记录的员工信息，并把员工信息回显在修改员工的窗体上(下个知识点学习)
>
> 在修改员工的窗体上，可以修改的员工数据：用户名、员工姓名、性别、图像、职位、入职日期、归属部门
>
> 思考：在修改员工数据时，要以什么做为条件呢？
>
> 答案：员工id

SQL语句：

```sql
update emp set username = 'linghushaoxia', name = '令狐少侠', gender = 1 , image = '1.jpg' , job = 2, entrydate = '2012-01-01', dept_id = 2, update_time = '2022-10-01 12:12:12' where id = 18;
```

接口方法：

```java
@Mapper
public interface EmpMapper {
    /**
     * 根据id修改员工信息
     * @param emp
     */
    @Update("update emp set username=#{username}, name=#{name}, gender=#{gender}, image=#{image}, job=#{job}, entrydate=#{entrydate}, dept_id=#{deptId}, update_time=#{updateTime} where id=#{id}")
    public void update(Emp emp);
    
}
```

测试类：

```java
@SpringBootTest
class SpringbootMybatisCrudApplicationTests {
    @Autowired
    private EmpMapper empMapper;

    @Test
    public void testUpdate(){
        //要修改的员工信息
        Emp emp = new Emp();
        emp.setId(23);
        emp.setUsername("songdaxia");
        emp.setPassword(null);
        emp.setName("老宋");
        emp.setImage("2.jpg");
        emp.setGender((short)1);
        emp.setJob((short)2);
        emp.setEntrydate(LocalDate.of(2012,1,1));
        emp.setCreateTime(null);
        emp.setUpdateTime(LocalDateTime.now());
        emp.setDeptId(2);
        //调用方法，修改员工数据
        empMapper.update(emp);
    }
}
```





## 1.6 查询

### 1.6.1 根据ID查询

在员工管理的页面中，当我们进行更新数据时，会点击 “编辑” 按钮，然后此时会发送一个请求到服务端，会根据Id查询该员工信息，并将员工数据回显在页面上。

![image-20221212101331292](assets/image-20221212101331292.png) 

SQL语句：

~~~mysql
select id, username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time from emp;
~~~

接口方法：

~~~java
@Mapper
public interface EmpMapper {
    @Select("select id, username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time from emp where id=#{id}")
    public Emp getById(Integer id);
}
~~~

测试类：

~~~java
@SpringBootTest
class SpringbootMybatisCrudApplicationTests {
    @Autowired
    private EmpMapper empMapper;

    @Test
    public void testGetById(){
        Emp emp = empMapper.getById(1);
        System.out.println(emp);
    }
}
~~~

> 执行结果：
>
> ![image-20221212103004961](assets/image-20221212103004961.png)
>
> 而在测试的过程中，我们会发现有几个字段(deptId、createTime、updateTime)是没有数据值的





### 1.6.2 数据封装

我们看到查询返回的结果中大部分字段是有值的，但是deptId，createTime，updateTime这几个字段是没有值的，而数据库中是有对应的字段值的，这是为什么呢？

![image-20221212103124490](assets/image-20221212103124490.png)

原因如下： 

- 实体类属性名和数据库表查询返回的字段名一致，mybatis会自动封装。
- 如果实体类属性名和数据库表查询返回的字段名不一致，不能自动封装。



 解决方案：

1. 起别名
2. 结果映射
3. 开启驼峰命名



**起别名**：在SQL语句中，对不一样的列名起别名，别名和实体类属性名一样

```java
@Select("select id, username, password, name, gender, image, job, entrydate, " +
        "dept_id AS deptId, create_time AS createTime, update_time AS updateTime " +
        "from emp " +
        "where id=#{id}")
public Emp getById(Integer id);
```

> 再次执行测试类：
>
> ![image-20221212111027396](assets/image-20221212111027396.png)



**手动结果映射**：通过 @Results及@Result 进行手动结果映射

```java
@Results({@Result(column = "dept_id", property = "deptId"),
          @Result(column = "create_time", property = "createTime"),
          @Result(column = "update_time", property = "updateTime")})
@Select("select id, username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time from emp where id=#{id}")
public Emp getById(Integer id);
```

> @Results源代码：
>
> ~~~java
> @Documented
> @Retention(RetentionPolicy.RUNTIME)
> @Target({ElementType.METHOD})
> public @interface Results {
>  String id() default "";
> 
>  Result[] value() default {};  //Result类型的数组
> }
> ~~~
>
> @Result源代码：
>
> ~~~java
> @Documented
> @Retention(RetentionPolicy.RUNTIME)
> @Target({ElementType.METHOD})
> @Repeatable(Results.class)
> public @interface Result {
>  boolean id() default false;//表示当前列是否为主键（true:是主键）
> 
>  String column() default "";//指定表中字段名
> 
>  String property() default "";//指定类中属性名
> 
>  Class<?> javaType() default void.class;
> 
>  JdbcType jdbcType() default JdbcType.UNDEFINED;
> 
>  Class<? extends TypeHandler> typeHandler() default UnknownTypeHandler.class;
> 
>  One one() default @One;
> 
>  Many many() default @Many;
> }
> ~~~



**开启驼峰命名(推荐)**：如果字段名与属性名符合驼峰命名规则，mybatis会自动通过驼峰命名规则映射

> 驼峰命名规则：   abc_xyz    =>   abcXyz
>
> - 表中字段名：abc_xyz
> - 类中属性名：abcXyz

```properties
# 在application.properties中添加：
mybatis.configuration.map-underscore-to-camel-case=true
```

> 要使用驼峰命名前提是 实体类的属性 与 数据库表中的字段名严格遵守驼峰命名。





### 1.6.3 条件查询

在员工管理的列表页面中，我们需要根据条件查询员工信息，查询条件包括：姓名、性别、入职时间。 

![image-20221212113422924](assets/image-20221212113422924.png)

通过页面原型以及需求描述我们要实现的查询：

- 姓名：要求支持模糊匹配
- 性别：要求精确匹配
- 入职时间：要求进行范围查询
- 根据最后修改时间进行降序排序



SQL语句：

```sql
select id, username, password, name, gender, image, job, entrydate, dept_id, create_time, update_time 
from emp 
where name like '%张%' 
      and gender = 1 
      and entrydate between '2010-01-01' and '2020-01-01 ' 
order by update_time desc;
```

接口方法：

- 方式一

```java
@Mapper
public interface EmpMapper {
    @Select("select * from emp " +
            "where name like '%${name}%' " +
            "and gender = #{gender} " +
            "and entrydate between #{begin} and #{end} " +
            "order by update_time desc")
    public List<Emp> list(String name, Short gender, LocalDate begin, LocalDate end);
}
```

> ![image-20221212115149151](assets/image-20221212115149151.png)
>
> 以上方式注意事项：
>
> 1. 方法中的形参名和SQL语句中的参数占位符名保持一致
>
> 2. 模糊查询使用${...}进行字符串拼接，这种方式呢，由于是字符串拼接，并不是预编译的形式，所以效率不高、且存在sql注入风险。



- 方式二（解决SQL注入风险）
  - 使用MySQL提供的字符串拼接函数：concat('%' , '关键字' , '%')

~~~java
@Mapper
public interface EmpMapper {

    @Select("select * from emp " +
            "where name like concat('%',#{name},'%') " +
            "and gender = #{gender} " +
            "and entrydate between #{begin} and #{end} " +
            "order by update_time desc")
    public List<Emp> list(String name, Short gender, LocalDate begin, LocalDate end);

}

~~~

> 执行结果：生成的SQL都是预编译的SQL语句（性能高、安全）
>
> ![image-20221212120006242](assets/image-20221212120006242.png)





### 1.6.4 参数名说明

在上面我们所编写的条件查询功能中，我们需要保证接口中方法的形参名和SQL语句中的参数占位符名相同。

> 当方法中的形参名和SQL语句中的占位符参数名不相同时，就会出现以下问题：
>
> ![image-20221212150611796](assets/image-20221212150611796.png)



参数名在不同的SpringBoot版本中，处理方案还不同：

- 在springBoot的2.x版本（保证参数名一致）

![image-20221212151156273](assets/image-20221212151156273.png)

> springBoot的父工程对compiler编译插件进行了默认的参数parameters配置，使得在编译时，会在生成的字节码文件中保留原方法形参的名称，所以#{…}里面可以直接通过形参名获取对应的值
>
> ![image-20221212151411154](assets/image-20221212151411154.png)



- 在springBoot的1.x版本/单独使用mybatis（使用@Param注解来指定SQL语句中的参数名）

![image-20221212151628715](assets/image-20221212151628715.png)

> 在编译时，生成的字节码文件当中，不会保留Mapper接口中方法的形参名称，而是使用var1、var2、...这样的形参名字，此时要获取参数值时，就要通过@Param注解来指定SQL语句中的参数名
>
> ![image-20221212151736274](assets/image-20221212151736274.png)











