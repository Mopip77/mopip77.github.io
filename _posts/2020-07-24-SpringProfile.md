---
layout: post
title: SpringProfile
tags:
- spring
- profile
---

基本内容这里就不介绍了，这里主要介绍一下profile文件的形式和如何指定。

## profile文件形式

### 单体文件
这是一般教程中最常使用的profile文件编写方式，最简单地就是使用yml格式实现
```yml
---
spring:
  profiles: dev

server:
  port: 8081
---
spring:
  profiles: local

server:
  port: 8082
---
spring:
  profiles:
    active: local
```

### 多profile文件
以`application-{profileName}.yml`来创建多个profile配置文件，然后在application.yml中指定生效的profile即可  
这个在配置文件不多的情况下可以使用

### 多profile folder
这个不是spring支持的写法，而是使用了maven的profile以及maven决定如何引入resource文件到classpath下  

我们可以以一种特定文件名格式例如profile_{profileName}来创建不同profile的文件夹
```
-resources (classpath)
    - profile_beta

    - profile_local
        - application.yml
        - log4j2.xml
        - sentry.properties

    - profile_prod
    - profile_test
```
然后里面可以放各种配置文件（所以这个**适合多种不同配置文件都需要迎合不同profile**时使用）  
**配置文件的起名方法需要依照在classpath下的起名方法起名**（例如默认配置文件就是application.yml），原因最后说

并且在pom.xml中指定各个profile以及生效的profile
```xml
<profiles>
    <profile>
        <id>prod</id>
        <properties>
            <profileActive>prod</profileActive>
        </properties>
    </profile>
    <profile>
        <id>local</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <profileActive>local</profileActive>
        </properties>
    </profile>
</profiles>
```
这里为了版面省略了两个文件夹，可以看到这里时默认生效local的profile

还需要指定如何引入resource文件到classpath下
```xml
<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>false</filtering>
            <excludes>
                <exclude>profile_*/*</exclude>
            </excludes>
        </resource>
        <resource>
            <directory>src/main/resources/profile_${profileActive}</directory>
            <filtering>false</filtering>
        </resource>
    </resources>
</build>
```
这样就很清晰了，因为这四个profile文件夹都在classpath下，所以首先我们把所有profile_xxx的文件夹都排除  
之后我们再单独引入生效的profile的文件夹到classpath下，其中`${profileActive}`为maven的变量，也就是我们上面指定的local  

可以简单理解为把profile_local的所有文件复制到resources目录下，再把这四个profile文件夹删除，因此为什么文件夹内文件的命名需要按照默认命名方法命名就很好理解了

最后验证一下，package后是不是把这些配置文件移动到了classpath下，检查一下jar包
发现确实application.yml移动到了classpath下

> BOOT-INF/classes/application.yml