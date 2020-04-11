---
title: JDBC复习（全面）
date: 2019-10-18 19:28:14
categories: JDBC
tags: 
- JDBC
- MySQL
---

## JDBC概述

### JDBC简介

- JDBC全称：Java DataBase Connectivity（java数据库连接）
- JDBC是SUN公司为了简化、统一对数据库的操作，定义了一套Java操作数据库的规范。
- JDBC是一组专门负责连接并操作数据库的标准，在整个JDBC中提供了大量的接口，由数据库厂商提供，不同数据库的JDBC驱动程序不同。
- JDBC与数据库驱动直接的关系：接口与实现的关系。

<!-- more -->

### JDBC操作的基本步骤

1. 加载数据库驱动程序，加载的时候需要将驱动程序配置到classpath中
2. 连接数据库，通过Connection接口和DriverManager类完成
3. 操作数据库，通过Statement，PreparedStatement、ResultSet三个接口完成
4. 关闭数据库，在实际开发中，数据库资源非常有限，操作完之后必须关闭。

## JDBC的一个类和三个接口

### java.sql.DriverManager类（注册驱动和创建连接）

1. 注册驱动

- 法1

```java
DriverManager.registerDriver(new com.mysql.jdbc.Driver());
//在mysql8.0版本以上： 
DriverManager.registerDriver(new com.mysql.cj.jdbc.Driver());
```
这种我们不推荐使用：一是导致驱动被注册两次，二是强烈依赖数据库的驱动jar包
com.mysql.jdbc.Driver类中有注册驱动的静态代码块，其中new Driver()会实例化一个当前的驱动

- 法2

```java
Class.forName("com.mysql.jdbc.Driver");  
//在mysql8.0版本以上： 
Class.forName("com.mysql.cj.jdbc.Driver");
```
第二种是用Class.forName("类名")的，向虚拟机实例化一个Class实例，在com.mysql.jdbc.Driver类中有注册驱动的静态代码块，只要执行这条语句，就完成了驱动的注册。

2. 与数据库建立连接

> URL:SUN公司与数据库厂商之间的一种协议，下面是一个URL的对应解释
> jdbc:mysql://localhost:3306/Test 　　　　　　
> 协议:子协议://     IP       :端口号/数据库名称

- 法1

```java
static Connection getConnection(String url)

DriverManager.getConnection("jdbc:mysql://localhost:3306/test?user=root&password=root");
//mysql8.0版本以上：
DriverManager.getConnection("jdbc:mysql://localhost:3306/stus?useSSL=false&serverTimezone=UTC&user=root&password=123456");
```

- 法2

```java
static Connection getConnection(String url, String user, String password)
DriverManager.getConnection("jdbc:mysql://localhost:3306/test", "root", "root");
//mysql8.0版本以上：
Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/stus?useSSL=false&serverTimezone=UTC", "root", "123456");
```

### java.sql.Connection接口（一个连接）

接口的实现在数据库驱动中，所有与数据库交互都是基于连接对象的。

```java
Statement  createStatement(); //创建操作sql语句的对象
PreparedStatement  prepareStatement(sql);
```

### java.sql.Statement接口:（操作sql语句，并返回相应结果的对象)

接口的实现在数据库驱动中，用于执行静态 SQL 语句并返回它所生成结果的对象。

```java
//根据查询语句返回结果集。只能执行select语句。  　　
ResultSet executeQuery(String sql) 　　
//根据执行的DML（insert update delete）语句，返回受影响的行数。 　　
int executeUpdate(String sql)
//此方法可以执行任意sql语句。返回boolean值，表示是否返回ResultSet结果集。仅当执行select语句，且有返回结果时返回true, 其它语句都返回false。
boolean execute(String sql) 　　 
```

### java.sql.ResultSet接口:（结果集（客户端存表数据的对象））

1）封装结果集
提供一个游标，默认游标指向结果集第一行之前
调用一次next()，游标向下移动一行 　　　　
提供一些get方法

2）封装数据的方法

