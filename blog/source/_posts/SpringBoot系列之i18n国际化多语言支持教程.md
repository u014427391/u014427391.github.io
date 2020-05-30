

title: SpringBoot系列之i18n国际化多语言支持教程
date: 2019-10-25 00:00:00
---
<Excerpt in index | 首页摘要> 

本博客介绍一下SpringBoot集成i18n，实现系统语言国际化处理，ok，先创建一个SpringBoot项目，具体的参考我的博客专栏：
<!-- more -->
<The rest of contents | 余下全文>


SpringBoot系列之i18n国际化多语言支持教程
@[toc]
### 1、环境搭建
本博客介绍一下SpringBoot集成i18n，实现系统语言国际化处理，ok，先创建一个SpringBoot项目，具体的参考我的博客专栏：[SpringBoot系列博客专栏链接](https://blog.csdn.net/u014427391/category_9195353.html)

环境准备：
* IntelliJ IDEA
* Maven

项目集成：
* Thymeleaf(模板引擎，也可以选jsp或者freemark)
* SpringBoot2.2.1.RELEASE

### 2、resource bundle资源配置
ok，要实现国际化语言，先要创建resource bundle文件：
在resources文件夹下面创建一个i18n的文件夹，其中：
* messages.properties是默认的配置
* messages_zh_CN.properties是（中文/中国）
* messages_en_US.properties是（英文/美国）
* etc.
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191124174259318.png)
IDEA工具就提供了很简便的自动配置功能，如图，只要点击新增按钮，手动输入，各配置文件都会自动生成属性
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019112417471286.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9zbWlsZW5pY2t5LmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)
messages.properties：

```
messages.loginBtnName=登录~
messages.password=密码~
messages.rememberMe=记住我~
messages.tip=请登录~
messages.username=用户名~
```

messages_zh_CN.properties：

```
messages.loginBtnName=登录
messages.password=密码
messages.rememberMe=记住我
messages.tip=请登录
messages.username=用户名

```

messages_en_US.properties：

```
messages.loginBtnName=login
messages.password=password
messages.rememberMe=Remember me
messages.tip=Please login in
messages.username=userName

```
在项目的application.properties修改默认配置，让SpringBoot的自动配置能读取到resource bundle资源文件

```
## 配置i18n
# 默认是i18n（中文/中国）
spring.mvc.locale=zh_CN
# 配置resource bundle资源文件的前缀名eg:i18n是文件夹名，messages是资源文件名，支持的符号有.号或者/
spring.messages.basename=i18n.messages
# 设置缓存时间，2.2.1是s为单位，之前版本才是毫秒
spring.messages.cache-duration=1
# 设置资源文件编码格式为utf8
spring.messages.encoding=utf-8

```

注意要点：
* spring.messages.basename必须配置，否则SpringBoot的自动配置将失效
MessageSourceAutoConfiguration.ResourceBundleCondition 源码：
```java
protected static class ResourceBundleCondition extends SpringBootCondition {
		//定义一个map缓存池
		private static ConcurrentReferenceHashMap<String, ConditionOutcome> cache = new ConcurrentReferenceHashMap<>();

		@Override
		public ConditionOutcome getMatchOutcome(ConditionContext context, AnnotatedTypeMetadata metadata) {
			String basename = context.getEnvironment().getProperty("spring.messages.basename", "messages");
			ConditionOutcome outcome = cache.get(basename);//缓存拿得到，直接从缓存池读取
			if (outcome == null) {//缓存拿不到，重新读取
				outcome = getMatchOutcomeForBasename(context, basename);
				cache.put(basename, outcome);
			}
			return outcome;
		}

		private ConditionOutcome getMatchOutcomeForBasename(ConditionContext context, String basename) {
			ConditionMessage.Builder message = ConditionMessage.forCondition("ResourceBundle");
			for (String name : StringUtils.commaDelimitedListToStringArray(StringUtils.trimAllWhitespace(basename))) {
				for (Resource resource : getResources(context.getClassLoader(), name)) {
					if (resource.exists()) {
					//匹配resource bundle资源
						return ConditionOutcome.match(message.found("bundle").items(resource));
					}
				}
			}
			return ConditionOutcome.noMatch(message.didNotFind("bundle with basename " + basename).atAll());
		}
		//解析资源文件
		private Resource[] getResources(ClassLoader classLoader, String name) {
			String target = name.replace('.', '/');//spring.messages.basename参数值的点号换成斜杆
			try {
				return new PathMatchingResourcePatternResolver(classLoader)
						.getResources("classpath*:" + target + ".properties");
			}
			catch (Exception ex) {
				return NO_RESOURCES;
			}
		}

	}
```

