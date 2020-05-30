[TOC]
##前言##
Spring框架作为JavaEE框架领域的一款重要的开源框架，在企业应用开发中有着很重要的作用，同时Spring框架及其子框架很多，所以知识量很广。
Spring Boot：一款Spring框架的子框架，也可以叫微框架，是2014年推出的一款使Spring框架开发变得容易的框架。学过Spring框架的都知识，Spring框架难以避免地需要配置不少XMl，而使用Spring Boot框架的话，就可以使用注解开发，极大地简化基于Spring框架的开发。Spring Boot充分利用了JavaConfig的配置模式以及“约定优于配置”的理念，能够极大的简化基于Spring MVC的Web应用和REST服务开发。
然后本博客介绍基于IDEA编辑器的Spring Boot项目创建和部署。
##Spring Boot项目创建##

 1. 创建Maven项目
![这里写图片描述](http://img.blog.csdn.net/20170424195247952?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDQyNzM5MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
 2. 在pom.xml加入Spring Boot的jar 
如果只是测试一个字符串输出的话，只要加入spring-boot-starter（核心模块）和spring-boot-starter-web（因为这个一个Web项目），可以参考我的配置，这里使用了Spring Boot热部署，需要去github上搜索jar：springloaded-1.2.4.RELEASE.jar，然后下载放在项目的lib文件夹里
```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>demo</artifactId>
  <packaging>war</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>demo Maven Webapp</name>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.1.RELEASE</version>
    <relativePath/>
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <java.version>1.8</java.version>
    <spring-boot-admin.version>1.4.5</spring-boot-admin.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>

    <dependency>
      <groupId>de.codecentric</groupId>
      <artifactId>spring-boot-admin-starter-client</artifactId>
      <version>${spring-boot-admin.version}</version>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
    </dependency>

  </dependencies>
  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin </artifactId>
        <dependencies>
          <!--springloaded hot deploy -->
          <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>springloaded</artifactId>
            <systemPath>${basedir}/src/main/webapp/WEB-INF/lib/springloaded-1.2.5.RELEASE.jar</systemPath>
          </dependency>
        </dependencies>
        <executions>
          <execution>
            <goals>
              <goal>repackage</goal>
            </goals>
            <configuration>
              <classifier>exec</classifier>
            </configuration>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
</project>

```
刷新，下载jar到maven项目里
![这里写图片描述](http://img.blog.csdn.net/20170424200716783?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDQyNzM5MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
 3. 
 编写程序，项目结构如图
 ![这里写图片描述](http://img.blog.csdn.net/20170424195815592?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDQyNzM5MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

写个启动类Application.java:
启动类设置端口为8087，因为默认端口是8080，而有很多应用都是8080端口，避免重复，最好自己改端口
其中@SpringBootApplication申明让spring boot自动给程序进行必要的配置，等价于以默认属性使用
@Configuration，@EnableAutoConfiguration和@ComponentScan
```
package com;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.context.embedded.ConfigurableEmbeddedServletContainer;
import org.springframework.boot.context.embedded.EmbeddedServletContainerCustomizer;
import org.springframework.boot.web.support.SpringBootServletInitializer;
import org.springframework.scheduling.annotation.EnableAsync;

@SpringBootApplication
@EnableAsync
public class Application implements EmbeddedServletContainerCustomizer {

	public static void main(String[] args) {

		SpringApplication.run(Application.class, args);
	}

	@Override
	public void customize(ConfigurableEmbeddedServletContainer configurableEmbeddedServletContainer) {

		configurableEmbeddedServletContainer.setPort(8087);
	}
}

```
写个Controller类：

```
package com.example;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * Created by Administrator on 2017/4/24.
 */
@RestController
@RequestMapping("/")
public class DemoController {
    @RequestMapping("/demo")
    private String demo() {
        return "this is spring boot demo!!!";
    }
}
```
导入不想自己写demo，可以通过http://start.spring.io/ ，在平台自动生成一个demo代码，然后打开项目就好
## Spring Boot部署 ##
 添加个Spring Boot配置服务器
![这里写图片描述](http://img.blog.csdn.net/20170424200305485?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDQyNzM5MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
 
 ![这里写图片描述](http://img.blog.csdn.net/20170424200930926?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDQyNzM5MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
 
 访问：
![这里写图片描述](http://img.blog.csdn.net/20170424201119146?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDQyNzM5MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

附录：
学习教程：http://412887952-qq-com.iteye.com/blog/2291496


 



