- 使用映射器

我们可以使用 MapperFactoryBean 将映射器 Mapper 接口配置成 Spring bean 实体
分别配置每一个映射器Mapper接口是一个非常单调的过程。我们可以使用MapperScannerConfigurer 来扫描包 (package)中的映射器 Mapper 接口，并自动地注册。
```xml

<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer"> <property name="basePackage" value="com.mybatis3.mappers" />
</bean>
```

> MyBatis-Spring-1.2.0 介绍了两种新的扫描映射器 Mapper 接口的方法:
- 使用`<mybatis:scan/>`元素
- 使用`@MapperScan`注解(需Spring3.1+版本)

- `<mybatis:scan />`

`<mybatis:scan>`元素将在特定的以逗号分隔的包名列表中搜索映射器 Mapper 接口。使用这个新的 MyBatis- Spring 名空间你需要添加以下的 schema 声明:

```xml
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mybatis="http://mybatis.org/schema/mybatis-spring" xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://mybatis.org/schema/mybatis-spring
        http://mybatis.org/schema/mybatis-spring.xsd">
    <mybatis:scan base-package="com.mybatis3.mappers" />
</beans>
```


```
<mybatis:scan>元素提供了下列的属性来自定义扫描过程:
- annotation: 扫描器将注册所有的在 base-package 包内并且匹配指定注解的映射器 Mapper 接口。 factory-ref:当 Spring 上下文中有多个 SqlSessionFactory 实例时，需要指定某一特定的
SqlSessionFactory 来创建映射器 Mapper 接口。正常情况下，只有应用程序中有一个以上的数据源
才会使用。
- marker-interface: 扫描器将注册在base-package包中的并且继承了特定的接口类的映射器Mapper接口

- template-ref: 当 Spring 上下文中有多个 SqlSessionTemplate 实例时，需要指定某一特定的
SqlSessionTemplate 来创建映射器 Mapper 接口。正常情况下，只有应用程序中有一个以上的数据源 才会使用。

- name-generator:BeannameGenerator 类的完全限定类名，用来命名检测到的组件。
```


- MapperScan

Spring 框架 3.x+版本支持使用@Configuration 和@Bean 注解来提供基于 Java 的配置。如果你倾向于使用基于 Java 的配置，你可以使用@MapperScan 注解来扫描映射器 Mapper 接口。@MapperScan 和`<mybatis:scan/>`工作方式 相同，并且也提供了对应的自定义选项。

```java

@Configuration
@MapperScan("com.mybatis3.mappers")
public class AppConfig
{
    @Bean
    public DataSource dataSource()
    {
        return new PooledDataSource("com.mysql.jdbc.Driver",
                                    "jdbc:mysql://localhost:3306/elearning", "root", "admin");
    }
    @Bean
    public SqlSessionFactory sqlSessionFactory() throws Exception
    {
        SqlSessionFactoryBeansessionFactory = newSqlSessionFactoryBean();
        sessionFactory.setDataSource(dataSource());
        return sessionFactory.getObject();
} }
```

```
@MapperScan 注解有以下属性供自定义扫描过程使用:
- annotationClass: 扫描器将注册所有的在 base-package 包内并且匹配指定注解的映射器 Mapper 接口。
- markerInterface: 扫描器将注册在 base-package 包中的并且继承了特定的接口类的映射器 Mapper 接口 sqlSessionFactoryRef:当 Spring 上下文中有一个以上的 SqlSesssionFactory 时，用来指定特定
SqlSessionFactory
- sqlSessionTemplateRef: 当 Spring 上下文中有一个以上的 sqlSessionTemplate 时，用来指定特定
sqlSessionTemplate
- nameGenerator:BeanNameGenerator 类用来命名在 Spring 容器内检测到的组件。
- basePackageClasses:basePackages()的类型安全的替代品。包内的每一个类都会被扫描。
- basePackages:扫描器扫描的基包，扫描器会扫描内部的 Mapper 接口。注意包内的至少有一个方法声明的才会被
     注册。具体类将会被忽略。
```

>  与注入 Sqlsession 相比，更推荐使用注入 Mapper，因为它摆脱了对 MyBatis API 的依赖。

- 使用 Spring 进行事务管理

```xml
<bean id="transactionManager"
     class="org.springframework.jdbc. datasource.DataSourceTransactionManager">
   <property name="dataSource" ref="dataSource" />
</bean>
```

> 现在你可以在 Spring service bean 上使用@Transactional 注解，表示在此 service 中的每一个方法都应该在 一个事务中运行。如果方法成功运行完毕，Spring 会提交操作。如果有运行期异常发生，则会执行回滚操作。另外， Spring 会将 MyBatis 的异常转换成合适的 DataAccessExceptions，这样会为特定错误上提供额外的信息。