* cache-duration在2.2.1版本，指定的是s为单位，找到SpringBoot的MessageSourceAutoConfiguration自动配置类
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191124191550125.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9zbWlsZW5pY2t5LmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)


### 3、LocaleResolver类
SpringBoot默认采用AcceptHeaderLocaleResolver类作为默认LocaleResolver，LocaleResolver类的作用就是作为i18n的分析器，获取对应的i18n配置，当然也可以自定义LocaleResolver类

```java

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.lang.Nullable;
import org.springframework.util.StringUtils;
import org.springframework.web.servlet.LocaleResolver;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.Locale;

/**
 * <pre>
 *  自定义LocaleResolver类
 * </pre>
 * @author nicky
 * <pre>
 * 修改记录
 *    修改后版本:     修改人：  修改日期: 2019年11月23日  修改内容:
 * </pre>
 */
public class CustomLocalResolver implements LocaleResolver {

    Logger LOG = LoggerFactory.getLogger(this.getClass());

    @Nullable
    private Locale defaultLocale;

    public void setDefaultLocale(@Nullable Locale defaultLocale) {
        this.defaultLocale = defaultLocale;
    }

    @Nullable
    public Locale getDefaultLocale() {
        return this.defaultLocale;
    }

    @Override
    public Locale resolveLocale(HttpServletRequest request) {
        Locale defaultLocale = this.getDefaultLocale();//获取application.properties默认的配置
        if(defaultLocale != null && request.getHeader("Accept-Language") == null) {
            return defaultLocale;//http请求头没获取到Accept-Language才采用默认配置
        } else {//request.getHeader("Accept-Language")获取得到的情况
            Locale requestLocale = request.getLocale();//获取request.getHeader("Accept-Language")的值
            String localeFlag = request.getParameter("locale");//从URL获取的locale值
            //LOG.info("localeFlag:{}",localeFlag);
            //url链接有传locale参数的情况，eg:zh_CN
            if (!StringUtils.isEmpty(localeFlag)) {
                String[] split = localeFlag.split("_");
                requestLocale = new Locale(split[0], split[1]);
            }
            //没传的情况，默认返回request.getHeader("Accept-Language")的值
            return requestLocale;
        }
    }

    @Override
    public void setLocale(HttpServletRequest request, HttpServletResponse response, Locale locale) {

    }
}
```

### 4、I18n配置类
**I18n还是要继承WebMvcConfigurer，注意，2.2.1版本才是实现接口就可以，之前1.+版本是要实现WebMvcConfigurerAdapter适配器类的**