```java
Object getObject(int columnIndex); 　　根据序号取值，索引从1开始
Object getObject(String ColomnName); 　　根据列名取值

boolean next() 　　将光标从当前位置向下移动一行
int getInt(int colIndex) 　　　　 以int形式获取ResultSet结果集当前行指定列号值
int getInt(String colLabel)　　  以int形式获取ResultSet结果集当前行指定列名值
float getFloat(int colIndex) 　　以float形式获取ResultSet结果集当前行指定列号值
float getFloat(String colLabel)　以float形式获取ResultSet结果集当前行指定列名值
String getString(int colIndex) 　以String 形式获取ResultSet结果集当前行指定列号值
String getString(String colLabel) 以String形式获取ResultSet结果集当前行指定列名值
Date getDate(int columnIndex);  
Date getDate(String columnName);
void close() 　　关闭ResultSet 对象
```

## 细说JDBC连接过程

连接数据库所需要的信息：

```properties
driverClass=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/bank?useSSL=false&serverTimezone=UTC
name=root
password=123456
```

### 注册驱动

第一种：Class.forName("com.mysql.cj.jdbc.Driver");　　
第二种：DriverManager.register(new com,mysql.cj.jdbc.Driver());　　

1）什么是驱动

**驱动就是JDBC实现类**，通俗点讲，就是能够连接到数据库功能的东西就是驱动，由于市面上有很多数据库，Oracle、MySql等等，所以**java就有一个连接数据库的实现规范接口**，定义一系列的连接数据库接口**(java.sql.Driver接口)**，**但是不提供实现，而每个数据库厂家来提供这些接口的具体实现**，这样一来，不管使用的是什么数据库，我们开发者写的代码都是相同的，就不必因为数据库的不同，而写法不同，唯一的不同就是数据库驱动不一样，使用mysql，那么就必须使用mysql的驱动，使用Oracle就必须使用oracle的驱动实现类。

2）DriverManager，一个工具类，是用于操作管理JDBC实现类的

```java
原始写法：DriverManager.register(new Driver());　　//因为使用的是MySql，所以在导包时就需要导入com.mysql.jdbc.Driver

现在写法：Class.forName("com.mysql.jdbc.Driver");　　//不用导包，会执行com.mysql.jdbc.Driver类中的静态代码块，其静态代码块的内容为
static {
    try {                      
        java.sql.DriverManager.registerDriver(new Driver());
    } catch (SQLException E) {
        throw new RuntimeException("Can't register driver!");
    }
}　　
```

分析：　

会发现第二种加载驱动的方法的底层其实就是第一种加载驱动。为什么要这样呢？原因很简单，　第一种是硬编程，直接将数据库驱动给写死了，无法扩展，如果使用第一种，那么连接的数据库只能是mysql，因为导包导的是mysql的驱动包，如果换成Oracle，就会报错，需要在代码中将Oracle的驱动包导入，这样很麻烦，而第二种写法就不一样了，**第二种是使用的字符串方法注册驱动的，我们只需要将该字符串提取到一个配置文件中，以后想换成oracle数据库，只需要将该字符串换成oracle驱动的类全名即可,而不需要到代码中去修改什么东西。**

### 获取（Connection）连接对象

```java
第一种：Connection conn = DriverManager.getConnection(url);
第二种：Connection conn = DriverManager.getConnection(url,user,password);
```

### 获取Statement对象（执行sql语句对象）

```java
Statement st = conn.createStatement();　　//获取sql语句执行对象
st.excuteUpdate(sql);　　//执行增删改语句
st.excuteQuery(sql);　　 //执行查询语句　

PraparedStatment ps = conn.prapareStatement(sql);　　//获取sql语句执行对象praparedStatment
//赋值
//可以设置很多中类型，index从1开始，代表sql语句中的第几个未知参数
ps.setInt(Index,value);　　ps.setString(index,value);　　
ps.excuteUpdate();　　  //执行增删改语句
ps.excuteQuery(sql);　　//执行查询语句
```

### 执行SQL语句,获取结果集对象

```java
//执行增删改的sql语句时，返回一个int类型的整数，代表数据库表影响的行数，
int count = ps.excuteUpdate();　　　

//执行查询sql语句时，返回一个结果集对象，该对象装着所有查询到的数据信息，一行一行的存储数据库表信息。
ResultSet rs = ps.executeQuery(sql);
```

### 如果有结果集（ResultSet），则处理结果集　

```java
while(rs.next()){
    // 获取行数据的第一种方式
    rs.getString(index);     //index代表第几列，从1开始
    // 获取行数据的第二中方式
    rs.getString(string);　　//string：代表字段名称。
}
```

### 关闭连接，避免计算机资源消耗　

