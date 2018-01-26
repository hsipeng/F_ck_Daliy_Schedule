## environment

MyBatis 支持配置多个 dataSource 环境，可以将应用部署到不同的环境上，如 DEV(开发环境)，TEST(测试换将)， QA(质量评估环境),UAT(用户验收环境),PRODUCTION(生产环境)，可以通过将默认 environment 值设置成想要的 environment id 值
如果你的应用需要连接多个数据库，你需要将每个数据库配置成独立的环境，并且为每一个数据库创建一个 SqlSessionFactory。

```java

inputStream = Resources.getResourceAsStream("mybatis-config.xml");
// 默认environment
defaultSqlSessionFactory = new SqlSessionFactoryBuilder().
build(inputStream);
// environment id 为shoppingcart
//
cartSqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStre
        am, "shoppingcart");
```


## 数据源 DataSource

dataSource 的类型可以配置成其内置类型之一，如 UNPOOLED，POOLED，JNDI。
- 如果将类型设置成UNPOOLED，MyBatis会为每一个数据库操作创建一个新的连接，并关闭它。该方式
适用于只有小规模数量并发用户的简单应用程序上。
- 如果将属性设置成POOLED，MyBatis会创建一个数据库连接池，连接池中的一个连接将会被用作数据
库操作。一旦数据库操作完成，MyBatis 会将此连接返回给连接池。在开发或测试环境中，经常使用此
种方式。
- 如果将类型设置成JNDI，MyBatis从在应用服务器向配置好的JNDI数据源dataSource获取数据库
连接。在生产环境中，优先考虑这种方式。


## 事务管理器 TransactionManager

MyBatis 支持两种类型的事务管理器: JDBC and MANAGED.
- JDBC事务管理器被用作当应用程序负责管理数据库连接的生命周期(提交、回退等等)的时候。当你将
TransactionManager 属性设置成 JDBC，MyBatis 内部将使用 JdbcTransactionFactory 类创建
TransactionManager。例如，部署到 Apache Tomcat 的应用程序，需要应用程序自己管理事务。
- MANAGED 事务管理器是当由应用服务器负责管理数据库连接生命周期的时候使用。当你将 TransactionManager 属性设置成 MANAGED 时，MyBatis 内部使用 ManagedTransactionFactory 类创建事务管理器TransactionManager。例如，当一个JavaEE的应用程序部署在类似 JBoss，WebLogic， GlassFish 应用服务器上时，它们会使用 EJB 进行应用服务器的事务管理能力。在这些管理环境中，你 可以使用 MANAGED 事务管理器。
(译者注:Managed 是托管的意思，即是应用本身不去管理事务，而是把事务管理交给应用所在的服务 器进行管理。)


## 属性 Properties

属性配置元素可以将配置值具体化到一个属性文件中，并且使用属性文件的 key 名作为占位符。在上述的配置中，我 们将数据库连接属性具体化到了 application.properties 文件中，并且为 driver，URL 等属性使用了占位符。

```xml

<properties resource="application.properties">
  <property name="jdbc.username" value="db_user" />
  <property name="jdbc.password" value="verysecurepwd" />
</properties>
<dataSource type="POOLED">
  <property name="driver" value="${jdbc.driverClassName}" />
  <property name="url" value="${jdbc.url}" />
  <property name="username" value="${jdbc.username}" />
  <property name="password" value="${jdbc.password}" />
</dataSource>
```


## 类型别名 typeAliases

```xml
<typeAliases>
<typeAlias alias="Student" type="com.mybatis3.domain.Student" /> <typeAlias alias="Tutor" type="com.mybatis3.domain.Tutor" /> <package name="com.mybatis3.domain" />
<package name="com.mybatis3.webservices.domain" />
</typeAliases>
```

另外一种方式为 JavaBeans 起别名，使用注解@Alias:

```java
@Alias("StudentAlias")
public class Student
{
}
```

> @Alias 注解将会覆盖配置文件中的<typeAliases>定义。

## 类型处理器 typeHandlers

> MyBaits 会将 java.util.Date 类型转换为 into java.sql.Timestamp 并设值:
Java Code`pstmt.setTimestamp(4, new Timestamp((student.getDob()).getTime()));`

一旦我们实现了自定义的类型处理器，我们需要在mybatis-config.xml中注册它
```java
packagecom.mybatis3.typehandlers;
importjava.sql.CallableStatement;
importjava.sql.PreparedStatement;
importjava.sql.ResultSet;
importjava.sql.SQLException;
importorg.apache.ibatis.type.BaseTypeHandler;
importorg.apache.ibatis.type.JdbcType;
importcom.mybatis3.domain.PhoneNumber;
public class PhoneTypeHandler extends BaseTypeHandler<PhoneNumber>
{
    @Override
    public void setNonNullParameter(PreparedStatement ps, int i,
                                    PhoneNumber parameter, JdbcType jdbcType) throws
        SQLException
    {
        ps.setString(i, parameter.getAsString());
    }
    @Override
    public PhoneNumber getNullableResult(ResultSet rs, String
columnName)
    throws SQLException
    {
        return new PhoneNumber(rs.getString(columnName));
    }
    @Override
    public PhoneNumber getNullableResult(ResultSet rs, int
                                         columnIndex)
    throws SQLException
    {
        return new PhoneNumber(rs.getString(columnIndex));
        }
    @Override
    public PhoneNumber getNullableResult(CallableStatement cs, int
                                         columnIndex)
    throws SQLException
    {
        return new PhoneNumber(cs.getString(columnIndex));
    }
}
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <properties resource="application.properties" />
  <typeHandlers>
    <typeHandler handler="com.mybatis3.typehandlers.PhoneTypeHandler" />
  </typeHandlers>
</configuration>
```


## 全局参数设置 Settings

```xml
<settings>
               <setting name="cacheEnabled" value="true" />
               <setting name="lazyLoadingEnabled" value="true" />
               <setting name="multipleResultSetsEnabled" value="true" />
               <setting name="useColumnLabel" value="true" />
               <setting name="useGeneratedKeys" value="false" />
               <setting name="autoMappingBehavior" value="PARTIAL" />
               <setting name="defaultExecutorType" value="SIMPLE" />
               <setting name="defaultStatementTimeout" value="25000" />
               <setting name="safeRowBoundsEnabled" value="false" />
               <setting name="mapUnderscoreToCamelCase" value="false" />
               <setting name="localCacheScope" value="SESSION" />
<setting name="jdbcTypeForNull" value="OTHER" />
<setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode ,toString" />
</settings>

```

## SQL映射定义Mappers
```xml
<mappers>
<mapper resource="com/mybatis3/mappers/StudentMapper.xml" /> <mapper url="file:///D:/mybatisdemo/app/mappers/TutorMapper.xml" /> <mapper class="com.mybatis3.mappers.TutorMapper" />
<package name="com.mybatis3.mappers" />
</mappers>
```

以上每一个<mapper> 标签的属性有助于从不同类型的资源中加载映射 mapper:
- resource属性用来指定在classpath中的mapper文件。
- url属性用来通过完全文件系统路径或者webURL地址来指向mapper文件  class属性用来指向一个mapper接口
- package属性用来指向可以找到Mapper接口的包名


