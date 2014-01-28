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

我学习PHP也是有段时间了，接触过的PHP开发框架也就少数几个,有ThinkPHP、CakePHP等，而且它们大都很简单，开发效率也挺高的。早之前就听过了Symfony2，但是一直没想去了解它。有一天心血来潮，就Google了一些资源，于是就发现了这一些列不错的教程。教程中的内容和《Ruby on Rails Tutorial》差不多，都是以一个真是的案例来给大家讲解框架的使用，其中有教你怎样使用Symfony2进行开发，怎么样进行测试，怎么编写命令行任务等等。又鉴于国内关于Symfony2的资料稀缺，我认为这份资料可以说是很珍贵的。所以，我试着翻译（虽然并不是很难），目的是想让更多的小伙伴能见识一下Symfony2的强大，PHP的强大。

# 从何入手 #

学习一个大而全的框架无疑会增加我们的学习成本，而且相对于其它的PHP开发框架，Symfony是复杂了。如何选择一个PHP框架来进行开发自己的应用呢？首先我们当然要对备选的框架的功能和特性有点了解吧。如何才能够快速地了解一个框架在开发中所体现出来的特性呢？那么当然是通过一个实例啦。用实例和最佳实践来展示框架或者语言的特性，那么这一些列Tutorial是你不错的选择。

# 目录 #

* [第一天：开始你的Jobeet项目]()
* [第二天：Jobeet是什么]()
* [第三天：数据模型]()
* [第四天：控制器和视图]()
* [第五天：路由]()
* [第六天：更多的数据模型]()
* [第七天：实现分类页面]()
* [第八天：单元测试]()
* [第九天：功能测试]()
* [第十天：表单]()
* [第十一天：对你的表单进行测试]()
* [第十二天：后台管理工具包—Sonata Admin]()
* [第十三天：实现程序的安全性]()
* [第十四天：完成订阅功能]()
* [第十五天：为你的应用提供接口]()
* [第十六天：Mailer]()
* [第十七天：实现搜索功能]()
* [第十八天：使用Ajax]()
* [第十九天：国际化和本地化]()

# 感谢 #

感谢[IntelligentBee](http://www.intelligentbee.com/)提供这么好的文章。

# 许可证 #

这些文章基于Attribution-NonCommercial 3.0 Unported license发布。**您不需要为本教程付钱。**

您有权复制、分发、修改或展示本教程的内容，但请您指定声明文章的出处（<http://www.intelligentbee.com/>），也请勿将其用于任何商业用途。

您可以在以下链接查看该许可证的全文：

![](imgs/license.png)

<http://creativecommons.org/licenses/by-nc/3.0/legalcode>