```java
rs.close();
stat.close();
conn.close();　
```

### 总结

java的JDBC就分为5步，
4个属性
​	driver、url、user、password
五步：
​	注册驱动、获取连接、获取执行sql语句对象、获取结果集对象、处理结果。

## Statement和PrepareStatment

### 关系与区别

　　关系：PreparedStatement继承自Statement,都是接口。
　　区别：PreparedStatement可以使用占位符，是预编译的，批处理比Statement效率高。sql语句可以不是完整的，可以将参数用?替代，然后在预编译后加入未知参数

### 实例显示区别

　　1）背景：有一个数据库，里面有个tb_users表，有id，name和passwd三列，然后按照给定的name和password值进行数据查询。

使用Statement查询：

```java
sql="select * from tb_users where name='"+name+"' and passwd='"+passwd+"'";
Statement stmt=conn.createStatement();
rs=stmt.executeQuery(sql);
```

使用PrepareStatement查询：

```java
sql="select * from tb_users where username=? and userpwd=?";
PreparedStatement pstmt=conn.prepareStatement(sql);
pstmt.setString(1,name);
pstmt.setString(2,passwd);
rs=pstmt.executeQuery();
```

　　2）有一个数据库表tb_books，包含id，name，author，price四列，向该表中插入数据。

使用Statement：

```java
sql="insert into tb_books(id,name,author,price) values('"+id+"','"+name+"',"+author+",'"+price+"')";
Statement stmt = conn.createStatement();  
stmt.executeUpdate(sql); 
```

使用PrepareStatement：

```java
String sql="insert into book(id,name,author,price) values (?,?,?,?)";  
PreparedStatement pstmt = conn.prepareStatement(sql);  
pstmt.setString(1,var1);
pstmt.setString(2,var2);
pstmt.setString(3,var3);
pstmt.setString(4,var4);
pstmt.executeUpdate();
```

### PrepareStatement的优点

　　1）PrepareStatement可以提高代码的可读性
　　2）ParperStatement提高了代码的灵活性和执行效率

PrepareStatement接口是Statement接口的子接口，他继承了Statement接口的所有功能。它主要是拿来解决我们使用Statement对象多次执行同一个SQL语句的效率问题的。 　　　 

ParperStatement接口的机制是在数据库支持预编译的情况下预先将SQL语句编译，当多次执行这条SQL语句时，可以直接执行编译好的SQL语句，这样就大大提高了程序的灵活性和执行效率。

　　3）使用PrepareStatement比Statement安全　

举例：用户登录时进行密码验证

```java
sql="select * from tb_users where name= '"+name+"' and passwd='"+passwd+"'";
stmt = conn.createStatement();  
rs = stmt.executeUpdate(sql); 
```

上面是登录时进行用户名和密码的验证，但是当把‘’or ’1’=’’1’作为密码传递进去会发现如下的SQL语句：

```java
select * from user where username = 'user' and userpwd='' or '1'='1';
```

上面的SQL语句是个永真式。所以不管怎么样都能获取到权限，这还不是最坏的情况。
当把'or '1'=1';drop table tb_book;当成密码传进去时，会直接删除数据库表，这样的SQL语句就非常的不安全。
所以使用PrepareStatement可以解决SQL注入攻击的问题

## JDBC工具类

