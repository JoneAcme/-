# JDBC相关



# JDBC 相关

[TOC]

## JDBC基础实现

```java
//1. 导入jar
//2. 注册驱动
Class.forName("com.mysql.jdbc.Driver")
//3. 获取数据库连接对象
Connection conn = DriverManager.getConnection() 
//4. 获取执行sql 语句的连接
String sql = "update ..."
Statement stmt = conn.createStatement();
//5. 执行sql
int count = stmt.excuteUpdate(sql)
//6. 处理结果
  System.out.println(count)
//7. 释放资源
stmt.close();
conn.close();
```

### Connection

1. 执行sql 的对象
   1. createStatement()
   2. prepareStatement()
2. 管理事务
    1. 开启事务：setAutoCommite(Boolean autoCommit) 方法参数为false 则为开始事务
    2. 提交事务：commit() 
    3. 回滚事务：rollback()

### Statement

    1. boolean execute(String sql) ：执行任意sql
    2. int executeUpdate(String sql)：执行DML（insert、update、delete）、DDL(create、drop、alter)  返回影响的行数
    3. ResultSet executeQuery()：执行DQL(select)

### ResultSet

1. boolean next() 游标移动到下一行  返回值：是否还有下一行
2. getXXX() 获取数据

### PreparedStatement 

1. sql 注入问题

   例：

   用户名随意，密码 `'a' or  'a' ='a'`

   最后执行的sql 是`select * from user where username="" and password = 'a' or  'a' ='a'`

   其中 `'a' ='a'`是恒等式，故会查询出所有信息

PreparedStatement解决sql注入问题，sql预编译，所有参数用"?"占位符代替，最后执行之前给占位符赋值。

**使用PreparedStatement的原因：**

1. **方式SQL注入**
2. **更高效**

## JDBC工具类

以下加载文件中的参数可用静态代码块：

1. src下创建.properties文件
2. 在代码中使用Properties 类 load() 
3. 从加载的勒种获取url、user、password 

> 动态配置资源获取：
>
> 1. 类名.class.getClassLoader()获取到类加载器
> 2. 使用classLoader.getResource()    此方法是以src为根目录，返回时为一个url
> 3. 通过url获取到文件的绝对路径
> 4. 使用Properties.load()  加载配置文件



## 数据库连接池

**概述**:存放数据库连接的容器

**优势**：

    1. 节约资源
    2. 访问高效

**实现**：

   实现标准接口`javax.sql.DataSource`

   使用接口中的`getConnection()`获取连接(*如果是通过这种方式获取的连接，最后在close()后，不会关闭，而是归还给连接池*)

   一般是数据库厂商来实现：

   1. C3P0

   2. Druid(阿里巴巴)



**使用**：

- C3P0

  1. 导入jar

  2. 定义配置文件 

     名称：必须为 c3p0.properties 或者 c3p0.config.xml

     路径：src目录下

  3. 创建数据库连接池对象 `ComboPooledDataSource` (DataSource的实现类)

     说明：同一个文件下可创建多个Config，默认是用默认配置，在获取连接池对象时，可使用指定名称配置即(`ComboPooledDataSource(String configName)`)

  4. `getConnection()`获取连接

- Druid(阿里巴巴)

  1. 导入jar

  2. 定义配置文件

     名称：是properties 文件形式，任意名称、任意目录

     ```properties
     driverClassName=com.mysql.jdbc.Driver
     url = jdbc:mysql://127.0.0.1:3306/db
     username=root
     password = rootroot
     initialSize = 5
     maxActive = 10
     maxWait = 300
     ```

     

  3. 通过`DruiDataSourceFactory`获取数据库连接池对象

     ```java
     Properties pro = new Properties();
     InputStream is = [类名].class.getClassLoader().getResourceAsStream([properties文件路径]);
     pro.load(is);
     DataSource ds = DruiDataSourceFactory.createDataSource(pro);
     Connection conn = ds.getConnection();
     ```

     

  此处最好提供一个Util类，提供以下方法：

    1. 静态代码块初始化数据库连接池
    2. 获取数据库连接池连接
    3. 关闭、回收资源、连接归还给连接池

   

## Spring JDBC

**JDBCTemplate**：Spring 对JDBC的简单封装

1. 导入jar

2. 创建JDBCTemplate，依赖于DataSource

   `JdbcTemplate template = new JdbcTemplate(dataSource);`

3. 调用JDBCTemplate 完成CRUD

   - `update()`：指定DML语句，增删改

   - `queryForMap()`：查询结果将被封装为map，列名为key，值为value。

     注意：这个方法查询的结果集长度只能是1

   - `queryForList()`：查询结果将被封装为List

     每一条记录为map，然后将map放入List中，即List<Map<列名,值>>的结构

   - `query()` 查询结果，将结果封装为JavaBean

     参数：RowMapper接口：将查询到的数据转换为指定的JavaBean对象

   - `queryForObject()`：将结果封装为对象，一般用于聚合函数的查询