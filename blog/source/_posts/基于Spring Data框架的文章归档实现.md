title: 基于Spring Data框架的文章归档实现
date: 2017-12-03 00:00:00
---
<Excerpt in index | 首页摘要> 
最近在写自己的个人博客系统，框架采用SpringMVC、Spring4.0、Spring Data/JPA组合，本博客就文档归档功能在Spring Data JPA框架下是如何实现的进行记录。
现在可以star(收藏)，watch(关注)，但是还在开发中，所以还是还在先别下载
项目github：https://github.com/u014427391/myblog
<!-- more -->
<The rest of contents | 余下全文>

### 目录

[TOC]

###前言
最近在写自己的个人博客系统，框架采用SpringMVC、Spring4.0、Spring Data/JPA组合，本博客就文档归档功能在Spring Data JPA框架下是如何实现的进行记录。
现在可以star(收藏)，watch(关注)，但是还在开发中，所以还是还在先别下载
项目github：https://github.com/u014427391/myblog
###文章信息设计
数据暂时这样设计，仅供学习参考，对于文章评论回复，栏目之间的关联还没设计，不过本博客的目的是记录文档归档功能的实现，这个并不会影响
![这里写图片描述](http://img.blog.csdn.net/20170403114527194?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDQyNzM5MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

VO类：全部采用注解，注意因为我数据库表名为article，所以不需要写@Table注解，表名为其它的话，就需要自己添加@Table注解了

```
package net.myblog.entity;

import java.util.Date;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.Temporal;
import javax.persistence.TemporalType;

/**
 * 博客系统文章信息的实体类
 * @author Nicky
 */
@Entity
public class Article {
	
	/** 文章Id，自增**/
	private int articleId;
	
	/** 文章名称**/
	private String articleName;
	
	/** 文章发布时间**/
	private Date articleTime;
	
	/** 图片路径，测试**/
	private String imgPath;
	
	/** 文章内容**/
	private String articleContent;
	
	/** 查看人数**/
	private int articleClick;
	
	/** 是否博主推荐。0为否；1为是**/
	private int articleSupport;
	
	/** 是否置顶。0为；1为是**/
	private int articleUp;
	
	/** 文章类别。0为私有，1为公开，2为仅好友查看**/
	private int articleType;
	
	@GeneratedValue
	@Id
	public int getArticleId() {
		return articleId;
	}

	public void setArticleId(int articleId) {
		this.articleId = articleId;
	}

	@Column(length=100, nullable=false)
	public String getArticleName() {
		return articleName;
	}

	public void setArticleName(String articleName) {
		this.articleName = articleName;
	}

	@Temporal(TemporalType.DATE)
	@Column(nullable=false, updatable=false)
	public Date getArticleTime() {
		return articleTime;
	}

	public void setArticleTime(Date articleTime) {
		this.articleTime = articleTime;
	}

	@Column(length=100)
	public String getImgPath() {
		return imgPath;
	}

	public void setImgPath(String imgPath) {
		this.imgPath = imgPath;
	}

	@Column(nullable=false)
	public String getArticleContent() {
		return articleContent;
	}

	public void setArticleContent(String articleContent) {
		this.articleContent = articleContent;
	}

	public int getArticleClick() {
		return articleClick;
	}

	public void setArticleClick(int articleClick) {
		this.articleClick = articleClick;
	}

	public int getArticleSupport() {
		return articleSupport;
	}

	public void setArticleSupport(int articleSupport) {
		this.articleSupport = articleSupport;
	}

	public int getArticleUp() {
		return articleUp;
	}

	public void setArticleUp(int articleUp) {
		this.articleUp = articleUp;
	}

	@Column(nullable=false)
	public int getArticleType() {
		return articleType;
	}

	public void setArticleType(int articleType) {
		this.articleType = articleType;
	}
}

```

###代码实现步骤

文章表里有很多数据，要按照年月获取文章进行归档的话，我们可以使用如下SQL对数据进行分组

```
SELECT YEAR(articleTime) AS 'year',MONTH(articleTime) AS 'month',COUNT(*) AS 'count' FROM article 
	GROUP BY YEAR(articleTime) DESC,MONTH(articleTime);
```

然后编写数据库层的Repository类，类实现Spring Data JPA提供的接口

```
package net.myblog.repository;

import java.util.Date;
import java.util.List;

import net.myblog.entity.Article;

import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.PagingAndSortingRepository;
import org.springframework.data.repository.query.Param;

public interface ArticleRepository extends PagingAndSortingRepository<Article,Integer>{
	/**
	 * 文章归档信息获取
	 * @return
	 */
	@Query(value="select year(a.articleTime) as year,month(a.articleTime) as month,"
			+ "count(a) as count from Article a group by year(a.articleTime),month(a.articleTime)",
			countQuery="select count(1) from (select count(1) from Article a group by year(a.articleTime),month(a.articleTime))")
	public List<Object[]> findArticleGroupByTime();
	
}
```
然后在Service类，用@Autowired注解调用

```
package net.myblog.service;

import java.util.Date;
import java.util.List;

import net.myblog.entity.Article;
import net.myblog.repository.ArticleRepository;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Sort.Direction;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class ArticleService {
	
	@Autowired ArticleRepository articleRepository;
	/**
	 * 文章归档信息获取
	 * @return
	 */
	@Transactional
	public List<Object[]> findArticleGroupByTime(){
		return articleRepository.findArticleGroupByTime();
	}
	
}

```
然后在Controller里调用，用的是SpringMVC框架

```
package net.myblog.web.controller;

import java.util.ArrayList;
import java.util.List;
import java.util.Vector;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import net.myblog.service.ArticleService;
import net.sf.json.JSONArray;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Sort.Direction;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

@Controller
public class BlogIndexController extends BaseController{
	
	@Autowired
	ArticleService articleService;
	
	/**
	 * 访问博客主页
	 * @return
	 */
	@RequestMapping(value="/toblog",produces="text/html;charset=UTF-8")
	public ModelAndView toBlog(HttpServletRequest request, HttpServletResponse response, Model model)throws ClassNotFoundException{
		
		//获取归档文章信息
		List<Object[]> archiveArticles = articleService.findArticleGroupByTime();
		
		model.addAttribute("archiveArticles", archiveArticles);
		mv.setViewName("myblog/frame/index");
		return mv;
	}
	
}

```
在JSP页面调用显示：

```
<h2><p>文章归档</p></h2>
<ul class="news">
  <c:choose>
	<c:when test="${not empty archiveArticles }">
		<c:forEach items="${archiveArticles }" var="ac">
		<li><a href="getArchiveArticles.do?yearmonth=${ac[0]}-${ac[1]}">
		${ac[0]}年${ac[1]}月(${ac[2] })</a></li>
		</c:forEach>
	</c:when>
	<c:otherwise>
		<li><a href="#">没有相关数据</a></li>
	</c:otherwise>  
  </c:choose>
</ul>
```
效果如图所示：
![这里写图片描述](http://img.blog.csdn.net/20170403120245873?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDQyNzM5MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

###文档归档信息查询
然后介绍点击文档归档信息后，获取文章信息的实现，其实也就是按年月查询文档信息

在Repository类里添加方法：

```
/**
	 * 按月份获取文章信息
	 * @param month
	 * 			月份数
	 * @return
	 */
	@Query("from Article a where date_format(a.articleTime,'%Y%m')=date_format((:yearmonth),'%Y%m') "
			+ "order by articleTime desc")
	public List<Article> findArticleByMonth(@Param("yearmonth")Date yearmonth);

```

Service类里调用：

```
	/**
	 * 按月份获取文章信息
	 * @param month
	 * @return
	 */
	@Transactional
	public List<Article> findArticleByMonth(Date month){
		return articleRepository.findArticleByMonth(month);
	}
```

在JSP页面写入，getArchiveArticles.do就是要访问的url，传入yearmonth参数就可以
```
<h2><p>文章归档</p></h2>
<ul class="news">
  <c:choose>
	<c:when test="${not empty archiveArticles }">
		<c:forEach items="${archiveArticles }" var="ac">
		<li><a href="getArchiveArticles.do?yearmonth=${ac[0]}-${ac[1]}">
		${ac[0]}年${ac[1]}月(${ac[2] })</a></li>
		</c:forEach>
	</c:when>
	<c:otherwise>
		<li><a href="#">没有相关数据</a></li>
	</c:otherwise>  
  </c:choose>
</ul>
```
在Controller类里进行处理：

```
@RequestMapping("/getArchiveArticles")
	public ModelAndView getArticleByMonth(HttpServletRequest request){
		String yearMonthString = request.getParameter("yearmonth");
		System.out.println("month:"+yearMonthString);
		ModelAndView mv = this.getModelAndView();
		
		Date yearmonth = DateUtils.parse("yyyy-MM", yearMonthString);
		
		List<Article> articles = articleService.findArticleByMonth(yearmonth);

		mv.addObject("articles", articles);
		mv.setViewName("myblog/article/archive_articles");
		return mv;
	}
```
![这里写图片描述](http://img.blog.csdn.net/20170403121235134?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDQyNzM5MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

###附录(工具类、公共类代码)
DateUtils.java、BaseController.java类
DateUtil.java

```
package net.myblog.utils;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Locale;

import net.myblog.core.Constants;

public class DateUtils {
	
	public static String formatDate(Date date) throws ParseException{
		SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		return dateFormat.format(date);
	}
	
	 /**
	   * 解析日期，注:此处为严格模式解析，即20151809这样的日期会解析错误
	   * 
	   * @param pattern
	   * @param date
	   * @return
	   */
	  public static Date parse(String pattern, String date){
	    return parse(pattern, date, Constants.LOCALE_CHINA);
	  }

	  /**
	   * 解析日期，注:此处为严格模式解析，即20151809这样的日期会解析错误
	   * 
	   * @param pattern
	   * @param date
	   * @param locale
	   * @return
	   */
	  public static Date parse(String pattern, String date, Locale locale){
	    SimpleDateFormat format = new SimpleDateFormat(pattern, locale);
	    format.setLenient(false);
	    Date result = null;
	    try{
	      result = format.parse(date);
	    }catch(Exception e){
	      e.printStackTrace();
	    }

	    return result;
	  }

}

```
BaseController.java:

```
package net.myblog.web.controller;

import javax.servlet.http.HttpServletRequest;

import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;
import org.springframework.web.servlet.ModelAndView;

public class BaseController {
	
	/**
	 * 得到request对象
	 */
	public HttpServletRequest getRequest() {
		HttpServletRequest request = ((ServletRequestAttributes)RequestContextHolder.getRequestAttributes()).getRequest();
		
		return request;
	}
	
	/**
	 * 得到ModelAndView
	 */
	public ModelAndView getModelAndView(){
		return new ModelAndView();
	}


}

```