```java
package com.itheima.uitl;

import java.io.FileInputStream;
import java.io.InputStream;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.Properties;

public class JDBCUtil {

    static String driverClass = null;
    static String url = null;
    static String name = null;
    static String password = null;

    static {
        try {
            //1. 创建一个属性配置对象
            Properties properties = new Properties();
            //InputStream is = new FileInputStream("jdbc.properties");

            //使用类加载器，去读取src底下的资源文件。 后面在servlet
            InputStream is = JDBCUtil.class.getClassLoader().getResourceAsStream("jdbc.properties");
            //导入输入流。
            properties.load(is);

            //读取属性
            driverClass = properties.getProperty("driverClass");
            url = properties.getProperty("url");
            name = properties.getProperty("name");
            password = properties.getProperty("password");

        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 获取连接对象
     *
     * @return
     */
    public static Connection getConn() {
        Connection conn = null;
        try {
            Class.forName(driverClass);
            //静态代码块 ---> 类加载了，就执行。 java.sql.DriverManager.registerDriver(new Driver());
            //DriverManager.registerDriver(new com.mysql.jdbc.Driver());
            //DriverManager.getConnection("jdbc:mysql://localhost/test?user=monty&password=greatsqldb");
            //2. 建立连接 参数一： 协议 + 访问的数据库 ， 参数二： 用户名 ， 参数三： 密码。
            conn = DriverManager.getConnection(url, name, password);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return conn;
    }

    /**
     * 释放资源
     *
     * @param conn
     * @param st
     * @param rs
     */
    public static void release(Connection conn, Statement st, ResultSet rs) {
        closeRs(rs);
        closeSt(st);
        closeConn(conn);
    }

    public static void release(Connection conn, Statement st) {
        closeSt(st);
        closeConn(conn);
    }

    private static void closeRs(ResultSet rs) {
        try {
            if (rs != null) {
                rs.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            rs = null;
        }
    }

    private static void closeSt(Statement st) {
        try {
            if (st != null) {
                st.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            st = null;
        }
    }

    private static void closeConn(Connection conn) {
        try {
            if (conn != null) {
                conn.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            conn = null;
        }
    }
}
```

## JDBC处理MySQL中的TEXT数据

TEXT数据是长文本数据，即CLOB（Character Large Object），使用CHAR来保存数据

### text类型也可以存储字符串

```java
@Test
public void clobTest_1() {
    Connection conn = null;
    PreparedStatement pstmt = null;

    try {
        conn = JDBCUtil.getConn();
        //            String sql = "create table tb_clob_test(id int primary key auto_increment,clob_data text)";
        String sql = "insert into tb_clob_test(clob_data)values(?)";
        pstmt = conn.prepareStatement(sql);
        pstmt.setString(1, "hello world");
        pstmt.execute();
    } catch (SQLException e) {
        e.printStackTrace();
    } finally {
        JDBCUtil.release(conn, pstmt, null);
    }
}
```

### 数据的存入　

```java
//数据只有ASCII，中文好像也可以
//InputStream is = new FileInputStream("java.txt");
//pstmt.setAsciiStream(1, is);

//数据包含中文字符或者其他特殊字符，以下两种都可以
//pstmt.setClob(1,new FileReader("java.txt"));
//pstmt.setCharacterStream(1,new FileReader("java.txt"));

@Test
public void clobTest_2() {
    Connection conn = null;
    PreparedStatement pstmt = null;

    try {
        conn = JDBCUtil.getConn();
        String sql = "insert into tb_clob_test(clob_data)values(?)";
        pstmt = conn.prepareStatement(sql);

        //数据只有ASCII，中文好像也可以
        InputStream is = new FileInputStream("java.txt");
        pstmt.setAsciiStream(1, is);

        //数据包含中文字符或者其他特殊字符，以下两种都可以
        //pstmt.setClob(1,new FileReader("java.txt"));
        //pstmt.setCharacterStream(1,new FileReader("java.txt"));
        pstmt.execute();
    } catch (SQLException e) {
        e.printStackTrace();
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } finally {
        JDBCUtil.release(conn, pstmt, null);
    }
}
```

###  数据的获取

- 直接拿出字符流　

```java
@Test
public void clobSelectTest_1() {
    Connection conn = null;
    PreparedStatement pstmt = null;
    ResultSet rs = null;
    PrintWriter writer = null;
    BufferedReader bf = null;
    try {
        conn = JDBCUtil.getConn();
        String sql = "select * from tb_clob_test where id = 6";
        pstmt = conn.prepareStatement(sql);
        rs = pstmt.executeQuery();
        writer = new PrintWriter("java.txt.bak");
        while(rs.next()){
            int id = rs.getInt(1);
            Reader reader = rs.getCharacterStream(2);
            bf = new BufferedReader(reader);
            String str = null;
            while ((str=bf.readLine())!=null){
                writer.write(str);
                writer.write("\r\n");
                writer.flush();
            }

        }
        //pstmt.execute();
    } catch (SQLException e) {
        e.printStackTrace();
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        try {
            bf.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
        try {
            writer.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
        JDBCUtil.release(conn, pstmt, null);
    }
}
```

- 拿出Clob，然后转化成字符流

