---
title: Maven从入门到入坑
top: false
cover: false
toc: true
mathjax: true
date: 2021-07-12 16:13:59
password:
summary:
categories: 敏捷开发
tags: 
	- Maven
	- IDEA
---

# 学习Maven的基础

目前所有的项目项目都在使用Maven，那到底Maven是什么，为什么Maven可以大行其道，它究竟为我们的项目开发带来了怎样的好处呢——

## 为什么要学习Maven

1. 在开发庞大的项目时，使用package的结构来区分复杂的代码结构就显得力不从心了，这时我们需要引入**模块(Module)**的设计理念。而Maven就使用了这一设计理念；
2. 每个项目下导入重复的jar包，不符合"write once,only once"的追求；
3. 手动导包时可能会导致漏包，版本不一致，导致项目未执行错误；

## Maven是什么

Maven的核心思想——

> Maven的核心思想：约定大于配置，它用于约定java项目架构，并推出了各种各样的模板项目。

## 使用Maven的好处

1. 通俗的说，Maven可以通过对**pom.xml**的配置实现**自动导包**，且可以解决令人头大的jar包冲突，版本冲突，且可以实现一次配置，多次复用的目的；
2. Maven模块管理的功能使项目结构更加明晰，通过降低模块间的耦合度使项目的再开发变得容易；
3. 自动生成包含测试的目录结构就很香；
4. 通常我们的项目在整合时会包括庞杂的代码，配置文件，打包。甚至web工程的话还需要部署到服务器上。而Maven却可以帮我们构建工程，管理jar包，编译运行测试文件，自动打包，生成*报表*。堪称程序员的贴心小棉袄。
5. 使用Maven构建的项目，导入的jar包只是一个**坐标**。大大减少了打包后的项目大小。

# Maven的使用

了解了Maven的概况，你心动了吗？

接下来我们首先将进行Maven的安装，因为Maven包含构建的功能，这就需要JAVA_HOME的环境变量配置，当我们安装好Maven后，就要考虑jar包如何获取呢，这时就需要引入Maven仓库。另外，Maven是如何使项目结构令人耳目一新的呢，常说的生命周期又是什么？让我们一探究竟——

------

## Maven的安装与配置

![](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20200810201856834.png)

1. 因为Maven是服务于java语言的，首先需要查看系统是否配置了JAVA_HOME的环境变量，也就是jdk的路径，即java的运行环境；

2. 配置Maven的相关环境变量

   1. 首先Maven支持一键构建的功能，因而需要配置环境变量**PATH**，此外，Maven安装路径(*无中文，无空格的目录下*)**MAVEN_HOME**的说明也必不可少！
   2. ![image-20200810202557992](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20200810202557992.png)

   ![image-20200810202753425](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20200810202753425.png)

   1. 验证是否配置成功

      键入**mvn  -v**命令查看maven版本，存在则配置成功！！

## Maven仓库

> Maven最为人所知的功能便是**自动导包**。那导的jar包来源于哪里呢？Maven仓库便是存储jar包的地方。

### Maven仓库的分类

Maven仓库分为本地仓库，远程仓库，中央仓库。

- 本地仓库：默认位置为C:\Users\zyz\.m2\repository(mine)，也可自己指定仓库位置。
- 远程仓库：又称私服，当本地仓库找不到jar包或者插件，会默认从远程仓库获取，远程仓库可以在互联网上，也可以在局域网(公司)中；
- 中央仓库：在Maven软件中内置了一个远程仓库的地址，http://repo1.maven.org/maven2，它由Maven团队维护，提供了全面的jar包，其中包含了世界和是哪个大部分流行的开源项目软件。

### Maven仓库的配置

打开MAVE_HOME/conf目录

![image-20200810205434607](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20200810205434607.png)

在settings.xml文件中配置本地仓库位置

![image-20200810205701220](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20200810205701220.png)

### Maven仓库的使用顺序

![image-20200817104957595](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20200817104957595.png)

###  全局setting与用户setting

Maven的仓库地址，私服等的配置信息需要在setting.xml中配置，而它的配置又分为全局setting和用户setting。

