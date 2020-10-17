---
title: maven简单介绍
date: 2020-10-15 21:27:25
tags:
  - Maven
---



## 简介

1. Maven 翻译为"专家"、"内行"，是 Apache 下的一个纯 Java 开发的开源项目。基于项目对象模型（缩写：POM）概念，Maven利用一个中央信息片断能管理一个项目的构建、报告和文档等步骤。

2. Maven 是一个项目管理工具，可以对 Java 项目进行构建、依赖管理。

3. Maven 也可被用于构建和管理各种项目，例如 C#，Ruby，Scala 和其他语言编写的项目。Maven 曾是 Jakarta 项目的子项目，现为由 Apache 软件基金会主持的独立 Apache 项目。

    注：摘自[菜鸟教程]:https://www.runoob.com/maven/maven-tutorial.html

### 使用原因

#### 减少冗余

在项目开发中，我们常常需要引入第三方的框架和工具包来提高开发速度，以前要使用这些 jar 包就是复制粘贴到 WEB-INF/lib 目录下。当你建了多个项目时有项目都需要引入同一jar包，需要将 jar 包重复复制到 lib 目录下，从而造成工作区中存在大量重复的文件，使工程显得很臃肿。 

使用maven后会在本地建一个统一的仓库，用于存放所有jar包、源文件等，当项目需要引入某个jar包时，只需复制其依赖坐标至pom文件中，maven就会自动帮我们导入到项目中。每个jar包本地只保存一份，不仅极大的节约了存储空间，让项目更轻巧，更避免了重复文件太多而造成的混乱。

#### 解决jar包依赖

jar包往往不是孤立存在的，很多 jar 包都需要在其他 jar 包的支持下才能够正常工作，我们称之为 jar 包之间的依赖关系。Maven 就可以替我们自动的将当前 jar 包所依赖的其他所有 jar 包全部导入进来， 无需人工参与，节约了我们大量的时间和精力。

#### 统一下载

我们通常在查找第三方jar包，选择版本上花费很多时间，maven提供个中央仓库，只需你依赖坐标填对，就会自动去 [中央仓库]:https://mvnrepository.com/  下载。

#### 插件

maven有很多插件，便于功能扩展，比如生产站点，自动发布版本，打包项目等



## maven项目构建过程的几个主要环节 

1. 清理：删除以前的编译结果，为重新编译做好准备。 
2. 编译：将 Java 源程序编译为字节码文件。
3. 测试：针对项目中的关键点进行测试，确保项目在迭代开发过程中关键点的正确性。 
4. 报告：在每一次测试后以标准的格式记录和展示测试结果。 
5. 打包：将一个包含诸多文件的工程封装为一个压缩文件用于安装或部署。Java 工程对应 jar 包，Web 工程对应 war 包。 
6. 安装：在 Maven 环境下特指将打包的结果——jar 包或 war 包安装到本地仓库中。 
7. 部署：将打包的结果部署到远程仓库或将 war 包部署到服务器上运行。 