```java
@Test
public void clobSelectTest_2() {
    Connection conn = null;
    PreparedStatement pstmt = null;
    ResultSet res = null;
    PrintWriter writer = null;
    BufferedReader bf = null;
    try {
        conn = JDBCUtil.getConn();
        String sql = "select * from tb_clob_test where id = 6";
        pstmt = conn.prepareStatement(sql);
        res = pstmt.executeQuery();
        writer = new PrintWriter("java.txt.bak");
        while (res.next()) {
            File file = new File("note_bak.txt");
            FileWriter fw = new FileWriter(file);
            Clob clob = res.getClob(2);
            Reader reader = clob.getCharacterStream();
            char[] cs = new char[1024];
            int length = 0;
            while((length = reader.read(cs))!=-1){
                fw.write(cs, 0, length);
                fw.flush();
            }
            fw.close();
            reader.close();
        }
        //pstmt.execute();
    } catch (SQLException e) {
        e.printStackTrace();
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        JDBCUtil.release(conn, pstmt, null);
    }
}
```

## JDBC处理BLOB数据

BLOB（binary large object）是二进制形式的长文本数据，使用二进制保存数据（如图片等）

### 存储图片　

- 方式1

```java
@Test
public void blobInsert_1(){
    Connection conn = null;
    PreparedStatement pstmt = null;

    try {
        conn = JDBCUtil.getConn();
        //            String sql = "create table tb_blob_test(id int primary key auto_increment,blob_data longblob)";
        String sql = "insert into tb_blob_test(blob_data)values(?)";
        pstmt = conn.prepareStatement(sql);
        pstmt.setBinaryStream(1,new FileInputStream("1.png"));
        boolean execute = pstmt.execute();
        System.out.println(execute);
    } catch (SQLException e) {
        e.printStackTrace();
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    }finally {
        JDBCUtil.release(conn, pstmt, null);
    }
}
```

- 方式2

```java
@Test
public void blobInsert_1(){
    Connection conn = null;
    PreparedStatement pstmt = null;

    try {
        conn = JDBCUtil.getConn();
        //            String sql = "create table tb_blob_test(id int primary key auto_increment,blob_data longblob)";
        String sql = "insert into tb_blob_test(blob_data)values(?)";
        pstmt = conn.prepareStatement(sql);
        pstmt.setBinaryStream(1,new FileInputStream("1.png"));
        boolean execute = pstmt.execute();
        System.out.println(execute);
    } catch (SQLException e) {
        e.printStackTrace();
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    }finally {
        JDBCUtil.release(conn, pstmt, null);
    }
}
```

### 从数据库中读取图片

```java
@Test
public void blobReader() {
    Connection conn = null;
    PreparedStatement pstmt = null;

    try {
        conn = JDBCUtil.getConn();
        String sql = "select * from tb_blob_test where id =1";
        pstmt = conn.prepareStatement(sql);
        ResultSet rs = pstmt.executeQuery();
        rs.next();
        InputStream is = rs.getBinaryStream(2);
        OutputStream os = new FileOutputStream("blob.jpg");
        byte[] bs = new byte[1024];
        int len = -1;
        while ((len=is.read(bs))!=-1){
            os.write(bs,0,len);
        }
        os.flush();
        os.close();
    } catch (SQLException e) {
        e.printStackTrace();
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    }finally {
        JDBCUtil.release(conn, pstmt, null);
    }
}
```

## JDBC中设置事务的隔离级别

### 在JDBC中一些基本的设置事务的操作　

```java
//MySQL设置事务隔离级别，一般不会再JDBC代码中设置，会直接在MySQL服务器中去设置
conn.setTransactionIsolation(Connection.TRANSACTION_REPEATABLE_READ);
//在JDBC设置手动提交事务
conn.setAutoCommit(false);
//设置回滚点
sp = conn.setSavepoint();
//事务回滚
conn.rollback(sp);
```

### 事务处理具体实例

