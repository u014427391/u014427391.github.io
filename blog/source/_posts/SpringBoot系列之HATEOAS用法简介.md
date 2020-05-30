

title: SpringBoot系列之HATEOAS用法简介
date: 2019-10-25 00:00:00
---
<Excerpt in index | 首页摘要> 

介绍HATEOAS之前先简单介绍一下REST，REST 是 Representational state transfer 的缩写，翻译过来的意思是表达性状态转换。REST是一种架构的风格

<!-- more -->
<The rest of contents | 余下全文>

## REST风格简介

介绍HATEOAS之前先简单介绍一下REST，REST 是 Representational state transfer 的缩写，翻译过来的意思是表达性状态转换。REST是一种架构的风格

## Richardson Maturity Model
 Richardson 提出了REST一种 成熟度模型，我们称之为Richardson Maturity Model，这种模式将REST按照成熟度划分为4个等级
 * Level0：使用HTTP作为WEB服务的传输方式，以REST样式公开SOAP Web服务
 * Level1：使用适当的URI（使用名词）公开资源，这种方式提出了资源的概念
 * Level2：资源使用正确的URI + HTTP方法，比如更新用户就用put方式，查询用get方式
 * Level3：使用HATEOAS(作为应用程序状态引擎的超媒体)，在资源的表达中包含了链接信息，客户端可以在链接信息中发现可以执行的操作
 * 
## HATEOAS是什么？
> HATEOAS代表“超媒体是应用程序状态的引擎”

从前言我们已经可以清楚知道，使用HATEOAS约束是REST风格中成熟度最高的，也是官方推荐的一种方式，没使用HATEOAS的项目，服务端和客户端是耦合的，客户端只能通过相关文档来知道服务端做了什么修改，使用HATEOAS约束的REST服务，服务端修改接口信息后，客户端可以通过服务器提供的资源的表达来智能地发现可以执行的操作，客户端不需要做啥修改，因为资源信息是会动态改变的

在Spring的官网，已经有提供这个项目的相关文档，链接：https://spring.io/projects/spring-hateoas


## SpringBoot HATEOAS
SpringBoot中也有集成HATEOAS，本博客介绍一下如何使用

**工具准备：**
* JDK8.0
* Maven 3.0+构建工具
* Eclipse或者IntelliJ IDEA
* git&gitlab

**Maven相关配置**
在pom.xml加上hateoas配置
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-hateoas</artifactId>
</dependency>

```

因为是要写个web简单curd例子，其它需要的也加上

```xml
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-hateoas</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>druid</artifactId>
			<version>1.0.25</version>
		</dependency>
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.40</version>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
			<exclusions>
				<exclusion>
					<groupId>org.junit.vintage</groupId>
					<artifactId>junit-vintage-engine</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		
```

**实体类实现ResourceSupport**

Model类实现hateoas提供的ResourceSuppor

```java
import com.fasterxml.jackson.annotation.JsonCreator;
import com.fasterxml.jackson.annotation.JsonProperty;
import org.springframework.hateoas.ResourceSupport;

import javax.persistence.*;
import java.io.Serializable;
@Entity
@Table(name="sys_user")
public class SysUserInfo extends ResourceSupport implements Serializable{

    @Id
    @GeneratedValue
    private Long userId;
    @Column(unique=true,length=20,nullable=false)
    private String username;
    @Column(length=2,nullable=true)
    private String sex;
    @Column(length=10,nullable=true)
    private String password;

    public SysUserInfo(){

    }

    @JsonCreator
    public SysUserInfo(@JsonProperty("userId")Long userId,@JsonProperty("username")String username,
                       @JsonProperty("sex")String sex,@JsonProperty("password")String password){
        this.userId = userId;
        this.username = username;
        this.sex = sex;
        this.password = password;
    }
}
....
```

**接口调用，基于HATEOAS模式**
```java
@GetMapping("/findBySysUserId/{userId}")
    public SysUserInfo findBySysUserId(@PathVariable("userId") long userId) {
        if (LOG.isInfoEnabled()) {
            LOG.info("请求参数userId : {}" , userId);
        }
        Optional<SysUserInfo> sysUserInfo = Optional.ofNullable(sysUserRepository.findByUserId(userId));
        if (!sysUserInfo.isPresent()) {
            throw new NotFoundException("查询不到用户信息! userId:"+userId);
        }
        //Resource<SysUserInfo> resource = new Resource<SysUserInfo>(sysUserInfo.get());
        ControllerLinkBuilder linkBuilder = linkTo(methodOn(this.getClass()).findBySysUserId(userId));
        sysUserInfo.get().add(linkBuilder.withRel("findBySysUserId"));
        return sysUserInfo.get();
    }
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191026175757535.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9zbWlsZW5pY2t5LmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)

实例代码：[github链接下载](https://github.com/u014427391/springbootexamples)
