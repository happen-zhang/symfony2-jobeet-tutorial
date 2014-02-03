# 第五天：路由 #

*这一系列文章来源于Fabien Potencier，基于Symfony1.4编写的[Jobeet Tutirual](http://symfony.com/legacy/doc/jobeet?orm=Doctrine)。

## URLs ##

如果你在Jobeet首页点击任意一个*job*，你会发现这些URL看起来会是这样的：*/job/1/show*。如果你曾经使用PHP开发过网站，你可能更加熟悉这样的URL：*/job.php?id=1*。那么Symfony是怎么做到的呢？Symfony是如何根据URL来决定调用哪个*action*的呢？为什么*showAction($id)*中的*$id*的值就是检索*job*的id呢？这些问题我们会在这里一一解答。你在*src/Ibw/JobeetBundle/Resources/views/Job/index.html.twig*模板中可以看到如下代码：

```HTML
{{ path('ibw_job_show', { 'id': entity.id }) }}
```

这里使用了视图助手函数（template helper function）`path()`生成带有id值为1的*job* URL。*ibw\_job\_show*是路由使用的名称（具名路由），你可以在下面的内容中看到它们的定义。

## 路由配置 ##

在Symfony2中，路由通常是在*app/config/routing.yml*中进行配置的。这里导入（import）了指定bundle中的路由配置。在我们的教程中，文件*src/Ibw/JobeetBundle/Resources/config/routing.yml*就被导入其中：

```YAML
# app/config/routing.yml
ibw_jobeet:
    resource: "@IbwJobeetBundle/Resources/config/routing.yml"
    prefix:   /
```

现在，如果你可以打开*JobeetBundle*中的*routing.yml*文件，你可以看到它又导入了另外一个路由文件*src/Ibw/JobeetBundle/Resources/config/routing/job.yml*。在*routing.yml*中定义了两个路由，一个是URL样式为*/hello/{name}*的*ibw_jobeet_homepage*路由，另外一个则是定义给*JobController*的路由：

```YAML
# src/Ibw/JobeetBundle/Resources/config/routing.yml
IbwJobeetBundle_job:
    resource: "@IbwJobeetBundle/Resources/config/routing/job.yml"
    prefix: /job
 
ibw_jobeet_homepage:
    pattern:  /hello/{name}
    defaults: { _controller: IbwJobeetBundle:Default:index }
```

```YAML
# src/Ibw/JobeetBundle/Resources/config/routing/job.yml
ibw_job:
    pattern:  /
    defaults: { _controller: "IbwJobeetBundle:Job:index" }
 
ibw_job_show:
    pattern:  /{id}/show
    defaults: { _controller: "IbwJobeetBundle:Job:show" }
 
ibw_job_new:
    pattern:  /new
    defaults: { _controller: "IbwJobeetBundle:Job:new" }
 
ibw_job_create:
    pattern:  /create
    defaults: { _controller: "IbwJobeetBundle:Job:create" }
    requirements: { _method: post }
 
ibw_job_edit:
    pattern:  /{id}/edit
    defaults: { _controller: "IbwJobeetBundle:Job:edit" }
 
ibw_job_update:
    pattern:  /{id}/update
    defaults: { _controller: "IbwJobeetBundle:Job:update" }
    requirements: { _method: post|put }
 
ibw_job_delete:
    pattern:  /{id}/delete
    defaults: { _controller: "IbwJobeetBundle:Job:delete" }
    requirements: { _method: post|delete }
```

我们来仔细看看*ibw\_job\_show*路由。*ibw\_job\_show*路由定义的模式可以匹配_/*/show_的URL，这里的通配符\*代表的是id。在URL _/1/show_中，id变量的值为1，你可以在控制器中使用id变量的值。*_controller*是一个特殊的键（key），它告诉Symfony当URL匹配的时候要调用哪个*Controller*的*Action*，在这里的例子中应该调用的是*IbwJobeetBundle*中的*JobController*里的*showAction()*。

路由中的参数（比如*{id}*）是十分重要的，因为它们可以被*action*方法作为参数使用。

## 在开发（development）环境下的路由 ##

在开发环境下加载的路由文件是*app/config/routing_dev.yml*，这个文件包含了*Web Debug Toolbar*中需要用到的路由，而且在文件的末尾我们还可以看到*_main*中的路由是*routing.yml*。

## 自定义路由 ##

现在当你在浏览器中访问 */* URL的时候，你会得到一个*404页面*，那是因为*/*没有匹配到任何的已定义路由。*ibw\_jobeet\_homepage*路由匹配*/hello/jobeet* URL，然后调用*DefaultController*的*indexAction()*。我们来改变它来匹配 */* URL，然后调用*JobController*的*indexAction()*。修改的代码如下：

```YAML
# src/Ibw/JobeetBundle/Resources/config/routing.yml
# ...
ibw_jobeet_homepage:
    pattern:  /
    defaults: { _controller: IbwJobeetBundle:Job:index }
```

现在先去清除缓存（cache），然后你就能在浏览器中通过<http://jobeet.local>访问*Job*首页了。现在我们可以把Jobeet布局（layout）中logo的链接改成*ibw\_jobeet\_homepage*路由了：

```HTML
<!-- src/Ibw/JobeetBundle/Resources/views/layout.html.twig -->
<!-- ... -->    
    <h1><a href="{{ path('ibw_jobeet_homepage') }}">
        <img alt="Jobeet Job Board" src="{{ asset('bundles/ibwjobeet/images/logo.jpg') }}" />
    </a></h1>
<!-- ... -->
```

为了让一些页面能够被更加容易地打开，我们来把*job*页面的URL变得更加容易理解一些，就像下面那样：

    /job/sensio-labs/paris-france/1/web-developer

即使你不知道Jobeet是什么，也没有看过这个页面，但你可以通过这个URL就可以了解到Sensio Labs`正在Paris, France寻在一位Web developer。

下面的模式串可以匹配上面的URL：

    /job/{company}/{location}/{id}/{position}

编辑*job.yml*文件中的*ibw\_job\_show*路由：

```YAML
# src/Ibw/JobeetBundle/Resources/config/routing/job.yml
# ...
 
ibw_job_show:
    pattern:  /{company}/{location}/{id}/{position}
    defaults: { _controller: "IbwJobeetBundle:Job:show" }
```

现在我们需要传参数到路由中让它能工作：

```HTML
<!-- src/Ibw/JobeetBundle/Resources/views/Job/index.html.twig -->
<!-- ... -->
<a href="{{ path('ibw_job_show', { 'id': entity.id, 'company': entity.company, 'location': entity.location, 'position': entity.position }) }}">
    {{ entity.position }}
</a>
<!-- ... -->
```

你可以看看生成的URL，它们就像我们期望的那样：

    http://jobeet.local/app_dev.php/job/Sensio Labs/Paris,France/1/Web Developer

我们需要把URL中的所有非*ASCII*列值转化成一个*-*。打开*Job.php*文件，在类中添加下面的方法：

```PHP
// src/Ibw/JobeetBundle/Entity/Job.php
// ...
use Ibw\JobeetBundle\Utils\Jobeet as Jobeet;
 
class Job
{
    // ...
 
    public function getCompanySlug()
    {
        return Jobeet::slugify($this->getCompany());
    }
 
    public function getPositionSlug()
    {
        return Jobeet::slugify($this->getPosition());
    }
 
    public function getLocationSlug()
    {
        return Jobeet::slugify($this->getLocation());
    }
}
```

你必须把*use*语句（你可以查看PHP中怎么使用*命名空间*）放在类定义之前。

然后我们创建*src/Ibw/JobeetBundle/Utils/Jobeet.php*文件，并添加一个*slugify*方法：

```PHP
// src/Ibw/JobeetBundle/Utils/Jobeet.php
namespace Ibw\JobeetBundle\Utils;
 
class Jobeet
{
    static public function slugify($text)
    {
        // replace all non letters or digits by -
        $text = preg_replace('/\W+/', '-', $text);
 
        // trim and lowercase
        $text = strtolower(trim($text, '-'));
 
        return $text;
    }
}
```

我们已经定义了三个“虚拟的（virtual）”访问接口：*getCompanySlug()*，*getPositionSlug()*和*getLocationSlug()*，它们使用*slugify()*方法返回适当的列值。现在我们来替换在之前模板中使用的方法：

```HTML
<!-- src/Ibw/JobeetBundle/Resources/views/Job/index.html.twig -->
<!-- ... -->
<a href="{{ path('ibw_job_show', { 'id': entity.id, 'company': entity.companyslug, 'location': entity.locationslug, 'position': entity.positionslug}) }}">
   {{ entity.position }}
</a>
<!-- ... -->
```

## 路由Requirements ##

路由系统内建了验证功能。在每个路由中，可以使用*requirements*来定义正则表达式来规定路由中参数的取值：

```YAML
# src/Ibw/JobeetBundle/Resources/config/routing/job.yml
# ...
ibw_job_show:
    pattern:  /{company}/{location}/{id}/{position}
    defaults: { _controller: "IbwJobeetBundle:Job:show" }
    requirements:
        id:  \d+
 
# ...
```

上面的*requirements*强制id的值只能为数字，如果不是数字，那么路由则不能匹配成功。

## 调试路由 ##

当你添加一个自定义的路由之后，如果能通过可视化来得到更多路由的信息，我们相信这一定会对你很有帮助的。一个能够查看你应用的所有路由的好办法是使用*route:debug*命令。在你的项目下运行下面的命令：

    php app/console router:debug

这个命令会显示出应用中所有的路由配置。当然，你也可以显示一个指定的路由：

    php app/console router:debug ibw_job_show

## 最后的感言 ##

今天就讲这些啦！你可以从书中学习更多关于Symfony2路由的知识。

# 许可证 #

如果您需要转载的话，请尊重原作者的知识产权，您可以通过把如下链接放到您转载文章中的头部或者尾部，谢谢。

原文链接：<http://www.intelligentbee.com/blog/2013/08/07/symfony2-jobeet-day-1-starting-up-the-project/>

您可以在以下链接查看该许可证的全文：

![](../imgs/license.png)

<http://creativecommons.org/licenses/by-nc/3.0/legalcode>