## pom文件常用标签介绍

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

	<!--声明项目描述符遵循哪一个POM模型版本。为了当Maven引入了新的特性或者其他模型变更的时候，确保稳定性。 -->
    <modelVersion>4.0.0</modelVersion>
	<!--父项目的坐标。如果项目中没有规定某个元素的值，那么父项目中的对应值即为项目的默认值。-->
	<parent>
        <!--父项目的构件标识符 -->
        <groupId>org.springframework.boot</groupId>
        <!--父项目的全球唯一标识符(项目名) -->
        <artifactId>spring-boot-starter-parent</artifactId>
        <!--父项目的版本 -->
        <version>2.3.4.RELEASE</version>
        <!-- 父项目的pom.xml文件的相对路径。默认值是../pom.xml。
		Maven首先在构建当前项目的地方寻找父项目的pom，
		其次在文件系统的这个位置（relativePath位置），然后在本地仓库，最后在远程仓库寻找父项目的pom。 -->
        <relativePath/>
    </parent>


	<!--项目的全球唯一标识符，通常使用全限定的包名区分该项目和其他项目.(公司或组织的域名倒序) -->
	<groupId>com.czm</groupId>
	<!-- 构件的标识符，它和group ID一起唯一标识一个构件。(当前项目的模块名称) -->
    <artifactId>back</artifactId>
	<!--当前模块的版本 -->
    <version>1.0-SNAPSHOT</version>
	<!--项目打包格式：pom、jar、war
	父级项目的packing都为pom，packing默认是jar类型，如果不作配置，maven会将该项目打成jar包。作为父级项目-->
    <packaging>pom</packaging>
    
	<!--统一管理所依赖 jar 包的版本 对同一个框架的一组 jar 包最好使用相同的版本。
	为了方便升级框架，可以将 jar 包的版本信息统一提取出来  -->
	<properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Hoxton.SR8</spring-cloud.version>
        <mysql.version>8.0.12</mysql.version>
        <spring.boot.version>2.3.4.RELEASE</spring.boot.version>
    </properties>
	<!--项目描述 -->
	<description>just a demo</description>
    
  
     <!--jar包依赖坐标 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <!--引用上面统一声明的版本-->
        <version>${spring.boot.version}</version>
        
        <!-- 依赖类型，默认类型是jar。它通常表示依赖的文件的扩展名，但也有例外。一个类型可以被映射成另外一个扩展名或分类器。类型经常和使用的打包方式对应， 尽管这也有例外。一些类型的例子：jar，war，ejb-client和test-jar。 -->
            <type>jar</type>
         <!-- 
			依赖的范围 ：compile、test、provided 
		-->
        <scope>test</scope>
        <!--当计算传递依赖时， 从依赖构件列表里，列出被排除的依赖构件集。
			即告诉maven你只依赖指定的项目，不依赖项目的依赖。此元素主要用于解决版本冲突问题 -->
        <exclusions>
            <exclusion>
                <groupId>org.junit.vintage</groupId>
                <artifactId>junit-vintage-engine</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
   
    <build>
       <resources>  
          <!--描述与项目关联的文件是什么和在哪里-->
           <resource>  
               <!-- 描述了资源的目标路径。该路径相对target/classes目录（例如${project.build.outputDirectory}）。举个例子，如果你想资源在特定的包里(org.apache.maven.messages)，你就必须该元素设置为org/apache/maven /messages。然而，如果你只是想把资源放到源码目录结构里，就不需要该配置。 -->
             <targetPath> </targetPath> 
              <!-- true/false，表示为这个resource，filter是否激活-->
             <filtering>true</filtering>  
               <!--定义resource文件所在的文件夹，默认为${basedir}/src/main/resources-->
            <directory> </directory>  
               <!--  指定哪些文件将被匹配，以*作为通配符-->
            <includes>  
                <include>configuration.xml</include>  
            </includes>  
               <!-- 指定哪些文件将被忽略-->
            <excludes>  
                <exclude>**/*.properties</exclude>  
            </excludes>  
         </resource>  
    </resources>  
        <!-- 定义和resource类似，只不过在test时使用-->
    <testResources>  
        ...  
    </testResources>  
 	<!--指定使用的插件-->
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>
```



## 父项目pom常用坐标

```xml
	<!--引入springboot开始依赖-->
	<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.4.RELEASE</version>
        <relativePath/> 
    </parent>
<!--声明子模块，指定模块工程的相对路径即可 -->
    <modules>
        <module>model1</module>
        <module>model2</module>
        ……
    </modules>
<!--统一声明-->
	<properties>
        <java.version>1.8</java.version>
        <spring.boot.version>2.3.4.RELEASE</spring.boot.version>
        <spring-boot-maven-plugin>2.3.4.RELEASE</spring-boot-maven-plugin>
    </properties>

<!--声明依赖，并不实现引入，因此子项目需要显示的声明需要用的依赖。如果不在子项目中声明依赖，是不会从父项目中继承下来的；只有在子项目中写了该依赖项，并且没有指定具体版本，才会从父项目中继承该项，并且version和scope都读取自父pom;另外如果子项目中指定了版本号，那么会使用子项目中指定的jar版本。
 -->
<dependencyManagement>
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
            <version>${spring.boot.version}</version>
        </dependency>
    
    <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
        <!--注意使用import标签时，不再使用<parent>标签
			表示将父项目的dependencyManagement拿到本POM中，不再继承parent
			type必须是pom-->
            <scope>import</scope>
        </dependency>
</dependencyManagement>
<!--构建项目需要的信息 -->
<build>
    <!--与dependencyManagement类似，只声明-->
    <pluginManagement>
        <!--使用的插件列表 -->
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>${spring-boot-maven-plugin}</version>
            </plugin>
            <!--添加配置跳过测试类构建-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.22.1</version>
                <configuration>
                    <skipTests>true</skipTests>
                </configuration>
            </plugin>
        </plugins>
    </pluginManagement>
</build>

```

