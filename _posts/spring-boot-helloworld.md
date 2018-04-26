---
title: spring helloworld
date: 2017-09-09 22:00:58
categories: IT
tags: spring-boot
---
忙了这么久，感觉好久都没有学习一点新技术了，之前看过spring-boot的一些文档，现在打算实践一遍。很强势的，我们从hello world 开始。从不用ide开始。<!-- more -->
- 1、新建一个maven项目。
  可以用以下命令
  ``` bash
  mkdir -p spring-boot-hello/src/main/java
  touch pom.xml
  ```
  其中pom.xml文件中输入以下内容
  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>spring-boot-hello</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <!-- 这个代表着此项目是从  parent中继承来的 -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.0.M3</version>
    </parent>

    <!-- 添加web相关的包支持 -->
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
    </project>
    ```
-  2、然后创建一个名为HelloController.java的文件
    ```bash
     mkdir -p com/hello
     cd com/hello/
     touch HelloController.java
    ```
    其中内容如下:
    ```java
    package com.hello;

    @RestController
    @RequestMapping("/hello")
    public class HelloController{

        @RequestMapping("/say")
        public String sayHello(){
            return "Hello World!";
        }
    }
    ```
-  3、返回到java目录，新建名为Application.java的文件，内容如下:
    ```java
    @EnableAutoConfiguration
    public class Application {
        public static void main(String[] args) throws Exception{
            SpringApplication.run(Application.class, args);
        }
    }
    ```
- 4、利用mvn package 打包，进入到target目录下,生成一个以  spring-boot-hello 开头的jar文件。
- 5、用 java -jar 命令运行这个jar文件。可以在命令行看到spring-boot的启动信息。
- 6、在浏览器中用 localhost:8080/hello/say 访问，就可以看到  Hello World! 了。