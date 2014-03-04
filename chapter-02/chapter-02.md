# 第二天：Jobeet是什么 #

*这一系列文章来源于Fabien Potencier，基于Symfony1.4编写的[Jobeet Tutirual](http://symfony.com/legacy/doc/jobeet?orm=Doctrine)。

今天我们同样一行代码也不用写，我们在[第一天](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-01/chapter-01.md)的时候就已经把开发环境搭建起来了，同时我们还新建好了一个空的*Symfony*项目。

今天我们来看看需要为*Jobeet*实现哪些功能。在我们埋头写代码之前，我们先来描述一下*Jobeet*究竟该做些什么。下面的内容描述的是我们想要为*Jobeet*实现的功能，这些功能和[第一版中的Jobeet](http://symfony.com/legacy/doc/jobeet?orm=Doctrine)有着相同的*story*。

## 用户 *Stories* ##

在*Jobeet*中一共会有4种类型的用户：管理员（admin）（网站的管理者和拥有者），用户（user）（在网站中需找求职机会的用户），发布者（poster）（在网站中发布招聘信息的用户）和*Affiliate*（转发其网站上招聘信息的用户）。

在[初始版本的Jobeet](http://symfony.com/legacy/doc/jobeet?orm=Doctrine)中，我们需要创建两个应用，分别是前端（frontend）应用和后端（backend）应用。前端负责用户和网站的交互，后端负责管理员管理网站。因为我们这次使用的是*Symfony2.3.2*来进行开发，所以这次我们就不再那样做了。我们将会只使用一个应用，在这个应用中包含前端和管理员功能，而且我们还会为它添加安全机制。

### *Story F1*：在首页，用户能够看到最新发布的职位 ###

当用户进入*Jobeet*网站的时候，用户能够看到一个职位列表。这些职位被放在不同的分类（category）中，而且它们还是按最新发布时间排序的。对于每个职位，我们只让它显示公司所在地（location），公司位置（position）和公司名称（company）这三种信息。

对于每个分类，我们将会显示出该分类中的前10条职位信息。点击该分类的链接会进入该分类的页面，在这个页面中会列出所有该分类下的职位（Story F2）。

在首页中，用户能够关键字来筛选职位列表，而且发布者还能够发布一个新的职位信息。

### *Story F2*：用户能够查看分类中的所有职位 ###

当用户在首页中点击分类名或者“*more jobs*”链接的时候，他们将会跳转到该分类的页面，在页面中可以看到所有属于该分类下的职位信息，而且职位信息是按最新发布时间排序的。

分类页面中的职位列表是可以进行分页的，每页显示出20条职位。

### *Story F3*：一个用户能够通过关键字（keywords）来筛选职位信息 ###

用户能够输入过滤关键字来得到想要的职位信息。关键字会在每条职位信息中的公司所在地（location），公司位置（position），分类（category）和公司名称（company）字段中进行检索。

### *Story F4*：用户能够点击一个职位信息后查看该职位的详细信息 ###

用户能够从职位列表中选择一个职位查看详细信息。

### *Story F5*：用户发布职位 ###

用户能够发布职位，一个职位信息由如下数据组成：

* 公司名称（company）
* 类型（type）（全职、兼职或者是自由职业者）
* Logo（可选）
* 网址（URL）
* 公司位置（position）
* 公司所在地（location）
* 分类（categoty）（用户能够选择的分类）
* 职位描述（description）
* 应聘条件（How to apply）
* 公开（职位是否能够发布在*Affiliate*的网站中）
* Email（发布者邮箱）

这个发布过程只有两步：第一步，用户填写好所有需要填写的职位信息表单，然后通过职位预览页面（previewing）查看即将发布的职位信息。

我们不需要单独创建出发布者（poster），即任何用户都能发布职位。那么我们就需要为修改职位信息添加一些限制，修改一个指定的职位信息时需要通过一个指定的*URL*才能够进行操作（这个*URL*在职位发布后就生成了，这个*URL*中还带有一个*token*）。

每个职位的有效期是30天（管理员可以配置）。用户能够重新激活或者延续该职位的期限多30天，但这个操作必须得在职位信息过期5天之内进行（否则就不能重新激活或者延期了）。

### *Story F6*：用户需要申请才能成为*Affiliate* ###

用户需要申请才能成为*Affiliate*，他们需要通过*Jobeet API*的认证才能生效。如果想要去申请*Affiliate*，用户需要给出如下的信息：

* 姓名（name）
* Email
* 网站地址（Website URL）

*Affiliate*必须被管理员激活（*Story B3*）。一旦*Affiliate*被激活，他们将会收到一封*email*，*email*内容里面包含了进行登录验证的*token*。

### *Story F7*：*Affiliate*能够检索当前的职位列表 ###

*Affiliate*能够通过他/她的*Affiliate token*来调用*API*进行检索当前的职位列表。*API*接口可以返回*XML*、*JSON*、或者是*YAML*格式的列表信息。*Affiliate*能够返回指定数量的职位列表，而且也可以返回指定分类下的职位列表。

### *Story B1*：管理员配置网站 ###

管理员能够编辑分类。

### *Story B2*：管理员管理职位信息 ###

管理员能够编辑和删除发布的职位。

### *Story B3*：管理员管理*Affiliate* ###

管理员能够创建或者是编辑*Affiliate*。管理员可以激活*Affiliate*或者是禁用*Affiliate*。当管理员激活了新的*Affiliate*后，系统会创建一个唯一的*token*给该*Affiliate*。

作为一个开发者，我们最好不要一开始上来就写代码。通常情况下，我们首先得了解项目的需求和具体要做的是什么，这一步十分重要。那好吧，今天我们就先到这啦，明天见！

# 许可证 #

如果您需要转载的话，请尊重原作者的知识产权，您可以通过把如下链接放到您转载文章中的头部或者尾部，谢谢。

原文链接：<http://www.intelligentbee.com/blog/2013/08/08/symfony2-jobeet-day-2-the-project/>

您可以在以下链接查看该许可证的全文：

![](../imgs/license.png)

<http://creativecommons.org/licenses/by-nc/3.0/legalcode>