```java
import com.example.springboot.i18n.component.CustomLocalResolver;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.web.servlet.WebMvcProperties;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.LocaleResolver;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import org.springframework.web.servlet.i18n.LocaleChangeInterceptor;

/**
 * <pre>
 *  I18nConfig配置类
 * </pre>
 * <p>
 * <pre>
 * @author nicky.ma
 * 修改记录
 *    修改后版本:     修改人：  修改日期: 2019/11/24 11:15  修改内容:
 * </pre>
 */
 //Configuration必须加上，不然不能加载到Spring容器
@Configuration
//使WebMvcProperties配置类可用，这个可以不加上，本博客例子才用
@EnableConfigurationProperties({ WebMvcProperties.class})
public class I18nConfig implements WebMvcConfigurer{
	
	//装载WebMvcProperties 属性
    @Autowired
    WebMvcProperties webMvcProperties;
    /**
     * 定义SessionLocaleResolver
     * @Author nicky.ma
     * @Date 2019/11/24 13:52
     * @return org.springframework.web.servlet.LocaleResolver
     */
//    @Bean
//    public LocaleResolver localeResolver() {
//        SessionLocaleResolver sessionLocaleResolver = new SessionLocaleResolver();
//        // set default locale
//        sessionLocaleResolver.setDefaultLocale(Locale.US);
//        return sessionLocaleResolver;
//    }

    /**
     * 定义CookieLocaleResolver
     * @Author nicky.ma
     * @Date 2019/11/24 13:51
     * @return org.springframework.web.servlet.LocaleResolver
     */
//    @Bean
//    public LocaleResolver localeResolver() {
//        CookieLocaleResolver cookieLocaleResolver = new CookieLocaleResolver();
//        cookieLocaleResolver.setCookieName("Language");
//        cookieLocaleResolver.setCookieMaxAge(1000);
//        return cookieLocaleResolver;
//    }

    /**
     * 自定义LocalResolver
     * @Author nicky.ma
     * @Date 2019/11/24 13:45
     * @return org.springframework.web.servlet.LocaleResolver
     */
    @Bean
    public LocaleResolver localeResolver(){
        CustomLocalResolver localResolver = new CustomLocalResolver();
        localResolver.setDefaultLocale(webMvcProperties.getLocale());
        return localResolver;
    }

    /**
     * 定义localeChangeInterceptor
     * @Author nicky.ma
     * @Date 2019/11/24 13:45
     * @return org.springframework.web.servlet.i18n.LocaleChangeInterceptor
     */
    @Bean
    public LocaleChangeInterceptor localeChangeInterceptor(){
        LocaleChangeInterceptor localeChangeInterceptor = new LocaleChangeInterceptor();
        //默认的请求参数为locale，eg: login?locale=zh_CN
        localeChangeInterceptor.setParamName(LocaleChangeInterceptor.DEFAULT_PARAM_NAME);
        return localeChangeInterceptor;
    }

    /**
     * 注册拦截器
     * @Author nicky.ma
     * @Date 2019/11/24 13:47
     * @Param [registry]
     * @return void
     */
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
     registry.addInterceptor(localeChangeInterceptor()).addPathPatterns("/**");
    }
}

```

注意要点：
* 旧版代码可以不加LocaleChangeInterceptor 拦截器，2.2.1版本必须通过拦截器
* 如下代码，bean的方法名必须为localeResolver，否则会报错

```java
@Bean
    public LocaleResolver localeResolver(){
        CustomLocalResolver localResolver = new CustomLocalResolver();
        localResolver.setDefaultLocale(webMvcProperties.getLocale());
        return localResolver;
    }
```
原理：
跟一下源码，点进LocaleChangeInterceptor类
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191124193029452.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9zbWlsZW5pY2t5LmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019112419322959.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9zbWlsZW5pY2t5LmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)
DispatcherServlet是Spring一个很重要的分发器类，在DispatcherServlet的一个init方法里找到这个LocaleResolver的init方法
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191124193508742.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9zbWlsZW5pY2t5LmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)
**这个IOC获取的bean类名固定为localeResolver，写例子的时候，我就因为改了bean类名，导致一直报错，跟了源码才知道Bean类名要固定为localeResolver**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191124193647136.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9zbWlsZW5pY2t5LmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)
抛异常的时候，也是会获取默认的LocaleResolver的

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191124193926753.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9zbWlsZW5pY2t5LmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191124193952223.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9zbWlsZW5pY2t5LmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191124194025637.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9zbWlsZW5pY2t5LmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)
找到资源文件，确认，还是默认为AcceptHeaderLocaleResolver
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191124194052217.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9zbWlsZW5pY2t5LmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)
配置了locale属性的时候，还是选用AcceptHeaderLocaleResolver作为默认的LocaleResolver
```bash
spring.mvc.locale=zh_CN
```
WebMvcAutoConfiguration.localeResolver方法源码，ConditionalOnMissingBean主键的意思是LocaleResolver没有自定义的时候，才作用，ConditionalOnProperty的意思，有配了属性才走这里的逻辑
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191124194217184.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9zbWlsZW5pY2t5LmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)

