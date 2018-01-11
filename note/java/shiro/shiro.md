## shiro

shiro 功能

认证 授权 加密 回话管理 与web集成 缓存

API

- Authentication  身份认证
- Authorization  授权
- SessionManager  回话管理
- Cryptography  加密
- Web Support  web支持
- Concurrency  多线程并发验证
- Testing 测试支持
- Run As 伪装另一个用户
- Remeber Me 记住我

### Realm

域，Shiro 从从 Realm 获取安全数据(如用户、角色、权限)，就是说 SecurityManager 要验证用户身份，那么它需要从 Realm 获取相应的用户进行比较以确定用户身份是否合法;也需要从 Realm 得到用户相应的角色/权限进行验证用户是否能进行操作;可以把 Realm 看
成 DataSource，即安全数据源。

> 也就是说对于我们而言，最简单的一个 Shiro 应用:
1、 应用代码通过 Subject 来进行认证和授权，而 Subject 又委托给 SecurityManager;
2、 我们需要给 Shiro 的 SecurityManager 注入 Realm，从而让 SecurityManager 能得到合法
  的用户及其权限进行判断。
 

```java
 public class MyRealm1 implements Realm {

  public String getName() {
    return "myrealm1";
  }

  public boolean supports(AuthenticationToken token) {
//仅支持 UsernamePasswordToken 类型的 Token
    return token instanceof UsernamePasswordToken;
  }

  public AuthenticationInfo getAuthenticationInfo(AuthenticationToken token)
      throws AuthenticationException {
    String username = (String) token.getPrincipal(); //得到用户名
    String password = new String((char[]) token.getCredentials()); //得到密码
    if (!"zhang".equals(username)) {
      throw new UnknownAccountException(); //如果用户名错误
      //
    }
    if (!"123".equals(password)) {
      throw new IncorrectCredentialsException(); //如果密码错误
    }
//    如果身份认证验证成功，返回一个 AuthenticationInfo 实现;
    return new SimpleAuthenticationInfo(username, password, getName());
  }
}
```

### 多 Realm 配置
shiro-multi-realm.ini

```
#声明一个 realm 
myRealm1=cn.lirawx.shiro.realm.MyRealm1 
myRealm2=cn.lirawx.shiro.realm.MyRealm2 
#指定 securityManager 的 realms 实现 
securityManager.realms=$myRealm1,$myRealm2

```
> securityManager.realm 显示指定realm执行顺序



### 身份验证
 

身份验证的步骤:

- 1、收集用户身份/凭证，即如用户名/密码;
- 2、调用 Subject.login 进行登录，如果失败将得到相应的 AuthenticationException 异常，根 据异常提示用户错误信息;否则登录成功;
- 3、最后调用 Subject.logout 进行退出操作。

```java
@Test
  public void test() {
    //1、获取 SecurityManager 工厂，此处使用 Ini 配置文件初始化SecurityManager

    Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro.ini");
    //2、得到 SecurityManager 实例 并绑定给 SecurityUtils

    org.apache.shiro.mgt.SecurityManager securityManager = factory.getInstance();
    SecurityUtils.setSecurityManager(securityManager);
    //3、得到 Subject 及创建用户名/密码身份验证 Token(即用户身份/凭证)

    Subject subject = SecurityUtils.getSubject();
    UsernamePasswordToken token = new UsernamePasswordToken("zhang", "123");
    try { //4、登录，即身份验证
      subject.login(token);
    } catch (AuthenticationException e) { //5、身份验证失败
    }
    Assert.assertEquals(true, subject.isAuthenticated()); //断言用户已经登录
//6、退出
    subject.logout();
  }
```
 

### JDBC Realm 使用

> 到数据库 shiro 下建三张表:user(s 用户名/密码)、user_role(s 用户/角色)、roles_permissions (角色/权限)，具体请参照 shiro-example-chapter2/sql/shiro.sql;并添加一个用户记录，用 户名/密码为 zhang/123

```sql

drop database if exists shiro;
create database shiro;
use shiro;

create table users (
  id bigint auto_increment,
  username varchar(100),
  password varchar(100),
  password_salt varchar(100),
  constraint pk_users primary key(id)
) charset=utf8 ENGINE=InnoDB;
create unique index idx_users_username on users(username);

create table user_roles(
  id bigint auto_increment,
  username varchar(100),
  role_name varchar(100),
  constraint pk_user_roles primary key(id)
) charset=utf8 ENGINE=InnoDB;
create unique index idx_user_roles on user_roles(username, role_name);

create table roles_permissions(
  id bigint auto_increment,
  role_name varchar(100),
  permission varchar(100),
  constraint pk_roles_permissions primary key(id)
) charset=utf8 ENGINE=InnoDB;
create unique index idx_roles_permissions on roles_permissions(role_name, permission);

insert into users(username,password)values('zhang','123');
```

shiro-jdbc-realm.ini


