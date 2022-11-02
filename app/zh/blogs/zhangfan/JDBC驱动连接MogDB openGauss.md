---
title: 'JDBC驱动连接MogDB/openGauss'

date: '2022-04-07'

category: 'blog'
tags: ['JDBC驱动连接MogDB/openGauss']

archives: '2022-04'

author: '云和恩墨-张凡'

summary: 'JDBC驱动连接MogDB/openGauss'

img: '/zh/blogs/zhangfan/title/img20.png'

times: '10:20'
---

# JDBC 驱动连接 MogDB/openGauss

## 一、环境说明

```
[root@node1 ~]# cat /etc/redhat-release
CentOS Linux release 7.6.1810 (Core)
[root@node1 ext]# java -version
java version "1.8.0_301"
Java(TM) SE Runtime Environment (build 1.8.0_301-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.301-b09, mixed mode)
```

## 二、数据库配置

### 1.配置数据库参数，允许用户登录

```
数据库配置文件postgresql.conf和pg_hba.conf中加上如下内容
[omm@node1 data]$ tail -4 postgresql.conf
listen_addresses = '0.0.0.0'
password_encryption_type = 0
log_directory = 'pg_log'
remote_read_mode=non_authentication
[omm@node1 data]$ tail -1 pg_hba.conf
host all all 0.0.0.0/0 md5
重启数据库
gs_om -t stop
gs_om -t start
```

### 2.创建连接用户及数据库

```
postgres=# create database jdbc_db;
CREATE DATABASE
postgres=# create user jdbc_usr password 'jdbc@123';
NOTICE:  The encrypted password contains MD5 ciphertext,    which is not secure.
CREATE ROLE
postgres=# alter user jdbc_usr sysadmin;
ALTER ROLE
postgres=#
```

## 三、Java 程序编写

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class ConnTest {
    //创建数据库连接。
    public static Connection GetConnection(String username, String passwd) {
        String driver = "org.postgresql.Driver";
        String sourceURL = "jdbc:postgresql://8.131.53.xxx:26000/jdbc_db";
        Connection conn = null;
        try {
            //加载数据库驱动。
            Class.forName(driver).newInstance();
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }

        try {
            //创建数据库连接。
            conn = DriverManager.getConnection(sourceURL,"jdbc_usr", "jdbc@123");
            System.out.println("连接成功!");
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
        return conn;
    }
    ;
        /**
         * 把查询到的结果放入ResultSet
         * 通过迭代的方法去读取结果集中的查询结果
         * 输出查询结果
         */
        public static void Select(Connection conn) {
            PreparedStatement ps = null;
            ResultSet rs = null;
            String sql = "SELECT version()";
            try {
                ps = conn.prepareStatement(sql);
                rs = ps.executeQuery();		//将查询的结果放入ResultSet结果集中
                /**
                 * 从结果集ResultSet中迭代取出查询结果并输出
                 */
                while(rs.next()) {
//                    String values = rs.getString("id");
                    String values = rs.getString("version");

                    System.out.println( "数据库版本："+values);
                }
            } catch (SQLException e) {
                System.out.println("操作失败o(╥﹏╥");
                e.printStackTrace();
            }
        }
    /**
     * 主程序，逐步调用各静态方法。
     * @param args
     */
    public static void main(String[] args) {
        //创建数据库连接。
        Connection conn = GetConnection("jdbc_usr", "jdbc@123");
        Select(conn);
        //关闭数据库连接。
        try {
            conn.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

## 四、程序测试

### 1.放置 jDBC 驱动

```java
将jdbc驱动放到jdk中的如下目录，让程序能找到驱动包
[root@node1 ext]# pwd
/usr/java/jdk1.8.0_301-amd64/jre/lib/ext
[root@node1 ext]# wget https://opengauss.obs.cn-south-1.myhuaweicloud.com/2.0.1/x86/openGauss-2.0.0-JDBC.tar.gz
2021-12-01 17:30:52 (13.2 MB/s) - 已保存 “openGauss-2.0.0-JDBC.tar.gz” [4937896/4937896])
[root@node1 ext]# tar -zxvf openGauss-2.0.0-JDBC.tar.gz
postgresql.jar
```

### 2.运行程序

这里采用了俩种方式运行程序，一种是单个程序直接运行，另一个则是将 Java 程序打成 jar 在运行，这里简单介绍一下

#### (1)单个程序运行

```java
[root@node1 hello]# ls
conn.jar  ConnTest.java  MANIFEST.MF  postgresql.jar
[root@node1 hello]# pwd
/root/java_program/hello
[root@node1 hello]# javac ConnTest.java
[root@node1 hello]# java ConnTest
连接成功!
数据库版本：PostgreSQL 9.2.4 (MogDB 2.0.1 build f892ccb7) compiled at 2021-07-09 16:12:59 commit 0 last mr   on     x86_64-unknown-linux-gnu, compiled by g++ (GCC) 7.3.0, 64-bit
```

#### (2)jar 包运行

##### 编译 ConnTest.java

```java
[root@node1 hello]# javac ConnTest.java
```

##### 编写 MANIFEST.MF 文件

```java
MANIFEST.MF文件介绍
META-INF文件夹相当于一个信息包，目录中的文件和目录获得Java 2平台的认可与解释，用来配置应用程序、扩展程序、类加载器和服务。这个文件夹和其中的 MANIFEST.MF文件，在用jar打包时自动生成。执行jar文件的时候，这个jar里是需要具备 META-INF/MANIFEST.MF的，否则java -jar就找不到main class。
[root@node1 hello]# cat MANIFEST.MF
Manifest-Version: 1.0
Main-Class: ConnTest
```

##### 程序打包

```java
[root@node1 hello]# jar -cvfm conn.jar MANIFEST.MF ConnTest.class
已添加清单
正在添加: ConnTest.class(输入 = 2126) (输出 = 1212)(压缩了 42%)
```

##### 运行程序

```java
[root@node1 hello]# java -jar conn.jar
连接成功!
数据库版本：PostgreSQL 9.2.4 (MogDB 2.0.1 build f892ccb7) compiledat    2021-07-09 16:12:59   commit 0 last mr   onx86_64-unknown-linux-gnu,   compiled by g++ (GCC) 7.3.0, 64-bit
```

## 五、总结

上述文章简单介绍了 JDBC 连接 MogDB 数据库,数据如何配置，以及 JDBC 驱动如何加载，如何配置，并运行在 Linux 上。更多细节参考官方文档https://docs.mogdb.io/zh/mogdb/v2.0.1/1-development-based-on-jdbc-overview