```java
@Test
public void tx_test_1() {
    Connection conn = null;
    PreparedStatement pstmt = null;
    ResultSet rs = null;
    Savepoint sp = null;

    try {
        conn = JDBCUtil.getConn();
        // MySQL设置事务隔离级别，一般不会再JDBC代码中设置，会直接在MySQL服务器中去设置
        conn.setTransactionIsolation(Connection.TRANSACTION_REPEATABLE_READ);
        // 在JDBC设置手动提交事务
        conn.setAutoCommit(false);
        // 在转账的时候，先判断源账号的余额是否大于转账金额
        // 判断之前先查询源账户的余额
        String sql0 = "select money from account where id =2";
        pstmt = conn.prepareStatement(sql0);
        rs = pstmt.executeQuery();
        int balance = 0;
        if (rs.next()) {
            balance = rs.getInt(1);
        }
        // 转账金额
        int much = 1000;
        // 判断余额是否大于转账金额
        if (balance < much) {
            JDBCUtil.release(conn, pstmt, rs);
            System.exit(0);
        }

        sp = conn.setSavepoint();
        // 张三的余额减去1000
        String sql1 = "update account set money=money-1000 where id=2";
        pstmt = conn.prepareStatement(sql1);
        pstmt.executeUpdate();
        // 李四的余额加上1000
        String sql2 = "update account set money=money+1000 where id=1";
        //            pstmt.executeUpdate(sql2);这是调用父类的方法一样可以
        pstmt = conn.prepareStatement(sql2);
        pstmt.executeUpdate();
    } catch (SQLException e) {
        try {
            //                conn.rollback();
            conn.rollback(sp);
        } catch (SQLException e1) {
            e1.printStackTrace();
        }
        e.printStackTrace();
    } finally {
        try {
            conn.commit();
        } catch (SQLException e) {
            try {
                conn.rollback();
            } catch (SQLException e1) {
                e1.printStackTrace();
            }
            e.printStackTrace();
        }
        JDBCUtil.release(conn, pstmt, rs);
    }
}
```

## 数据库元数据的查看

- 前言：前面使用JDBC来处理数据库的接口主要有三个，即Connection，PreparedStatement和ResultSet这三个，而对于这三个接口，还可以获取不同类型的元数据，通过这些元数据类获得一些数据库的信息。

- **元数据(MetaData)**，**即定义数据的数据**。打个比方，就好像我们要想搜索一首歌(歌本身是数据)，而我们可以通过歌名，作者，专辑等信息来搜索，那么这些歌名，作者，专辑等等就是这首歌的元数据。因此数据库的元数据就是一些注明数据库信息的数据。

① 由Connection对象的getMetaData()方法获取的是DatabaseMetaData对象。
② 由PreparedStatement对象的getParameterMetaData ()方法获取的是ParameterMetaData对象。
③ 由ResultSet对象的getMetaData()方法获取的是ResultSetMetaData对象。

### DatabaseMetaData　

DatabaseMetaData是由Connection对象通过getMetaData方法获取而来，主要封装了是对数据库本身的一些整体综合信息。
例如数据库的产品名称，数据库的版本号，数据库的URL，是否支持事务等等，能获取的信息比较多，具体可以参考DatabaseMetaData的API文档。

```java
//查看当前连接中有关MySQL的系统信息，比如版本号，是否支持事务，数据库名字。
DatabaseMetaData data = conn.getMetaData(); 

以下有一些关于DatabaseMetaData的常用方法：
　　·getDatabaseProductName：获取数据库的产品名称
　　·getDatabaseProductName：获取数据库的版本号
　　·getUserName：获取数据库的用户名
　　·getURL：获取数据库连接的URL
　　·getDriverName：获取数据库的驱动名称
　　·getDriverVersion：获取数据库的驱动版本号
　　·isReadOnly：查看数据库是否只允许读操作
　　·supportsTransactions：查看数据库是否支持事务
```

### ParameterMetaData

ParameterMetaData是由PreparedStatement对象通过getParameterMetaData方法获取而来，主要是针对PreparedStatement对象和其预编译的SQL命令语句提供一些信息。

比如像”insert into account(id,name,money) values(?,?,?)”这样的预编译SQL语句，ParameterMetaData能提供占位符参数的个数，获取指定位置占位符的SQL类型等等，详细请看有关ParameterMetaData的API文档

- 常用方法：

getParameterCount：获取预编译SQL语句中占位符参数的个数
在我看来，ParameterMetaData对象能用的只有获取参数个数的getParameterCount()方法。
注意：ParameterMetaData许多方法MySQL并不友好支持，比如像获取指定参数的SQL类型的getParameterType方法，如果数据库驱动连接URL只是简单的“jdbc:mysql://localhost:3306/jdbcdemo”那么MyEclipse会抛出SQLException异常，必须要将URL修改为“jdbc:mysql://localhost:3306/jdbcdemo?generateSimpleParameterMetadata=true”才行。但是像getParameterType等等与其他的方法也没多好用，因为如下面的例子，这些方法好像只会将所有的参数认为是字符串(VARCHAR)类型。