```ini
jdbcRealm=org.apache.shiro.realm.jdbc.JdbcRealm dataSource=com.alibaba.druid.pool.DruidDataSource dataSource.driverClassName=com.mysql.jdbc.Driver dataSource.url=jdbc:mysql://localhost:3306/shiro dataSource.username=root
#dataSource.password= jdbcRealm.dataSource=$dataSource securityManager.realms=$jdbcRealm
```

### Authenticator 及 AuthenticationStrategy

SecurityManager 接口继承了 Authenticator，另外还有一个 ModularRealmAuthenticator 实现， 其委托给多个 Realm 进行验证，验证规则通过 AuthenticationStrategy 接口指定，默认提供 的实现:

- FirstSuccessfulStrategy:只要有一个 Realm 验证成功即可，只返回第一个 Realm 身份验证 成功的认证信息，其他的忽略;
- AtLeastOneSuccessfulStrategy:只要有一个 Realm 验证成功即可，和 FirstSuccessfulStrategy 不同，返回所有 Realm 身份验证成功的认证信息;
- AllSuccessfulStrategy:所有 Realm 验证成功才算成功，且返回所有 Realm 身份验证成功的 认证信息，如果有一个失败就失败了。


## 授权

授权，也叫访问控制，即在应用中控制谁能访问哪些资源(如访问页面/编辑数据/页面操作 等)。在授权中需了解的几个关键对象:主体(Subject)、资源(Resource)、权限(Permission)、 角色(Role)。

- 主体

> 主体，即访问应用的用户，在 Shiro 中使用 Subject 代表该用户。用户只有授权后才允许访 问相应的资源。

- 资源

> 在应用中用户可以访问的任何东西，比如访问 JSP 页面、查看/编辑某些数据、访问某个业 务方法、打印文本等等都是资源。用户只要授权后才能访问。


- 权限

> 安全策略中的原子授权单位，通过权限我们可以表示在应用中用户有没有操作某个资源的 权力。即权限表示在应用中用户能不能访问某个资源，如:
访问用户列表页面
查看/新增/修改/删除用户数据(即很多时候都是 CRUD(增查改删)式权限控制) 打印文档等等。。。
如上可以看出，权限代表了用户有没有操作某个资源的权利，即反映在某个资源上的操作 允不允许，不反映谁去执行这个操作。所以后续还需要把权限赋予给用户，即定义哪个用 户允许在某个资源上做什么操作(权限)，Shiro 不会去做这件事情，而是由实现人员提供。
Shiro 支持粗粒度权限(如用户模块的所有权限)和细粒度权限(操作某个用户的权限，即 实例级别的)，后续部分介绍。

- 角色 

> 角色代表了操作集合，可以理解为权限的集合，一般情况下我们会赋予用户角色而不是权 限，即这样用户可以拥有一组权限，赋予权限时比较方便。典型的如:项目经理、技术总 监、CTO、开发工程师等都是角色，不同的角色拥有一组不同的权限。 隐式角色:即直接通过角色来验证用户有没有操作权限，如在应用中 CTO、技术总监、开 发工程师可以使用打印机，假设某天不允许开发工程师使用打印机，此时需要从应用中删 除相应代码;再如在应用中 CTO、技术总监可以查看用户、查看权限;突然有一天不允许 技术总监查看用户、查看权限了，需要在相关代码中把技术总监角色从判断逻辑中删除掉; 即粒度是以角色为单位进行访问控制的，粒度较粗;如果进行修改可能造成多处代码修改。 显示角色:在程序中通过权限控制谁能访问某个资源，角色聚合一组权限集合;这样假设 哪个角色不能访问某个资源，只需要从角色代表的权限集合中移除即可;无须修改多处代 码;即粒度是以资源/实例为单位的;粒度较细。

### 授权方式

- 编程式:通过写 if/else 授权代码块完成:

```java
Subject subject = SecurityUtils.getSubject(); if(subject.hasRole(“admin”)) {
//有权限 
} else {
//无权限 
}
```

- 注解式:通过在执行的 Java 方法上放置相应的注解完成:

```java
@RequiresRoles("admin") public void hello() {
//有权限 
}
```
> 没有权限将抛出相应的异常

- SP/GSP 标签:在 JSP/GSP 页面通过相应的标签完成:

```java
<shiro:hasRole name="admin"> <!— 有权限 —> </shiro:hasRole>
```

### 授权

#### 基于角色的访问控制(隐式角色)

- ini 配置文件配置用户拥有的角色(shiro-role.ini)

```ini
[users] 
zhang=123,role1,role2 
wang=123,role1
```

> 规则即:“用户名=密码,角色 1，角色 2”，如果需要在应用中判断用户是否有相应角色， 就需要在相应的 Realm 中返回角色信息，也就是说 Shiro 不负责维护用户-角色信息，需要 应用提供，Shiro 只是提供相应的接口方便验证，后续会介绍如何动态的获取用户角色


Shiro 提供了 hasRole/hasRole 用于判断用户是否拥有某个角色/某些权限;但是没有提供如 hashAnyRole 用于判断是否有某些权限中的某一个。

