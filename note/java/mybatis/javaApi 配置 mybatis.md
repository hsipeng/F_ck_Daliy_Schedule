## Java API 创建 SqlSessionFactory

```java
public static SqlSessionFactory getSqlSessionFactory()
{
    SqlSessionFactory sqlSessionFactory = null;
try
    {
        DataSource dataSource = DataSourceFactory.getDataSource();
        TransactionFactory transactionFactory = new
        JdbcTransactionFactory();
        Environment environment = new Environment("development",
                transactionFactory, dataSource);
        Configuration configuration = new Configuration(environment);
        configuration.getTypeAliasRegistry().registerAlias("student",
                Student.class);
        configuration.getTypeHandlerRegistry().register(PhoneNumber.
                class, PhoneTypeHandler.class);
        configuration.addMapper(StudentMapper.class);
        sqlSessionFactory = new SqlSessionFactoryBuilder().
        build(configuration);
    }
    catch (Exception e)
    {
        throw new RuntimeException(e);
    }
    return sqlSessionFactory;
}

```

## 数据源DataSource

```java
public class DataSourceFactory
{
    public static DataSource getDataSource()
    {
        String driver = "com.mysql.jdbc.Driver";
        String url = "jdbc:mysql://localhost:3306/mybatisdemo";
        String username = "root";
        String password = "admin";
        PooledDataSource dataSource = new PooledDataSource(driver, url,
        username, password);
return dataSource;
}
}
```
- JNDI 获取 DataSource 对象
```java
public class DataSourceFactory
{
    public static DataSource getDataSource()
    {

String jndiName = "java:comp/env/jdbc/MyBatisDemoDS";
try
{
    InitialContext ctx = new InitialContext();
    DataSource dataSource = (DataSource) ctx.lookup(jndiName);
    return dataSource;
}
catch (NamingException e)
{
    throw new RuntimeException(e);
}
}
}
```

> 当前有一些流行的第三方类库，如 commons-dbcp 和 c3p0 实现了 java.sql.DataSource,你可以使用它们来创建 dataSource。

## 事务工厂 TransactionFactory

MyBatis 支持一下两种 TransactionFactory 实现:
- JdbcTransactionFactory
- ManagedTransactionFactory

```java

DataSource dataSource = DataSourceFactory.getDataSource();
TransactionFactory txnFactory = new JdbcTransactionFactory();
Environment environment = new Environment("development", txnFactory,
        dataSource);

```
托管
```java
DataSource dataSource = DataSourceFactory.getDataSource();
TransactionFactory txnFactory = new ManagedTransactionFactory();
Environment environment = new Environment("development", txnFactory,
        dataSource);
```


## 类型别名 typeAliases

```java
//1. 根据默认的别名规则，使用一个类的首字母小写、非完全限定的类名作为别名注册，可使用以下代码:

configuration.getTypeAliasRegistry().registerAlias(Student.class);

// 2. 指定指定别名注册，可使用以下代码:
configuration.getTypeAliasRegistry().registerAlias("Student",Student.class);

// 3. 通过类的完全限定名注册相应类别名，可使用一下代码:
configuration.getTypeAliasRegistry().registerAlias("Student",
        "com.mybatis3.domain.Student");

// 4. 为某一个包中的所有类注册别名，可使用以下代码:
configuration.getTypeAliasRegistry().registerAliases("com.mybatis3.domain");

// 5. 为在com.mybatis3.domainpackage包中所有的继承自Identifiable类型的类注册别名，可使用以下代码:
configuration.getTypeAliasRegistry().registerAliases("com.mybatis3.domain", Identifiable.class);

```

## 类型处理器 typeHandlers

```java
// 1. 为某个特定的类注册类处理器:
configuration.getTypeHandlerRegistry().register(PhoneNumber.class, PhoneTypeHandler.class);

// 2. 注册一个类处理器:
configuration.getTypeHandlerRegistry().register(PhoneTypeHandler.class);

// 3. 注册com.mybatis3.typehandlers包中的所有类型处理器:
configuration.getTypeHandlerRegistry().register("com.mybatis3.typehandlers");
```

## 全局参数设置 Settings

```java
configuration.setCacheEnabled(true);
configuration.setLazyLoadingEnabled(false);
configuration.setMultipleResultSetsEnabled(true);
configuration.setUseColumnLabel(true);
configuration.setUseGeneratedKeys(false);
configuration.setAutoMappingBehavior(AutoMappingBehavior.PARTIAL);
configuration.setDefaultExecutorType(ExecutorType.SIMPLE);
configuration.setDefaultStatementTimeout(25);
configuration.setSafeRowBoundsEnabled(false);
configuration.setMapUnderscoreToCamelCase(false);
configuration.setLocalCacheScope(LocalCacheScope.SESSION);
configuration.setAggressiveLazyLoading(true);
configuration.setJdbcTypeForNull(JdbcType.OTHER);
Set<String> lazyLoadTriggerMethods = new HashSet<String>();
lazyLoadTriggerMethods.add("equals");
lazyLoadTriggerMethods.add("clone");
lazyLoadTriggerMethods.add("hashCode");
lazyLoadTriggerMethods.add("toString");
configuration.setLazyLoadTriggerMethods(lazyLoadTriggerMethods );

```

## Mappers

```java
// 1. 添加一个Mapper接口，可使用以下代码:
configuration.addMapper(StudentMapper.class);

// 2. 添加com.mybatis3.mappers包中的所有MapperXML文件或者Mapper接口，可使用以下代码:
configuration.addMappers("com.mybatis3.mappers");

// 3. 添加所有com.mybatis3.mappers包中的拓展了特定Mapper接口的Maper接口，如aseMapper,可使用如下代 码:
configuration.addMappers("com.mybatis3.mappers", BaseMapper.class);

```


> Mappers应该在typeAliases和typeHandler注册后再添加到configuration中