- 见名知意，用户setting，指的是在用户文件夹下的setting，即：${user.dir} /.m2/settings.xml，它里面的配置主要是个性配置；

- 全局setting，即为Maven安装目录下的setting，如D:\Maven\apache-maven-3.6.3\conf\settings.xml。它里面的配置包括使用Maven构建的所有项目

  **小结**：Maven会先找用户配置，如果找不到，就使用全局配置。

## Maven的目录结构

- [x] 注：建议将所有的开发放在一个文件夹**develop**中

Maven工程**约定俗成**的目录结构如下图所示——

<img src="https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20200811000632615.png" alt="image-20200811000632615" style="zoom: 67%;" />

目录的每一部分都是不可或缺的。

当项目编译之后，会自动生成***target***目录，目录中存放编译后的class字节码文件以及配置文件。

## Maven的常用命令

- mvn clean：清理之前编译得到的文件

- mvn compile：编译主程序

- mvn test-compile：编译测试程序

- mvn test：执行测试

- mvn package：打包

- mvn install：安装

- mvn deploy:部署

  这里只演示compile和install命令——

  执行compile编译命令，编译指将src目录下的java文件编译为class文件，则首先定位到到项目的src目录，再执行mvn  compile命令。

  ![](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20200811003234622.png)

<img src="https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20200811003633190.png" alt="image-20200811003633190" style="zoom:50%;" />

## Maven的生命周期

Maven对项目的构建分为**三套**相互独立的**生命周期**

1. **Clean Lifecycle** :在进行真正的构建之前进行一些清理工作。
2. **Default Lifecycle** :构建的核心部分，编译，测试，打包，部署等等。
3. **Site Lifecycle** :生成项目报告，站点，发布站点。

# Maven的依赖管理

Maven在pom.xml中对依赖进行配置与管理，那么pom.xml中的依赖配置要符合怎样的规范才能从仓库中取得jar包呢，当使用一个jar包时，它所依赖的jar包还需要导入吗？如果不需要，那因为版本引发的冲突问题如何解决呢？我们来一一解惑——

------



## Maven的坐标

在pom.xml文件中，将可以找到仓库中对应jar位置的规范叫做**坐标**，坐标在<dependencies>标签下定义，并将每一个第三方依赖写在<dependency>标签下，我们首先来看下坐标的结构——

```java
 <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
 </dependency>
```

- groupId:公司，个人的唯一标识符，通常为域名倒写；

- artifactId:项目唯一标识符，即项目名称

- version:项目的版本，一般来说，使用SVN版本控制工具管理版本，因而一个项目有多个版本，版本的标识有RELEASE，SNAPSHOT

  - **RELEASE**：指正式发布的版本
  - **SNAPSHOT**：表示正在开发的版本，也就是一个版本的**快照**，快照版本，可以理解为它指定了**某个开发进度的副本**，Maven每次构建的时候都会在在仓库中检查新的快照版本。

- **scope**:每个jar包在编译时应用的时期是不同的，因为需要使用scope来指定**依赖范围**。scope有四个常用的取值，分别是compile(编译阶段)，test(测试阶段)，provide(供应阶段),runtime(运行阶段)——

  |     依赖范围      | 编译有效 | 测试有效 | 运行有效 | 打包有效 |       示例        |
  | :---------------: | :------: | :------: | :------: | :------: | :---------------: |
  | compile(**默认**) |    √     |    √     |    √     |    √     |    spring-core    |
  |       test        |    ×     |    √     |    ×     |    ×     |       junit       |
  |     provided      |    √     |    √     |    √     |    ×     | javax.servlet-api |
  |      runtime      |    ×     |    √     |    √     |    √     |     JDBC驱动      |

##  依赖冲突

首先来了解冲突是如何产生的呢？

> 当我们首先引入spring-mvc的坐标后，我们发现spring-mvc所依赖的spring-beans的坐标也被引入了进来，这被称为**依赖传递**。这固然减轻了我们逐个导包的繁复，但也由于**版本冲突**导致坐标不可用。

解决依赖冲突的的调解原则有三种——