```java
public void testParameterMetaData() throws SQLException {
    Connection conn = null;
    PreparedStatement st = null;
    ResultSet rs = null;
    try{
        conn = JdbcUtils.getConnection();
        String sql = "insert into user(id,name,age) values(?,?,?)";
        st = conn.prepareStatement(sql);
        st.setInt(1, 1);
        st.setString(2, "Ding");
        st.setInt(3, 25);

        ParameterMetaData paramMetaData = st.getParameterMetaData();
        //获取参数个数
        int paramCount = paramMetaData.getParameterCount();
        //以字符串形式获取指定参数的SQL类型，这里有问题
        String paramTypeName = paramMetaData.getParameterTypeName(1);
        //返回指定参数的SQL类型，以java.sql.Types类的字段表示，这里有问题
        int paramType = paramMetaData.getParameterType(1);
        //返回指定参数类型的Java完全限定名称，这里有问题
        String paramClassName = paramMetaData.getParameterClassName(1);
        //返回指定参数的模，这里有问题
        int paramMode = paramMetaData.getParameterMode(1);
        //返回指定参数的列大小，这里有问题
        int precision = paramMetaData.getPrecision(1);
        //返回指定参数的小数点右边的位数，这里有问题
        int scale = paramMetaData.getScale(1);
    }
}
```

**总结：**因此在以后使用参数元数据ParameterMetaData尽量只要使用其getParamterCount()方法获取参数个数，对于该对象其他方法请慎用。

### ResultSetMetaData

ResultSetMetaData是由ResultSet对象通过getMetaData方法获取而来，主要是针对由数据库执行的SQL脚本命令获取的结果集对象ResultSet中提供的一些信息。

比如结果集中的列数、指定列的名称、指定列的SQL类型等等，可以说这个是对于框架来说非常重要的一个对象。关于该结果集元数据对象的其他具体功能和方法请查阅有关ResultSetMetaData的API文档。　

```java
以下有一些关于ResultSetMetaData的常用方法：
　　·getColumnCount：获取结果集中列项目的个数
　　·getColumnType：获取指定列的SQL类型对应于Java中Types类的字段
　　·getColumnTypeName：获取指定列的SQL类型
　　·getClassName：获取指定列SQL类型对应于Java中的类型(包名加类名)
```

 **实例展示：**

- 数据表　

```sql
create table user(
    id int primary key,
    name varchar(40),
    age int
);
insert into user(id,name,age) values(1,'Ding',25);
insert into user(id,name,age) values(2,'LRR',24);
```

- 测试

```java
public void testResultSetMetaData() throws SQLException {
    Connection conn = null;
    PreparedStatement st = null;
    ResultSet rs = null;
    try{
        conn = JdbcUtils.getConnection();
        String sql = "select * from user";
        st = conn.prepareStatement(sql);

        rs = st.executeQuery();
        ResultSetMetaData resultMetaData = rs.getMetaData();
        //获取结果集的列数
        int columnCount = resultMetaData.getColumnCount();
        //获取指定列的名称
        String columnName = resultMetaData.getColumnName(1);
        //获取指定列的SQL类型对应于java.sql.Types类的字段
        int columnType = resultMetaData.getColumnType(1);
        //获取指定列的SQL类型
        String columnTypeName = resultMetaData.getColumnTypeName(1);
        //获取指定列SQL类型对应于Java的类型
        String className= resultMetaData.getColumnClassName(1);
        //获取指定列所在的表的名称
        String tableName = resultMetaData.getTableName(1);
    }
}
```

## 数据库连接池

- 问题：

我们在进行CRUD时，一直重复性的写一些代码，比如最开始的注册驱动，获取连接代码，一直重复写，通过编写一个获取连接的工具类后，解决了这个问题，但是又会出现新的问题，每进行一次操作，就会获取一个连接，用完之后，就销毁，就这样一直新建连接，销毁连接，新建，销毁，连接Connection 创建与销毁 比较耗时的。所以应该要想办法解决这个问题？

- 解决方法：

