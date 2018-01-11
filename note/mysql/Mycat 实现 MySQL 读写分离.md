## 环境
接 docker mysql 主从复制

实验拓扑：

        mycat
        /    \
  master -- slave (主从复制)

```bash
mycat: 172.17.0.1
master: 172.17.0.2
slave: 172.17.0.3

```


Mycat 提供了编译好的安装包，下载地址：[http://dl.mycat.io](http://dl.mycat.io)

```
Index of /

../
1.6-RELEASE/                                      28-Oct-2016 12:56                  -
1.6.5-DEV/                                        15-Jan-2017 07:10                  -
2.0-dev/                                          02-Jan-2017 07:24                  -
mycat-web-1.0/                                    02-Jan-2017 07:40                  -
yum/                                              18-May-2016 02:51                  -
Mycat-server-1.4-beta-20150604171601-linux.tar.gz  27-Jun-2015 10:09            7663894
apache-maven-3.3.3-bin.tar.gz                      27-Jun-2015 10:09            8042383
apache-tomcat-7.0.62.tar.gz                        27-Jun-2015 10:09            8824528
jdk-7u79-linux-x64.tar.gz                          27-Jun-2015 10:09          153512879
jdk-8u20-linux-x64.tar.gz                          27-Jun-2015 10:09          160872342
phpMyAdmin-4.4.9-all-languages.tar.gz              27-Jun-2015 10:09            9352049
probe-2.3.3.zip                                    27-Jun-2015 10:09            7957290
toolset.sh                                        26-Oct-2015 05:03              16015
zookeeper-3.4.6.tar.gz

```


下载

```bash
wget http://dl.mycat.io/1.6-RELEASE/Mycat-server-1.6-RELEASE-20161028204710-linux.tar.gz

```

Mycat-server 包解压后可直接使用。

```bash
tar -xf Mycat-server-1.6-RELEASE-20161028204710-linux.tar.gz -C /usr/local/
```

目录结构:

```bash
[root@vm2 local]# tree -L 1 mycat
mycat
|-- bin
|-- catlet
|-- conf
|-- lib
|-- logs
-- version.txt

5 directories, 1 file
```

> bin 目录中是可执行文件以及脚本，我们可以使用其中的 mycat 脚本控制mycat的启动和关闭。
conf 目录中是配置文件，这里配置读写分离主要使用 schema.xml 和 server.xml。其他配置分片的配置请参考官方文档。
logs 目录存放日志文件，遇到mycat出错了，就在这里查看问题的原因。


这里只讲解一下读写分离用到的配置文件：server.xml, schema.xml。

前提：已经有一个配置好的 mysql 一主一从架构。

一个主从集群在Mycat里面由一个 dataNode 定义，dataNode 定义了一个数据库实例及其中的一个具体的库。Mycat 的一个数据库实例可以实际上是一个主从复制架构：一主多从，一主一从，多主多从等等，具体在 dataHost 中定义。

这里建立一个非拆分库（将mycat逻辑库绑定到一个具体的 dataNode 上）testdb，绑定到 dn1 这个 dataNode 上。

schema.xml:

```xml
<schema name="testdb" checkSQLschema="false" sqlMaxLimit="100" dataNode="dn1">

</schema>

```

现在所有的表会走默认的节点 dn1。逻辑库 testdb，对应了数据节点 dn1。dn1 对应着真实的数据库实例上的一个真实的库。

```xml
<dataNode name="dn1" dataHost="vm3306" database="db1" >
</dataNode>


```

> dataNode 标签定义了 MyCat 中的数据节点,也就是我们通常说所的数据分片。一个 dataNode 标签就是 一个独立的数据分片。
例子中所表述的意思为:使用名字为 vm3306 数据库实例上的 db1 物理数据库,这就组成一个数据分片,最 后,我们使用名字 dn1 标识这个分片。
该属性用于定义该分片属性哪个具体数据库实例上的具体库,因为这里使用两个纬度来定义分片,就是:实 例+具体的库。因为每个库上建立的表和表结构是一样的。所以这样做就可以轻松的对表进行水平拆分。
dataHost: 包含一个 writeHost 和 一个 readHost，它们之前已经配置好主从复制了。
balance="3"：表示写请求只发给节点，读请求只发给读节点。


```xml

<dataHost name="vm3306" maxCon="1000" minCon="10" balance="3" writeType="0" dbType="mysql" dbDriver="native">

    <heartbeat>select user()</heartbeat>

    <!-- can have multi write hosts -->

    <writeHost host="hostM1" url="192.168.0.120:3306" user="tuser" password="guli123">
        <!-- can have multi read hosts -->
        <readHost host="hostS1" url="192.168.0.122:3306" user="tuser" password="guli123"/>
    </writeHost>

    <!-- <writeHost host="hostM2" url="localhost:3316" user="tuser" password="guli123"/> -->
</dataHost>

```

> user 及 password属性是后端主从mysql的账户密码信息。

dataHost属性说明：
- writeType="0", 所有写操作发送到配置的第一个 writeHost,第一个挂了切到还生存的第二个 writeHost,重新启动后已切换后的为准,切换记录在配置文件中:dnindex.properties .
- balance="3",所有读请求随机的分发到 wiriterHost 对应的 readhost 执行,writerHost 不负担读压力,注意 balance=3 只在 1.4 及其以后版本有,1.3 没有。


```xml
  <user name="test">
            <property name="password">root</property>
            <property name="schemas">testdb</property>
    </user>

    <user name="user">
            <property name="password">user</property>
            <property name="schemas">testdb</property>
            <property name="readOnly">true</property>
    </user>
```
> 这边定义mycat 的用户 配置相应的权限


启动mycat

```bash
cd /usr/local/mycat/bin
./mycat start
```


### Mycat 管理命令与监控

登录:目前 mycat 有两个端口,8066 数据端口,9066 管理端口,命令行的登陆是通过 9066 管理端口来操 作,登录方式类似于 mysql 的服务端登陆。

```bash
mysql -h127.0.0.1 -utest -ptest -P9066 [-dmycat]
-h 后面是主机,即当前 mycat 按照的主机地址,本地可用 127.0.0.1 远程需要远程 ip -u Mycat server.xml 中配置的逻辑库用户
-p Mycat server.xml 中配置的逻辑库密码
-P 后面是端口 默认 9066,注意 P 是大写
-d Mycat server.xml 中配置的逻辑库

```
```
mysql -h127.0.0.1 -uroot -p123456 -P9066

```


从 9066 管理端口登陆后，执行 show @@help 可以查看到所有帮助命令

> 其他主要和mysql 差不多，就不多说了。