1. 第一声明者优先：如果两个包的依赖都包括另一个包，但需要的版本不同，这时以先导入的版本为准；
2. 路径近者优先：也就是说如果先导入某一个包，再导入依赖这个的包的其他包，会以顺序在前的为准；
3. 锁定版本：在pom.xml中，有一个properties标签，它的作用是锁定版本。它的用法如下——

```java
<properties>
        <spring.version>5.0.2.RELEASE</spring.version>
        <slf4j.version>1.6.6</slf4j.version>
        <log4j.version>1.2.12</log4j.version>
        <oracle.version>12.1.0.1</oracle.version>
        <mybatis.version>3.4.5</mybatis.version>
        <spring.security.version>5.0.1.RELEASE</spring.security.version>
</properties>
```

# 在IDEA环境下，使用Maven构建SSM工程

当开发比较繁琐的项目时，Maven自动导包和一键构建特性的优势可见一斑——

首先，回顾一下开发项目的基本原则

1. 先从**被依赖的，简单的部分**开始；
2. 搭好**框架**是重中之重！框架中应该包含开发相关配置文件，分层结构等。

接下来我们总结开发流程

1. 新建使用**webapp骨架**的项目；
2. 在**pom.xm**文件中编写依赖，**applicationContext**文件，**spring-mvc.xml**，**log.properties**文件，导入**db.properties**文件。
3. 在数据库新建表，创建**pojo实体类**，编写**xml**映射文件，**dao**层，**service**层，**controller**层，**web**层；

![image-20200818233219851](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20200818233219851.png)

2.填写项目

![image-20200818233351689](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20200818233351689.png)

![image-20200818233900433](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20200818233900433.png)

首先编写pom.xml文件,除了这部分有区别外，其他的可以复用

```java
<groupId>com.zyz</groupId>
  <artifactId>mavenTest02</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>war</packaging>

  <!--版本锁定-->
  <properties>
    <spring.version>5.0.2.RELEASE</spring.version>
    <slf4j.version>1.6.6</slf4j.version>
    <log4j.version>1.2.12</log4j.version>
    <shiro.version>1.2.3</shiro.version>
    <oracle.version>12.1.0.1</oracle.version>
    <mybatis.version>3.4.5</mybatis.version>
    <spring.security.version>5.0.1.RELEASE</spring.security.version>
  </properties>
```

在新建的resources目录下统一存放配置文件

![image-20200818234706165](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20200818234706165.png)

![image-20200818234915712](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20200818234915712.png)

到这里，项目的骨架就已经搭好了，这是最基础，但同时也是最重要的一步。接下来具体编写的内容依需求而易，这里就不再赘述。

写到这里，我们已经了解Maven的全貌了吗？当然不是,Maven的进阶功能——继承与聚合同样值得一谈。

## Maven的继承与聚合

在开发中，如果几个互相依赖的类存在重复的代码，我们会将其提取到父类，在模块中，我们使用相同的思想将其提取到父工程中。同时，为了代码的二次开发，模块的设计也应运而生。我们先来分析一下常见的开发结构——

![image-20200819082820315](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20200819082820315.png)

### 理解继承和聚合

1. 何为继承？

> 继承是为了消除重复，如果将dao，service，web分开创建独立的工程，则每个工程的pom.xml文件中的内容存在重复。比如，设置编译版本，锁定spring的版本等，可以将这些重复的配置剔除出来在父工程的pon.xml中定义。

2. 何为聚合？  

> 项目开发通常是分组分模块开发，每个模块开发完成要运行整个工程需要将每个模块聚合在一起运行，比如dao,service,web三个工程最终回答一个独立的war运行。

## 补充

### 虚拟路径映射

流程：配置IDEA中的maven，配置tomcat服务器，在context处可选自定义的虚拟路径，最终部署项目。

### 可能出现的问题

| 序号 | 可能出现的问题              | 解决方式                   |
| ---- | --------------------------- | -------------------------- |
| 1    | maven3.6.2显示导包错误      | `版本降级`                 |
| 2    | 每个项目都要重新配置maven   | `配置maven的全局设置`      |
| 3    | Maven项目中的tomcat无法访问 | `配置，server->deployment` |
| 4    | 资源导出失败                | `在builder中配置resource`  |

------

> 山高路远，静水深流

