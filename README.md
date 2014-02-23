# Symfony2简介 #

![Symfony](imgs/logo_symfony.png)

网上有很多关于Symfony的介绍，如果你还不知道Symfony是什么，那么《[Symfony2是什么](http://www.cnblogs.com/Seekr/archive/2012/06/15/2550894.html)》(原文：《[What is Symfony2?](http://fabien.potencier.org/article/49/what-is-symfony2)》)将帮助你简单了解Symfony。
下面是常用的Symfony资源：

*  [Symfony官网](http://symfony.com/)
*  [Symfony中文](http://symfony.cn/docs/index.html)
*  [A Symfony2 Tutorial](http://twpug.net/docs/symblog/)
*  [Symfony2 Jobeet](http://www.intelligentbee.com/blog/2013/08/07/symfony2-jobeet-day-1-starting-up-the-project/)

# 关于这一系列的教程 #

基于Symfony2.3，开发一个在线招聘平台（job board），完成后的作品展示[Jobeet](http://www.jobeet.org/en/)。

原文地址：<http://www.intelligentbee.com/blog/2013/08/07/symfony2-jobeet-day-1-starting-up-the-project/>

Symfony官网的Jobeet教程：<http://symfony.com/legacy/doc/jobeet?orm=Doctrine>（**但这个教程基于1.x**）

# 为什么我要翻译这个教程 #

我学习PHP也是有段时间了，接触过的PHP开发框架也就少数几个,有ThinkPHP、CakePHP等，而且它们大都上手简单，开发效率也挺高的。早之前就听过了Symfony2，但是一直没想去了解它。有一天心血来潮，就Google了一些资源，于是就发现了这一些列不错的教程。教程中的内容和《Ruby on Rails Tutorial》差不多，都是以一个真是的案例来给大家讲解框架的使用，其中有教你怎样使用Symfony2进行开发，怎么样进行测试，怎么编写命令行任务等等。又鉴于国内关于Symfony2的资料稀缺，我认为这份资料可以说是很珍贵的。所以，我尝试着翻译（以前没做过翻译）。我翻译这个教程目的是想让更多国内的小伙伴能见识一下Symfony2的强大，PHP的强大，希望大家能对Symfony感兴趣。

# 从何入手 #

学习一个大而全的框架无疑会增加我们的学习成本，而且相对于其它的PHP开发框架，Symfony是复杂了。如何选择一个PHP框架来进行开发自己的应用呢？首先我们当然要对备选的框架的功能和特性有点了解吧。如何才能够快速地了解一个框架在开发中所体现出来的特性呢？那么当然是通过一个实例啦。用实例和最佳实践来展示框架或者语言的特性，那么这一些列Tutorial是你不错的选择。

> 如果你只是想快速了解怎么使用Symfony和开发网站的流程，你可以不必一行行地把代码起敲进去，我鼓励你使用粘贴复制代码的方式来学习。

# 补充 #

这一些列教程我并非是完完整整，一字不漏地翻译原文，因为有些词汇翻译成中文就比较难理解和拗口了，而且其中有些类名，变量名或者是数据模型的名字我都没有对其翻译（当你见到user这个单词时你会不会马上联想到他/她应该有username和password呢），我希望能为小伙伴们翻译出简单易懂的文字。

因为这是我第一次翻译英文教程，所以如果有什么地方翻译地不好或者是曲解了原意，还请大家帮忙指出，我会马上对其进行修正，谢谢。

# 目录 #

* [第一天：开始你的Jobeet项目](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-01/chapter-01.md)
    * [什么是Jobeet？](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-01/chapter-01.md#%E4%BB%80%E4%B9%88%E6%98%AFjobeet)
    * [搭建开发环境](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-01/chapter-01.md#%E6%90%AD%E5%BB%BA%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83)
    * [下载和安装Symfony2.3.2](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-01/chapter-01.md#%E4%B8%8B%E8%BD%BD%E5%92%8C%E5%AE%89%E8%A3%85symfony232)
    * [更新Vendors](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-01/chapter-01.md#%E6%9B%B4%E6%96%B0vendors)
    * [网站服务器配置](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-01/chapter-01.md#%E7%BD%91%E7%AB%99%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%85%8D%E7%BD%AE)
    * [Symfony2控制台](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-01/chapter-01.md#symfony2%E6%8E%A7%E5%88%B6%E5%8F%B0)
    * [创建应用程序的bundle](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-01/chapter-01.md#%E5%88%9B%E5%BB%BA%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F%E7%9A%84bundle)
    * [删除AcmeDemoBundle](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-01/chapter-01.md#%E5%88%A0%E9%99%A4acmedemobundle)
    * [环境](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-01/chapter-01.md#%E7%8E%AF%E5%A2%83)
* [第二天：Jobeet是什么](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-02/chapter-02.md)
    * [用户 Stories](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-02/chapter-02.md#%E7%94%A8%E6%88%B7-stories)
* [第三天：数据模型](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-03/chapter-03.md)
    * [关系模型](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-03/chapter-03.md#%E5%85%B3%E7%B3%BB%E6%A8%A1%E5%9E%8B)
    * [数据库](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-03/chapter-03.md#%E6%95%B0%E6%8D%AE%E5%BA%93)
    * [模式](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-03/chapter-03.md#%E6%A8%A1%E5%BC%8F)
    * [ORM](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-03/chapter-03.md#orm)
    * [查看浏览器](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-03/chapter-03.md#%E6%9F%A5%E7%9C%8B%E6%B5%8F%E8%A7%88%E5%99%A8)
* [第四天：控制器和视图](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-04/chapter-04.md)
    * [MVC架构](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-04/chapter-04.md#mvc%E6%9E%B6%E6%9E%84)
    * [布局](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-04/chapter-04.md#%E5%B8%83%E5%B1%80)
    * [Twig区块（Blocks）](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-04/chapter-04.md#twig%E5%8C%BA%E5%9D%97blocks)
    * [Stylesheets，Javascripts和图片](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-04/chapter-04.md#stylesheetsjavascripts%E5%92%8C%E5%9B%BE%E7%89%87)
    * [Job首页Action](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-04/chapter-04.md#job%E9%A6%96%E9%A1%B5action)
    * [Job首页模板](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-04/chapter-04.md#job%E9%A6%96%E9%A1%B5%E6%A8%A1%E6%9D%BF)
    * [Job页面模板](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-04/chapter-04.md#job%E9%A1%B5%E9%9D%A2%E6%A8%A1%E6%9D%BF)
    * [Job页面的Action](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-04/chapter-04.md#job%E9%A1%B5%E9%9D%A2%E7%9A%84action)
* [第五天：路由](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-05/chapter-05.md#urls)
    * [URLs](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-05/chapter-05.md)
    * [路由配置](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-05/chapter-05.md#%E8%B7%AF%E7%94%B1%E9%85%8D%E7%BD%AE)
    * [在开发（development）环境下的路由](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-05/chapter-05.md#%E5%9C%A8%E5%BC%80%E5%8F%91development%E7%8E%AF%E5%A2%83%E4%B8%8B%E7%9A%84%E8%B7%AF%E7%94%B1)
    * [自定义路由](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-05/chapter-05.md#%E8%87%AA%E5%AE%9A%E4%B9%89%E8%B7%AF%E7%94%B1)
    * [路由Requirements](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-05/chapter-05.md#%E8%B7%AF%E7%94%B1requirements)
    * [调试路由](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-05/chapter-05.md#%E8%B0%83%E8%AF%95%E8%B7%AF%E7%94%B1)
    * [最后的感言](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-05/chapter-05.md#%E6%9C%80%E5%90%8E%E7%9A%84%E6%84%9F%E8%A8%80)
* [第六天：更多的数据模型](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-06/chapter-06.md)
    * [Doctrine查询对象](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-06/chapter-06.md#doctrine%E6%9F%A5%E8%AF%A2%E5%AF%B9%E8%B1%A1)
    * [调试Doctrine生成的SQL](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-06/chapter-06.md#%E8%B0%83%E8%AF%95doctrine%E7%94%9F%E6%88%90%E7%9A%84sql)
    * [序列化对象](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-06/chapter-06.md#%E5%BA%8F%E5%88%97%E5%8C%96%E5%AF%B9%E8%B1%A1)
    * [加入更多的Fixtures](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-06/chapter-06.md#%E5%8A%A0%E5%85%A5%E6%9B%B4%E5%A4%9A%E7%9A%84fixtures)
    * [重构代码](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-06/chapter-06.md#%E9%87%8D%E6%9E%84%E4%BB%A3%E7%A0%81)
    * [首页中的Categories](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-06/chapter-06.md#%E9%A6%96%E9%A1%B5%E4%B8%AD%E7%9A%84categories)
    * [限制结果行数](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-06/chapter-06.md#%E9%99%90%E5%88%B6%E7%BB%93%E6%9E%9C%E8%A1%8C%E6%95%B0)
    * [自定义配置](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-06/chapter-06.md#%E8%87%AA%E5%AE%9A%E4%B9%89%E9%85%8D%E7%BD%AE)
    * [动态Fixtures](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-06/chapter-06.md#%E5%8A%A8%E6%80%81fixtures)
    * [过期的Job页面](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-06/chapter-06.md#%E8%BF%87%E6%9C%9F%E7%9A%84job%E9%A1%B5%E9%9D%A2)
* [第七天：实现分类页面](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-07/chapter-07.md)
    * [配置Category路由](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-07/chapter-07.md#%E9%85%8D%E7%BD%AEcategory%E8%B7%AF%E7%94%B1)
    * [添加链接到Category页面的URL](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-07/chapter-07.md#%E6%B7%BB%E5%8A%A0%E9%93%BE%E6%8E%A5%E5%88%B0category%E9%A1%B5%E9%9D%A2%E7%9A%84url)
    * [创建CategoryController控制器](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-07/chapter-07.md#%E5%88%9B%E5%BB%BAcategorycontroller%E6%8E%A7%E5%88%B6%E5%99%A8)
    * [更新数据库](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-07/chapter-07.md#%E6%9B%B4%E6%96%B0%E6%95%B0%E6%8D%AE%E5%BA%93)
    * [Category页面](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-07/chapter-07.md#category%E9%A1%B5%E9%9D%A2)
    * [包含一个Twig模板](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-07/chapter-07.md#%E5%8C%85%E5%90%AB%E4%B8%80%E4%B8%AAtwig%E6%A8%A1%E6%9D%BF)
    * [分页列表](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-07/chapter-07.md#%E5%88%86%E9%A1%B5%E5%88%97%E8%A1%A8)
* [第八天：单元测试](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-08/chapter-08.md)
    * [Symfony中的测试](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-08/chapter-08.md#symfony%E4%B8%AD%E7%9A%84%E6%B5%8B%E8%AF%95)
    * [为新功能添加测试](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-08/chapter-08.md#%E4%B8%BA%E6%96%B0%E5%8A%9F%E8%83%BD%E6%B7%BB%E5%8A%A0%E6%B5%8B%E8%AF%95)
    * [为Bug添加测试](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-08/chapter-08.md#%E4%B8%BAbug%E6%B7%BB%E5%8A%A0%E6%B5%8B%E8%AF%95)
    * [更好的slugfiy](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-08/chapter-08.md#%E6%9B%B4%E5%A5%BD%E7%9A%84slugfiy%E6%96%B9%E6%B3%95)
    * [代码覆盖率](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-08/chapter-08.md#%E4%BB%A3%E7%A0%81%E8%A6%86%E7%9B%96%E7%8E%87)
    * [Doctrine单元测试](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-08/chapter-08.md#doctrine%E5%8D%95%E5%85%83%E6%B5%8B%E8%AF%95)
    * [测试Job实体](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-08/chapter-08.md#%E6%B5%8B%E8%AF%95job%E5%AE%9E%E4%BD%93)
    * [测试Repository类](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-08/chapter-08.md#%E6%B5%8B%E8%AF%95repository%E7%B1%BB)
* [第九天：功能测试](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-09/chapter-09.md)
    * [第一个功能测试](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-09/chapter-09.md#%E7%AC%AC%E4%B8%80%E4%B8%AA%E5%8A%9F%E8%83%BD%E6%B5%8B%E8%AF%95)
    * [运行功能测试](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-09/chapter-09.md#%E8%BF%90%E8%A1%8C%E5%8A%9F%E8%83%BD%E6%B5%8B%E8%AF%95)
    * [编写功能测试](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-09/chapter-09.md#%E7%BC%96%E5%86%99%E5%8A%9F%E8%83%BD%E6%B5%8B%E8%AF%95)
* [第十天：表单](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-10/chapter-10.md)
    * [自定义Job表单](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-10/chapter-10.md#%E8%87%AA%E5%AE%9A%E4%B9%89job%E8%A1%A8%E5%8D%95)
    * [Symfony2处理上传文件](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-10/chapter-10.md#symfony2%E5%A4%84%E7%90%86%E4%B8%8A%E4%BC%A0%E6%96%87%E4%BB%B6)
    * [表单模板](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-10/chapter-10.md#%E8%A1%A8%E5%8D%95%E6%A8%A1%E6%9D%BF)
    * [表单行为（Action）](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-10/chapter-10.md#%E8%A1%A8%E5%8D%95%E8%A1%8C%E4%B8%BAaction)
    * [使用Token来保护表单](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-10/chapter-10.md#%E4%BD%BF%E7%94%A8token%E6%9D%A5%E4%BF%9D%E6%8A%A4%E8%A1%A8%E5%8D%95)
    * [预览页面](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-10/chapter-10.md#%E9%A2%84%E8%A7%88%E9%A1%B5%E9%9D%A2)
    * [Job激活和发布](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-10/chapter-10.md#job%E6%BF%80%E6%B4%BB%E5%92%8C%E5%8F%91%E5%B8%83)                    
* [第十一天：对你的表单进行测试](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-11/chapter-11.md)
    * [提交表单](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-11/chapter-11.md#%E6%8F%90%E4%BA%A4%E8%A1%A8%E5%8D%95)
    * [测试表单](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-11/chapter-11.md#%E6%B5%8B%E8%AF%95%E8%A1%A8%E5%8D%95)
    * [测试数据库记录](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-11/chapter-11.md#%E6%B5%8B%E8%AF%95%E6%95%B0%E6%8D%AE%E5%BA%93%E8%AE%B0%E5%BD%95)
    * [测试错误的表单](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-11/chapter-11.md#%E6%B5%8B%E8%AF%95%E9%94%99%E8%AF%AF%E7%9A%84%E8%A1%A8%E5%8D%95)
    * [用测试作保障](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-11/chapter-11.md#%E7%94%A8%E6%B5%8B%E8%AF%95%E4%BD%9C%E4%BF%9D%E9%9A%9C)
    * [“穿越未来”的测试](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-11/chapter-11.md#%E7%A9%BF%E8%B6%8A%E6%9C%AA%E6%9D%A5%E7%9A%84%E6%B5%8B%E8%AF%95)
    * [维护任务](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-11/chapter-11.md#%E7%BB%B4%E6%8A%A4%E4%BB%BB%E5%8A%A1)
* [第十二天：后台管理工具包—Sonata Admin](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-12/chapter-12.md)
    * [安装Sonata Admin Bundle](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-12/chapter-12.md#%E5%AE%89%E8%A3%85sonata-admin-bundle)
    * [CRUD控制器](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-12/chapter-12.md#crud%E6%8E%A7%E5%88%B6%E5%99%A8)
    * [创建Admin类](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-12/chapter-12.md#%E5%88%9B%E5%BB%BAadmin%E7%B1%BB)
    * [配置Admin类](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-12/chapter-12.md#%E9%85%8D%E7%BD%AEadmin%E7%B1%BB)
    * [批量操作（Batch Actions）](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-12/chapter-12.md#%E6%89%B9%E9%87%8F%E6%93%8D%E4%BD%9Cbatch-actions)
* [第十三天：安全性](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-13/chapter-13.md)
    * [实现程序的安全性](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-13/chapter-13.md#%E5%AE%9E%E7%8E%B0%E7%A8%8B%E5%BA%8F%E7%9A%84%E5%AE%89%E5%85%A8%E6%80%A7)
    * [User Providers](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-13/chapter-13.md#user-providers)
    * [登出（Logout）](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-13/chapter-13.md#%E7%99%BB%E5%87%BAlogout)
    * [用户Session](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-13/chapter-13.md#%E7%94%A8%E6%88%B7session)
    * [flash信息](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-13/chapter-13.md#flash%E4%BF%A1%E6%81%AF)   
* [第十四天：订阅](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-14/chapter-14.md)
    * [模板格式](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-14/chapter-14.md#%E6%A8%A1%E6%9D%BF%E6%A0%BC%E5%BC%8F)
    * [订阅](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-14/chapter-14.md#%E8%AE%A2%E9%98%85)
* [第十五天：Web Services](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-15/chapter-15.md)
    * [Affiliates](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-15/chapter-15.md#affiliates)
    * [The Job Web Service](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-15/chapter-15.md#the-job-web-service)
    * [测试Web Service](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-15/chapter-15.md#%E6%B5%8B%E8%AF%95web-service)
    * [Affiliate申请表单](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-15/chapter-15.md#affiliate%E7%94%B3%E8%AF%B7%E8%A1%A8%E5%8D%95)
    * [Affiliate后台管理](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-15/chapter-15.md#affiliate%E5%90%8E%E5%8F%B0%E7%AE%A1%E7%90%86)
* [第十六天：邮件](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-16/chapter-16.md)
    * [Swift Mailer](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-16/chapter-16.md#swift-mailer)
    * [测试](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-16/chapter-16.md#%E6%B5%8B%E8%AF%95)
* [第十七天：搜索](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-17/chapter-17.md)
    * [Zend Lucene](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-17/chapter-17.md#zend-lucene)
    * [安装和配置Zend Framework](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-17/chapter-17.md#%E5%AE%89%E8%A3%85%E5%92%8C%E9%85%8D%E7%BD%AEzend-framework)
    * [索引](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-17/chapter-17.md#%E7%B4%A2%E5%BC%95)
    * [搜索](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-17/chapter-17.md#%E6%90%9C%E7%B4%A2)
    * [单元测试](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-17/chapter-17.md#%E5%8D%95%E5%85%83%E6%B5%8B%E8%AF%95)
    * [任务](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-17/chapter-17.md#%E4%BB%BB%E5%8A%A1)
* [第十八天：Ajax](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-18/chapter-18.md)
    * [安装jQuery](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-18/chapter-18.md#%E5%AE%89%E8%A3%85jquery)
    * [包含jQuery](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-18/chapter-18.md#%E5%8C%85%E5%90%ABjquery)
    * [添加行为](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-18/chapter-18.md#%E6%B7%BB%E5%8A%A0%E8%A1%8C%E4%B8%BA)
    * [用户反馈](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-18/chapter-18.md#%E7%94%A8%E6%88%B7%E5%8F%8D%E9%A6%88)
    * [AJAX和Action](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-18/chapter-18.md#ajax%E5%92%8Caction)
    * [测试AJAX](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-18/chapter-18.md#%E6%B5%8B%E8%AF%95ajax)
* [第十九天：国际化和本地化](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-19/chapter-19.md)
    * [用户](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-19/chapter-19.md#%E7%94%A8%E6%88%B7)
    * [Culture in the URL](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-19/chapter-19.md#culture-in-the-url)
    * [Culture测试](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-19/chapter-19.md#culture%E6%B5%8B%E8%AF%95)
    * [语言切换](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-19/chapter-19.md#%E8%AF%AD%E8%A8%80%E5%88%87%E6%8D%A2)
    * [模板](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-19/chapter-19.md#%E6%A8%A1%E6%9D%BF)

# 感谢 #

感谢[IntelligentBee](http://www.intelligentbee.com/)提供这么好的文章。

# 许可证 #

这些文章基于Attribution-NonCommercial 3.0 Unported license发布。**您不需要为本教程付钱。**

您有权复制、分发、修改或展示本教程的内容，但请您指定声明文章的出处（<http://www.intelligentbee.com/>），也请勿将其用于任何商业用途。

您可以在以下链接查看该许可证的全文：

![](imgs/license.png)

<http://creativecommons.org/licenses/by-nc/3.0/legalcode>
