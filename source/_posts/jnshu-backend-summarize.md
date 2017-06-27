---
title: 修真院后端开发从配置到项目结构
date: 2017-06-27 21:32:29
category: 后端
tags: java
---


# 基本配置

1. 首先需要安装好JAVA和Maven，配好环境变量，没什么好说的网上多得是
2. 然后为maven添加公司的私有库，一是下载包的速度快，二是有的项目包只存在于公司私有库
2.1 在个人文件夹下打开隐藏文件夹`.m2`，mac电脑应该在`~/m2/`
2.2 加入或修改setting.xml文件：

```XML
<settings>
  <servers>
    <server>
      <id>nexus</id>
      <username>deployment</username>
      <password>ptteng</password>
      <filePermissions>664</filePermissions>
      <directoryPermissions>775</directoryPermissions>
      <configuration></configuration>
    </server>
    <server>
      <id>central-proxy</id>
      <username>deployment</username>
      <password>{xxxxx}</password>
      <filePermissions>664</filePermissions>
      <directoryPermissions>775</directoryPermissions>
      <configuration></configuration>
  </server>   
  </servers>
  <mirrors>
    <mirror>
      <id>central-proxy</id>
      <mirrorOf>central</mirrorOf>
      <url>http://xxxx.ptteng.com/xxxx/xxx/xxxx/central</url>
    </mirror>        
</mirrors>
<profiles>
    <profile>
      <id>my-repository</id>
      <activation>
          <activeByDefault>true</activeByDefault>
      </activation>
      <repositories>
        <repository>
            <id>nexus</id>
            <name>gemantic's nexus</name>    
            <releases>
                <enabled>true</enabled>
                <updatePolicy>always</updatePolicy>
                <checksumPolicy>warn</checksumPolicy>
            </releases>
            <snapshots>
                <enabled>true</enabled>
                <updatePolicy>always</updatePolicy>
                <checksumPolicy>warn</checksumPolicy>
            </snapshots>
            <url>http://xxxx.ptteng.com/xxx/xxx/xxx/public</url>
            <layout>default</layout>
        </repository> 
      </repositories>  
    </profile>
    <profile>
      <id>my-build</id>
      <activation>
          <activeByDefault>true</activeByDefault>
      </activation>
      <properties> 
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding> 
      </properties> 
    </profile>
</profiles>

</settings>
```


# 以skill为例的配置

1.从SVN拉代码：svn://xxxxx.xxxxx.com/xxxxx/skill
2.这里只知道IDEA应该怎么做，因为我没装eclipse。打开IDEA选择`import project`，并且选择`import project from external model` -> `maven`，之后一路下一步，在代码被加入后，点击右下方的小弹框中选项的自动导入。
这么做的原因是，目前公司项目都是通过maven构建。
3.修改host，添加服务地址
```base
xxx.xxx.xxx.xxx scallop.resource.center
xxx.xxx.xxx.xxx xxxx.mysql.rds.aliyuncs.com #开发环境数据库地址
127.0.0.1 common.skill.service
127.0.0.1 common.skill.admin.service #本地环境当然是请求本地127.0.0.1的service啦
```
这么做的原因是为了调用远程的service方法，当service服务部署在其他地方时，也能通过数据寻找到相应的地址。
如果要查看需要添加哪些地址到host，可以在service的`resources/dbscript/`里查看`xx_resources.sql`文件：
```sql
use resources;
insert into resources(name,resource) values('qa-common-skill-service-rmi','common.skill.service:9012');
```
> 说明这里往数据库的resource表注入了一行数据，`qa-common-skill-service-rmi`是服务名字，`common.skill.service:9012`是服务域名。之后就能通过查询数据库的resource表知道相应的服务需要去哪个域名下寻找。这里之所以不配置IP是因为不同环境下的IP不一样，因此只配置域名，然后在不同环境下只修改host就可以访问不同IP的服务。

