# 第二天：Jobeet是什么 #

*这一系列文章来源于Fabien Potencier，基于Symfony1.4编写的[Jobeet Tutirual](http://symfony.com/legacy/doc/jobeet?orm=Doctrine)。

今天我们同样一行代码也不用写，但我们在[第一天](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-01/chapter-01.md)的时候已经搭建好了我们的开发环境，同时新建好了一个空的Symfony项目。

今天我们来看看Jobeet的功能说明。在埋头写代码之前，我们来描述一下Jobeet该做什么。下面部分描述的是我们想要实现的功能，它们和在第一版的Jobeet有着相同的story。

## 用户 Stories ##

在Jobeet中会有4种类型的用户：管理员（admin）（网站的管理员和拥有者），用户（user）（在网站中求职的用户），发布者（poster）（在网站中发布招聘信息的用户）和affiliate（转发其网站上招聘信息的用户）。

在[初始版本的Jobeet](http://symfony.com/legacy/doc/jobeet?orm=Doctrine)中，我们需要创建两个应用，分别是前端（frontend）和后端（backend）。前端用于用户和网站交互，后端用于管理员管理网站。我们使用的是Symfony2.3.2，在此我们将不再那样做了。我们将会只使用一个应用，在这个应用中包含管理员功能，而且它将会是安全的。

### Story F1：在首页，用户能够看到最新发布的职位 ###

当用户进入Jobeet网站的时候，他们能够看到一个职位列表。这些职位被放在不同的分类（category）中，而且它们还是按最新时间排序的。对于每个职位，我们只让它显示公司所在地（location），公司位置（position）和公司名称（company）这三类信息。

对于每个分类，我们将会显示出该分类中的前10条职位。点击该分类的链接会打开该分类的页面，在这个页面中会显示所有该分类职位（Story F2）。

在首页中，用户能够过滤职位列表，发布者能够发布一个新的职位。

### Story F2：用户能够查看分类中的所有职位 ###

当用户在首页中点击分类名或者“more jobs”链接的时候，他们将会看到所有该分类的职位，职位是按最新时间排序的。

这个分类的职位列表是可以分页的，每页20条职位。

### Story F3：一个用户能够用过关键字（keywords）来过滤职位 ###

用户能够输入过滤关键字来得到想要的职位信息。关键字在公司所在地（location），公司位置（position），分类（category），公司名称（company）字段中检索。

### Story F4：用户能够点击一个职位后查看职位详细信息 ###

用户能够从职位列表中选择一个职位查看详细信息。

### Story F5：用户发布职位 ###

用户能够发布职位，一个职位由如下信息组成：

* 公司名称（company）
* 类型（type）（全职、兼职或者是自由职业者）
* Logo（可选）
* 网址（URL）
* 公司位置（position）
* 公司所在地（location）
* 分类（categoty）（用户能够选择的分类）
* 职位描述（description）
* 应聘条件（How to apply）
* 公开（职位是否能够发布在affiliate的网站中）
* Email（发布者邮箱）

这个发布过程只有两步：第一步，用户填写好所有需要填写的职位信息，然后通过职位预览页面（previewing）检查职位信息。

不需要发布者（poster），即任何用户都能发布职位。修改一个指定的职位信息需要通过一个指定的URL（这个URL在职位发布后就生成了，它还带有一个token）。

每个职位的有效期是30天（管理员可以配置）。用户能够重新激活或者延续该职位多30天，但该职位过时不能超过5天。


### Story F6：用户需要申请才能成为affiliate ###

用户需要申请才能成为affiliate，他们是通过Jobeet API认证的。如果想要去申请，用户需要给出如下的信息：

* 姓名（name）
* Email
* 网站地址（Website URL）

affiliate必须被管理员激活（Story B3）。一旦affiliate被激活，他们将会收到一封email，里面包含了验证的token。

### Story F7：affiliate检索当前的职位列表 ###

affiliate能够通过他的affiliate token来调用API进行检索当前的职位列表。它可以返回XML、JSON、或者是YAML格式的列表。affiliate能够指定数量来返回的职位列表，同样也可以通过指定分类来返回职位列表。

### Story B1：管理员配置网站 ###

管理员能够编辑分类。

### Story B2：管理员管理职位 ###

管理员能够编辑和删除发布的职位。

### Story B3：管理员管理affiliate ###

管理员能够创建或者是编辑affiliate。管理员负责激活affiliate或者是禁用affiliate。当管理员激活了新的affiliate后，系统会创建一个唯一的token给affiliate。

作为一个开发者，你最好不要一开始就写代码。相反的时候，你首先得了解项目的需求和具体要做什么。今天就先到这，明天见！

# 许可证 #

如果您需要转载的话，请尊重原作者的知识产权，您可以通过把如下链接放到您转载文章中的头部或者尾部，谢谢。

原文链接：<http://www.intelligentbee.com/blog/2013/08/08/symfony2-jobeet-day-2-the-project/>

您可以在以下链接查看该许可证的全文：

![](../imgs/license.png)

<http://creativecommons.org/licenses/by-nc/3.0/legalcode>