## DBCP数据库连接池
传统的JDBC中，**用户每次请求都需要向数据库获得链接(Connection)**，而数据库创建连接通常需要消耗相对较大的资源，创建时间也较长。

假设网站一天10万访问量，数据库服务器就需要创建10万次连接，极大的浪费数据库的资源，并且极易造成数据库服务器内存溢出、宕机。

### 连接池大致原理

在服务器端一次性创建多个连接，将**多个连接保存在一个连接池对象中**，当应用程序的请求需要操作数据库时，不会为请求创建新的连接，而是直接从连接池中获得一个连接，操作数据库结束之后，**并不需要真正关闭连接，而是将连接放回到连接池中**。

这样可以**节省创建连接、释放连接 资源**

### DBCP连接池
编写连接池需实现javax.sql.DataSource接口。

现在很多WEB服务器(Weblogic, WebSphere, Tomcat)都提供了DataSoruce的实现，即连接池的实现。通常我们把DataSource的实现，按其英文含义称之为数据源，数据源中都包含了数据库连接池的实现。

也有一些开源组织提供了数据源的独立实现：

 - Apache commons-dbcp 数据库连接池

 - C3P0 数据库连接池

 - Apache Tomcat内置的连接池（apache dbcp）

实际应用时不需要编写连接数据库代码，直接从数据源获得数据库的连接。程序员编程时也应尽量使用这些数据源的实现，以提升程序的数据库访问性能。

DBCP 是 Apache 软件基金组织下的开源连接池实现

使用DBCP连接池，需要有driverclass、url、username、password

参考代码

``` java
// 使用BasicDataSource 创建连接池
      BasicDataSource basicDataSource = new BasicDataSource();
      // 创建连接池 一次性创建多个连接池

      // 连接池 创建连接 ---需要四个参数
      basicDataSource.setDriverClassName("com.mysql.jdbc.Driver");
      basicDataSource.setUrl("jdbc:mysql:///day14");
      basicDataSource.setUsername("root");
      basicDataSource.setPassword("123");

      // 从连接池中获取连接
      Connection conn = basicDataSource.getConnection();
      String sql = "select * from account";
      PreparedStatement stmt = conn.prepareStatement(sql);
      ResultSet rs = stmt.executeQuery();

      while (rs.next()) {
          System.out.println(rs.getString("name"));
      }

```