4.了解一下一个项目至少有几个部分：
公司项目在目前的架构下一般分为三个部分`core`,`service`,`home`:
- `core`提供数据和服务的抽象模块
- `service`实现数据访问和实现服务
- `home`主要是实现controller层业务。

每个部分的根目录下都有一个`pom.xml`文件，它是maven的配置文件，通过`artifactId`,`groupId`,`version`三个标识符来标明项目的坐标，比如：
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<!--父项目的坐标-->
	<parent>
		<artifactId>skill</artifactId>
		<groupId>ptteng</groupId>
		<version>12.0</version>
		<relativePath>..</relativePath>
	</parent>
	<!--定义项目自身的坐标-->
	<artifactId>skill-admin-home</artifactId>
	<packaging>war</packaging>
	<name>skill-admin-home</name>
	<version>12.0.2-SNAPSHOT</version>
	<!--项目的依赖-->
	<dependencies>
	  <!--依赖的具体项目：ptteng.skill-core.12.0.0，在IDEA下如果maven配置无问题，更改这里的坐标，会去私服自动寻找下载相应项目版本，找不到时会标红，因此项目下来后应该先检查一下pom文件是否有标红的地方-->
		<dependency>
			<groupId>ptteng</groupId>
			<artifactId>skill-core</artifactId>
			<version>12.0.0</version>
		</dependency>
		...
```

5.让项目跑起来：
项目启动、部署的顺序是`core` -> `service` -> `home`
以IDEA为例，具体一点是这样的：

- 展开右侧的`Maven Project`界面，打开`core` -> `Lifecycle`，执行`install`命令，确定core可以构建成功。（这一步只有第一次或者core发生变化时需要执行）
- 然后在`service`项目里找到`/src/main/java/admin/server`文件，右键执行它的`Server.main()`函数，在命令行里查看是否执行成功。
- 最后还是展开右侧的`Maven Project`界面，打开`core` -> `Plugins` -> `jetty`，执行`jetty:run`，在命令行中查看项目是否成功跑起来。
- 如果有报错？那看着报错慢慢找原因咯。。。
> 如果是第一次拉下来的代码，可以在父项目下执行`skill(root)` -> `plugins` -> `install` -> `install:install`查看所有项目能否成功编译



# 以skill为例的项目主要结构
项目结构（假设添加一个user模块需要注意的文件）：
```
|- /core
  |- /src/main/java
    |- /model
      |- User.java -- model层，构建user的数据模型
    |- /service
      |- UserService.java -- 继承dao，user服务接口
    |- /client
      |- UserSCAClient.java -- user的SCA服务接口模型
  |- pom.xml -- maven配置文件
|- /service
  |- /src
    |- /main
      |- /java
        |- /admin
          |- server -- 启动函数
        |- /service.impl
         |- UserServiceImpl.java -- user的具体服务实现
      |- /resources
        |- /dbscript
          |- user.sql -- 存放user表的创建sql语句
          |- user_resources.sql -- 存放user服务的resourse的sql语句
        |- /META-INF
          |- applicationContext-server.xml -- 配置beans，类似于配置依赖注入
          |- server.composite -- SCA的服务组件配置文件
        |- user_dao.xml -- user模块的dao查询语句配置
        |- daoConfig.xml -- 项目下的dao查询模块配置
    |- /test
      |- /java
        |- UserServiceTest.java -- user服务的测试
      |- /resources
        |- applicationContext-server.xml
  |- pom.xml
|- /home
  |- /src/main
    |- /java
      |- /Controller
        |- UserController -- user的controller层
      |- /util -- 一些常用的公共方法
    |- /resources
      |- client.composite -- SCA的客户端配置文件
      |- exception-messages-play.properties -- 接口的错误信息代码表
    |- /webapp
      |- /WEB-INF
        |- /pages
          |- /user/json
            |- userDetailJson.jsp -- 配置userDetail接口输出格式
            |- userListJson.jsp -- 配置userList接口输出格式
        |- /spring
          |- applicationContext-server.xml -- 配置beans
```