* 拦截器拦截的请求参数默认为locale，要使用其它参数，必须通过拦截器设置 ,eg：`localeChangeInterceptor.setParamName("lang");`
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191124193121260.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9zbWlsZW5pY2t5LmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)
* LocalResolver种类有：CookieLocaleResolver(Cookie)、SessionLocaleResolver(会话)、FixedLocaleResolver、AcceptHeaderLocaleResolver(默认)、.etc


### 5、Thymeleaf集成
本博客的模板引擎采用Thymeleaf的，所以新增项目时候就要加上maven相关依赖，没有的话，自己加上：

```
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-thymeleaf</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
```

ok，然后去找个bootstrap的登录页面，本博客已尚硅谷老师的例子为例，进行拓展，引入静态资源文件：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191124180539499.png)

Thymeleaf的i18n支持是采用#符号的
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
	<head>
		<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
		<meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
		<meta name="description" content="">
		<meta name="author" content="">
		<title>SpringBoot i18n example</title>
		<!-- Bootstrap core CSS -->
		<link href="asserts/css/bootstrap.min.css" th:href="@{asserts/css/bootstrap.min.css}" rel="stylesheet">
		<!-- Custom styles for this template -->
		<link href="asserts/css/signin.css" th:href="@{asserts/css/signin.css}" rel="stylesheet">
	</head>

	<body class="text-center">
		<form class="form-signin" action="dashboard.html">
			<img class="mb-4" th:src="@{asserts/img/bootstrap-solid.svg}" alt="" width="72" height="72">
			<h1 class="h3 mb-3 font-weight-normal" th:text="#{messages.tip}">Please sign in</h1>
			<label class="sr-only" th:text="#{messages.username}">Username</label>
			<input type="text" class="form-control" th:placeholder="#{messages.username}" required="" autofocus="">
			<label class="sr-only" th:text="#{messages.password} ">Password</label>
			<input type="password" class="form-control" th:placeholder="#{messages.password}" required="">
			<div class="checkbox mb-3">
				<label>
          <input type="checkbox" value="remember-me" > [[#{messages.rememberMe}]]
        </label>
			</div>
			<button class="btn btn-lg btn-primary btn-block" type="submit" th:text="#{messages.loginBtnName}">Sign in</button>
			<p class="mt-5 mb-3 text-muted">© 2019</p>
			<a class="btn btn-sm" th:href="@{/login(locale='zh_CN')} ">中文</a>
			<a class="btn btn-sm" th:href="@{/login(locale='en_US')} ">English</a>
		</form>

	</body>

</html>
```

切换中文网页：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191124190802576.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9zbWlsZW5pY2t5LmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)
切换英文网页：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191124190833284.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9zbWlsZW5pY2t5LmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)

当然不点链接传locale的方式也是可以自动切换的，浏览器设置语言：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191124191042387.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9zbWlsZW5pY2t5LmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)

原理localeResolver类会获取Accept language参数
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191124191200847.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9zbWlsZW5pY2t5LmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)

**附录：**
logging manual：[SpringBoot官方手册](https://docs.spring.io/spring-boot/docs/2.2.1.RELEASE/reference/html/spring-boot-features.html)
example source：[例子代码下载链接](https://github.com/u014427391/springbootexamples)


