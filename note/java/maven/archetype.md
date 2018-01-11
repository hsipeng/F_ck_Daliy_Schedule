## 构建archetype 工程

```xml
<plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-archetype-plugin</artifactId>
        <version>3.0.0</version>
      </plugin>
```

```bash
cd E:\workspacesnew\luna-zjl\maven-simple-webapp
mvn archetype:create-from-project

# 进入到生成的archetype目录
cd target\generated-sources\archetype
# 将archetype安装到本地
mvn install

# 如果需要部署到网上则需要添加distributionManagement配置，修改当前目录下面的pom.xml文件添加类似下面的配置。
#   <distributionManagement>
#       <snapshotRepository>
#           <id>mynexus</id>
#           <url>http://nexus.zhaojunling.me/content/repositories/snapshots/</url>
#       </snapshotRepository>
#   </distributionManagement>
mvn deploy  # 部署到远程服务器，供其他人使用

# 最后执行下面操作更新本地的archetype-catalog.xml
mvn archetype:crawl

```