连接池就是为了解决这个问题而出现的一个方法，为了提高性能，开发连接池，连接池中一直保持有n个连接，供调用者使用，调用者用完返还给连接池，继续给别的调用使用，比如连接池中一开始就有10个连接，当有5个用户拿走了5个连接后，池中还剩5个，当第6个用户在去池中拿连接而前面5个连接还没归还时，连接池就会新建一个连接给第六个用户，让池中一直能够保存最少5个连接，而当这样新建了很多连接后，用户归还连接回来时，会比原先连接池中的10个连接更多，连接池就会设置一个池中最大空闲的连接数，如果超过了这个数，就会将超过的连接给释放掉，连接池就是这样工作的。

### 连接池概述

数据库连接池负责**分配、管理和释放**数据库连接，它允许应用程序重复使用一个现有的数据库连接，而不是再重新建立一个；释放空闲时间超过最大空闲时间的数据库连接来避免因为没有释放数据库连接而引起的数据库连接遗漏。这项技术能明显提高对数据库操作的性能。

### 比较应用程序直接获取连接和使用连接池

1. 应用程序直接获取连接

   缺点：用户每次请求都需要向数据库获得链接，而数据库创建连接通常需要消耗相对较大的资源，创建时间也较长。
   假设网站一天10万访问量，数据库服务器就需要创建10万次连接，极大的浪费数据库的资源，并且极易造成数据库服务器内存溢出、宕机。

2. 使用连接池连接

   目的：解决建立数据库连接耗费资源和时间很多的问题，提高性能。

## 常用的数据库连接池　　

现在很多WEB服务器(Weblogic, WebSphere, Tomcat)都提供了**DataSoruce的实现，即连接池的实现**。通常我们**把DataSource的实现，按其英文含义称之为数据源**，

数据源中都包含了数据库连接池的实现。也有一些开源组织提供了数据源的独立实现：

DBCP 数据库连接池
C3P0 数据库连接池  

实际应用时不需要编写连接数据库代码，直接从数据源获得数据库的连接。程序员编程时也应尽量使用这些数据源的实现，以提升程序的数据库访问性能。

DBCP、C3P0、tomcat内置连接池(JNDI)是我们开发中会用到的。

数据库连接池主要完成数据库的连接，示例代码如下：

```java
//静态代码块创建连接池，DataSource
static ComboPooledDataSource dataSource = null;
static{
    dataSource = new ComboPooledDataSource();
}

/**
* 获取连接对象
* @return
* @throws SQLException 
*/
public static Connection getConn() throws SQLException{
    return dataSource.getConnection();
}

/**
* 释放资源,代码略
*/
public static void release(Connection conn , Statement st , ResultSet rs)
```

## DbUtils

commons-dbutils 是 Apache 组织提供的一个开源 JDBC工具类库

常用API：

```
org.apache.commons.dbutils.QueryRunner --- 核心
org.apache.commons.dbutils.ResultSetHandler --- 结果集封装器
org.apache.commons.dbutils.DbUtils --- 工具类
```

### QueryRunner类

> 构造方法：QueryRunner(DataSource ds)

若使用C3P0连接池，可以通过连接池建立连接，用作后续的数据库CRUD等操作，代码如下：

> QueryRunner queryRunner = new QueryRunner(new ComboPooledDataSourse());

增删改操作：

> queryRunner.update(String sql, Object... params);
> 前面是sql语句，后面是参数，即sql语句中的？,例如：
> queryRunner.update("insert into account(id,name,money)  values(null,?,?)", "zhangsan",1000);

查询操作：

> queryRunner.query(String sql, Object... params);

### ResultSetHandler接口

该接口用于处理 java.sql.ResultSet，将获取的数据按要求转换为另一种形式

ResultSetHandler接口有很多的实现类，所以一般在用接口的时候，直接选择new一个实现类，调用相关方法。

> BeanHandler可以返回一个对象的信息
>  BeanListHandler可以返回多个对象的信息，并且用list集合封装。

``` java
// 查询单个对象
Account account=queryRunner.query（"select*from account where id=？"，new BeanHandler<Account>（Account.class），8）；System.out.println(（)account.tostring())；
// 查询多个对象
/*List<Account>list=queryRunner.query（"select*from account"，new BeanListHandler<Account>（Account.class））；for（Account account:list）{
system.out.print1n（account.tostring（））；
}*/

字节码的用途：反射得到实例
// 通过类的字节码得到该类的实例
Account a = new Account（）;
// 创建一个类的实例
Account a1 = Account.class.newInstance();
```

