
hexo：是一款基于NodeJS的博客框架，具有快速、简洁且高效的优点；使用Markdown解析文章，可以在几秒内快速生成静态文章博文

补充NodeJS：目前速度最快的Javascript引擎，和2014年出现的AngularJS不同，NodeJS是一款基于后端的语言，有数据库操作，快速、高效的优点，比起PHP来说开发效率更高

Markdown：它允许人们“使用易读易写的纯文本格式编写文档，然后转换成有效的XHTML(或者HTML)文档”。


![Hexo图解](http://img.blog.csdn.net/20161215150000069?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDQyNzM5MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

搭建过程：
环境准备：
Git:https://git-scm.com/downloads  (官网)
NodeJS:http://nodejs.cn/  (NodeJS中文网)
MarkDown:http://download.csdn.net/detail/u014427391/9712318

安装过程略过，需要注意的是NodeJS需要设置好环境变量：
点击我的电脑，属性，选择高级系统设置
在Path变量，加上NodeJS的安装路径和Git的安装路径

![这里写图片描述](http://img.blog.csdn.net/20161215152305450?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDQyNzM5MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

window键+R，输入cmd，进入你nodejs的安装路径，输入

```
npm install -g hexo
```
安装hexo环境
![这里写图片描述](http://img.blog.csdn.net/20161215152447510?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDQyNzM5MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


在nodejs文件夹里创建一个文件夹，用于存放
myblog就是你要创建的文件夹名称
```
mkdir myblog
```
然后进入myblog文件夹

```
cd myblog
```

Git初始化
```
git init
```
安装环境配置：

```
hexo init
```

```
npm install
```
然后
生成静态博客，使用hexo g
```
hexo g
```
启动hexo

```
hexo s
```
在本地系统输入http://localhost:4000  ,可以看到你的静态博客已经生成
然后，注册一个github账号，创建一个博客page
https://github.com

注意repository名称要命名为你的github用户名.github.io
选择你的github博客的主题，先发布出去

![这里写图片描述](http://img.blog.csdn.net/20161215153826817?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDQyNzM5MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

然后进入刚才创建的myblog文件夹，找到_config.yml，打开
修改deploy的type为git，repository为你github上创建repository的url，注意加个空格哦
![这里写图片描述](http://img.blog.csdn.net/20161215154008911?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDQyNzM5MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

解决hexo新版本的部署问题
```
npm install hexo-deployer-git -save
```
重新生成blog

```
hexo g
```
部署项目到github

```
hexo  d
```
输入https:你的用户名.github.io，可以看到你的个人博客已经创建好了
![这里写图片描述](http://img.blog.csdn.net/20161215154528287?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDQyNzM5MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