Shiro 提供的 checkRole/checkRoles 和 hasRole/hasAllRoles 不同的地方是它在判断为假的情 况下会抛出 UnauthorizedException 异常。


- 基于资源的访问控制(显示角色)


ini 配置文件配置用户拥有的角色及角色-权限关系(shiro-permission.ini)

```ini
[users]
zhang=123,role1,role2 
wang=123,role1
[roles] 
role1=user:create,user:update role2=user:create,user:delete
```

> 规则:“用户名=密码，角色 1，角色 2” “角色=权限 1，权限 2”，即首先根据用户名找 到角色，然后根据角色再找到权限;即角色是权限集合;Shiro 同样不进行权限的维护，需 要我们通过 Realm 返回相应的权限信息。只需要维护“用户——角色”之间的关系即可。


hiro 提供了 isPermitted 和 isPermittedAll 用于判断用户是否拥有某个权限或所有权限，也 没有提供如 isPermittedAny 用于判断拥有某一个权限的接口。


## 加密

### 散列算法

散列算法一般用于生成数据的摘要信息，是一种不可逆的算法，一般适合存储密码之类的 数据，常见的散列算法如 MD5、SHA 等，一般进行散列时最好提供一个 salt(盐)。

> 即如果直接对密码进行散列相 对来说破解更容易，此时我们可以加一些只有系统知道的干扰数据，如用户名和 ID(即盐); 这样散列的对象是“密码+用户名+ID”，这样生成的散列值相对来说更难破解。

```java
String str = "hello";
String salt = "123";
String md5 = new Md5Hash(str, salt).toString();//还可以转换为 toBase64()/toHex()
```


## 默认拦截器

|默认拦截器名|拦截器类|说明|
|----------|-------|---|
|authc|org.apache.shiro.web.filter. authc.FormAuthentication Filter|基于表单的拦截器;如“/**=authc”，如 果没有登录会跳到相应的登录页面登录; 主要属性:usernameParam:表单提交的 用户名参数名( username ); passwordParam:表单提交的密码参数名 (password); rememberMeParam:表 单提交的密码参数名(rememberMe); loginUrl:登录页面地址(/login.jsp); successUrl:登录成功后的默认重定向地 址; failureKeyAttribute:登录失败后错 误信息存储 key(shiroLoginFailure);|
|authcBasic|org.apache.shiro.web.filter. authc.BasicHttpAuthentica tionFilter|Basic HTTP 身份验证拦截器，主要属性: applicationName:弹出登录框显示的信息 (application);|
|logout|org.apache.shiro.web.filter. authc.LogoutFilter|退出拦截器，主要属性:redirectUrl:退 出成功后重定向的地址(/);示例 “/logout=logout”|
|user|org.apache.shiro.web.filter. authc.UserFilter|用户拦截器，用户已经身份验证/记住我 登录的都可;示例“/**=user”|
|anon|org.apache.shiro.web.filter. authc.AnonymousFilter|匿名拦截器，即不需要登录即可访问;一 般用于静态资源过滤;示例 “/static/**=anon”|
|授权相关|
|roles|org.apache.shiro.web.filter. authz.RolesAuthorizationF ilter|角色授权拦截器，验证用户是否拥有所有 角色;主要属性: loginUrl:登录页面地 址(/login.jsp);unauthorizedUrl:未授 权后重定向的地址;示例 “/admin/**=roles[admin]”|
|perms|org.apache.shiro.web.filter. authz.PermissionsAuthoriz ationFilter|权限授权拦截器，验证用户是否拥有所有 权限;属性和 roles 一样;示例 “/user/**=perms["user:create"]”|
|port|org.apache.shiro.web.filter. authz.PortFilter|端口拦截器，主要属性:port(80):可 以通过的端口;示例“/test= port[80]”， 如果用户访问该页面是非 80，将自动将 请求端口改为 80 并重定向到该 80 端口， 其他路径/参数等都一样|
|rest|org.apache.shiro.web.filter. authz.HttpMethodPermissi onFilter|rest 风格拦截器，自动根据请求方法构建 权限字符串( GET=read, POST=create,PUT=update,DELETE=delet e,HEAD=read,TRACE=read,OPTIONS=re ad, MKCOL=create)构建权限字符串;示 例“/users=rest[user]”，会自动拼出 “ user:read,user:create,user:update,user:del ete”权限字符串进行权限匹配(所有都得 匹配，isPermittedAll);|
|ssl|org.apache.shiro.web.filter. authz.SslFilter|SSL 拦截器，只有请求协议是 https 才能 通过;否则自动跳转会 https 端口(443); 其他和 port 拦截器一样;|
|其他|
|noSessionCreation|org.apache.shiro.web.filter. session.NoSessionCreation Filter|不创建会话拦截器，调用 subject.getSession(false)不会有什么问题， 但是如果 subject.getSession(true)将抛出 DisabledSessionException 异常;|