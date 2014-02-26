# 第四天：控制器和视图 #

*这一系列文章来源于Fabien Potencier，基于Symfony1.4编写的[Jobeet Tutirual](http://symfony.com/legacy/doc/jobeet?orm=Doctrine)。

在昨天的内容中，我们已经生成了`JobController`控制器，那么我们今天就来对`JobController`控制器进行自定义操作。Symfony自动生成的`JobController`控制器已经有了我们所需要的大部分代码了：

* 列表（list）页面
* 创建（new create）页面
* 编辑（edit update）页面
* 删除（delete）页面

现在我们在已有代码的基础上进行重构，让Jobeet的页面更加接近[Jobeet的模型](http://symfony.com/legacy/doc/jobeet/1_4/en/02?orm=Doctrine#chapter_02_the_project_user_stories)。

## MVC架构 ##

对于Web开发来说，现在最常见的代码组织方式就是使用MVC设计模式。简而言之，**MVC**设计模式定义了一种组织和编写代码的规范，这样会使得你的代码和结构看起来更加地合理。MVC模式把代码分为三个层次：

* Model层：定义业务逻辑（数据库操作也属于这层）。我们已经知道了在Symfony中，所有和Model层相关的类都放在*Entity/*目录下。
* View层：与用户交互的图形界面（模板引擎属于这个层）。Symfony 2.3.2的View层使用的模板引擎主要是*Twig*。Symfony中的视图文件被放在各个*Resources/views/*目录下，我们待会就来介绍它。
* Controller层：它通过Model层来获取数据，然后对数据进行处理后把结果渲染到View层并返回给客户端（浏览器）。在教程的第一天中我们就简单地介绍了怎么样通过Symfony来访问页面，我们已经看到所有的请求都被**前端控制器（front controller）**给管理起来了（*app.php*和*app_dev.php*），而前端控制器其实把真正的工作交给action来处理。

## 布局 ##

如果你有仔细观察[Jobeet的模型](http://symfony.com/legacy/doc/jobeet/1_4/en/02?orm=Doctrine#chapter_02_the_project_user_stories)，你可能会注意到它们的每个页面看起来都很相似。我们都知道，不管是PHP代码还是HTML代码，重复的代码给人的感觉就是很差，因此我们需要找到一个方法来提取重复的代码，这样做我们的程序才会更加简洁和容易维护。

我们有一种解决方法是，我们可以先定义出页面的*header*和*footer*，然后把它们包含在每个页面中。另一种更好的办法则是使用一种设计模式来解决问题：[装饰者模式](http://en.wikipedia.org/wiki/Decorator_pattern)。**装饰者模式**解决问题的方式是：即先把一个子模板渲染完成后再把它放到一个全局的模板中去，而那个全局的模板叫做**布局（layout）**。

Symfony2中并没有提供默认的布局，所以我们需要来创建一个布局来装饰其它的子页面模板。

我们在*src/Ibw/JobeetBundle/Resources/views/*目录下创建一个新文件*layout.html.twig*，它的代码如下：

```HTML
<!-- src/Ibw/JobeetBundle/Resources/views/layout.html.twig -->
<!DOCTYPE html>
<html>
    <head>
        <title>
            {% block title %}
                Jobeet - Your best job board
            {% endblock %}
        </title>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
        {% block stylesheets %}
            <link rel="stylesheet" href="{{ asset('bundles/ibwjobeet/css/main.css') }}" type="text/css" media="all" />
        {% endblock %}
        {% block javascripts %}
        {% endblock %}
        <link rel="shortcut icon" href="{{ asset('bundles/ibwjobeet/images/favicon.ico') }}" />
    </head>
    <body>
        <div id="container">
            <div id="header">
                <div class="content">
                    <h1><a href="{{ path('ibw_job') }}">
                        <img src="{{ asset('bundles/ibwjobeet/images/logo.jpg') }}" alt="Jobeet Job Board" />
                    </a></h1>
 
                    <div id="sub_header">
                        <div class="post">
                            <h2>Ask for people</h2>
                            <div>
                                <a href="{{ path('ibw_job') }}">Post a Job</a>
                            </div>
                        </div>
 
                        <div class="search">
                            <h2>Ask for a job</h2>
                            <form action="" method="get">
                                <input type="text" name="keywords" id="search_keywords" />
                                <input type="submit" value="search" />
                                <div class="help">
                                    Enter some keywords (city, country, position, ...)
                                </div>
                            </form>
                        </div>
                    </div>
                </div>
            </div>
 
           <div id="content">
               {% for flashMessage in app.session.flashbag.get('notice') %}
                   <div class="flash_notice">
                       {{ flashMessage }}
                   </div>
               {% endfor %}
 
               {% for flashMessage in app.session.flashbag.get('error') %}
                   <div class="flash_error">
                       {{ flashMessage }}
                   </div>
               {% endfor %}
 
               <div class="content">
                   {% block content %}
                   {% endblock %}
               </div>
           </div>
 
           <div id="footer">
               <div class="content">
                   <span class="symfony">
                       <img src="{{ asset('bundles/ibwjobeet/images/jobeet-mini.png') }}" />
                           powered by <a href="http://www.symfony.com/">
                           <img src="{{ asset('bundles/ibwjobeet/images/symfony.gif') }}" alt="symfony framework" />
                       </a>
                   </span>
                   <ul>
                       <li><a href="">About Jobeet</a></li>
                       <li class="feed"><a href="">Full feed</a></li>
                       <li><a href="">Jobeet API</a></li>
                       <li class="last"><a href="">Affiliates</a></li>
                   </ul>
               </div>
           </div>
       </div>
   </body>
</html>
```

## Twig区块（Blocks） ##

Twig是Symfony默认的模板引擎，你可以像我们在上面的代码中一样定义区块（blocks）。在Twig的block中，我们可以给它们定义默认的内容（比如上面的*title block*）。待会我们将会看到如何在子模板中替换block中默认的内容或者是继承block中的内容。

现在我们想要使用layout的话，我们还需要去编辑子模板（*src/Ibw/JobeetBundle/Resources/views/Job/*目录下的*index*，*edit*、*new*和*show*）去继承layout并重写（overwrite）layout中的*block content*。子模板的结构就像下面那样：

```HTML
{% extends 'IbwJobeetBundle::layout.html.twig' %}
 
{% block content %}
    <!-- original body block code goes here -->
{% endblock %}
```

## Stylesheets，Javascripts和图片 ##

我们的教程不是讲授怎样进行Web设计，所以这里已经准备好了所有需要使用到的资源。[点击这里下载图片](http://www.intelligentbee.com/blog/downloads/jobeet-images.zip)，然后把这些图片放在*src/Ibw/JobeetBundle/Resources/public/images/*目录下，[点击这里下载css文件](http://www.intelligentbee.com/blog/downloads/jobeet-css.zip)，然后把css文件放在*src/Ibw/JobeetBundle/Resources/public/css/*目录下。

现在运行下面的命令：

    php app/console assets:install web --symlink

上面的命令为了是告诉Symfony把*css*和*图片*资源作为公用（外部可以访问到）的。

在css文件夹中，我们可以看到有四个文件：*admin.css*，*job.css*，*jobs.css*和*main.css*。*main.css*需要被所有的Jobeet页面使用，所以我们把它放在layout中的*stylesheet twig block*里。剩下的其它三个css文件我们在需要用到它们的时侯才把它们加入到页面中。

为了能够在模板中加入新的css文件，我们需要重写*stylesheet block*中的内容，但需要先调用父block，这样才不会把*main.css*给覆盖掉（看代码就明白了）。

```HTML
<!-- src/Ibw/JobeetBundle/Resources/views/Job/index.html.twig -->
{% extends 'IbwJobeetBundle::layout.html.twig' %}
 
{% block stylesheets %}
    {{ parent() }}
    <link rel="stylesheet" href="{{ asset('bundles/ibwjobeet/css/jobs.css') }}" type="text/css" media="all" />
{% endblock %}
 
<!-- rest of the code -->
```

```HTML
<!-- src/Ibw/JobeetBundle/Resources/views/Job/show.html.twig -->
{% extends 'IbwJobeetBundle::layout.html.twig' %}
 
{% block stylesheets %}
    {{ parent() }}
    <link rel="stylesheet" href="{{ asset('bundles/ibwjobeet/css/job.css') }}" type="text/css" media="all" />
{% endblock %}
 
<!-- rest of the code -->
```

## Job首页Action ##

一个action就对应着一个控制器中的一个操作。简而言之，action就是一个方法。对于访问Job（职位）首页来说，此时的控制器是JobController，而对应的action是*indexAction()*。*indexAction()*会取出数据库中所有的Job数据。

```PHP
// src/Ibw/JobeetBundle/Controller/JobController.php
// ...
 
public function indexAction()
{
    $em = $this->getDoctrine()->getManager();
 
    $entities = $em->getRepository('IbwJobeetBundle:Job')->findAll();
 
    return $this->render('IbwJobeetBundle:Job:index.html.twig', array(
        'entities' => $entities
    ));
}
 
// ...
```

我们来仔细看看上面的代码：在*indexAction()*方法中得到了一个Doctrine实体管理对象（entity manager object），这个对象会负责把数据持久化到数据库或者把数据从数据库中取出。repository则生成一个查询去检索数据库中的Job数据，它会返回一个Job类型的Doctrine ArrayColletion对象，然后把这个对象传递给模板（视图）。

## Job首页模板 ##

*index.html.twig*模板有一个显示所有Job数据的*HTML*表格。下面是它的代码：

```HTML
<!-- src/Ibw/JobeetBundle/Resources/views/Job/index.html.twig -->
{% extends 'IbwJobeetBundle::layout.html.twig' %}
 
{% block stylesheets %}
    {{ parent() }}
    <link rel="stylesheet" href="{{ asset('bundles/ibwjobeet/css/jobs.css') }}" type="text/css" media="all" />
{% endblock %}
 
{% block content %}
    <h1>Job list</h1>
 
    <table class="records_list">
        <thead>
            <tr>
                <th>Id</th>
                <th>Type</th>
                <th>Company</th>
                <th>Logo</th>
                <th>Url</th>
                <th>Position</th>
                <th>Location</th>
                <th>Description</th>
                <th>How_to_apply</th>
                <th>Token</th>
                <th>Is_public</th>
                <th>Is_activated</th>
                <th>Email</th>
                <th>Expires_at</th>
                <th>Created_at</th>
                <th>Updated_at</th>
                <th>Actions</th>
            </tr>
        </thead>
        <tbody>
        {% for entity in entities %}
            <tr>
                <td><a href="{{ path('ibw_job_show', { 'id': entity.id }) }}">{{ entity.id }}</a></td>
                <td>{{ entity.type }}</td>
                <td>{{ entity.company }}</td>
                <td>{{ entity.logo }}</td>
                <td>{{ entity.url }}</td>
                <td>{{ entity.position }}</td>
                <td>{{ entity.location }}</td>
                <td>{{ entity.description }}</td>
                <td>{{ entity.howtoapply }}</td>
                <td>{{ entity.token }}</td>
                <td>{{ entity.ispublic }}</td>
                <td>{{ entity.isactivated }}</td>
                <td>{{ entity.email }}</td>
                <td>{% if entity.expiresat %}{{ entity.expiresat|date('Y-m-d H:i:s') }}{% endif%}</td>
                <td>{% if entity.createdat %}{{ entity.createdat|date('Y-m-d H:i:s') }}{% endif%}</td>
                <td>{% if entity.updatedat %}{{ entity.updatedat|date('Y-m-d H:i:s') }}{% endif%}</td>
                <td>
                    <ul>
                        <li>
                            <a href="{{ path('ibw_job_show', { 'id': entity.id }) }}">show</a>
                        </li>
                        <li>
                            <a href="{{ path('ibw_job_edit', { 'id': entity.id }) }}">edit </a>
                        </li>
                    </ul>
                </td>
            </tr>
        {% endfor %}
        </tbody>
    </table>
 
    <ul>
        <li>
            <a href="{{ path('ibw_job_new') }}">
                Create a new entry
            </a>
        </li>
    </ul>
{% endblock %}
```

我们来清除一些不需要显示出来的列（columns）。用下面的代码替换掉`twig block content`：

```HTML
<!-- src/Ibw/JobeetBundle/Resources/views/Job/index.html.twig -->
{% block content %}
    <div id="jobs">
        <table class="jobs">
            {% for entity in entities %}
                <tr class="{{ cycle(['even', 'odd'], loop.index) }}">
                    <td class="location">{{ entity.location }}</td>
                    <td class="position">
                        <a href="{{ path('ibw_job_show', { 'id': entity.id }) }}">
                            {{ entity.position }}
                        </a>
                    </td>
                    <td class="company">{{ entity.company }}</td>
                </tr>
            {% endfor %}
        </table>
    </div>
{% endblock %}
```

![](imgs/04-01.png)

## Job页面模板 ##

现在我们来自定义Job页面的模板。打开*show.html.twig*，用下面的代码替换它：

```HTML
<!-- src/Ibw/JobeetBundle/Resources/views/Job/show.html.twig -->
{% extends 'IbwJobeetBundle::layout.html.twig' %}
 
{% block title %}
    {{ entity.company }} is looking for a {{ entity.position }}
{% endblock %}
 
{% block stylesheets %}
    {{ parent() }}
    <link rel="stylesheet" href="{{ asset('bundles/ibwjobeet/css/job.css') }}" type="text/css" media="all" />
{% endblock %}
 
{% block content %}
    <div id="job">
        <h1>{{ entity.company }}</h1>
        <h2>{{ entity.location }}</h2>
        <h3>
            {{ entity.position }}
            <small> - {{ entity.type }}</small>
        </h3>
 
        {% if entity.logo %}
            <div class="logo">
                <a href="{{ entity.url }}">
                    <img src="/uploads/jobs/{{ entity.logo }}"
                        alt="{{ entity.company }} logo" />
                </a>
            </div>
        {% endif %}
 
        <div class="description">
            {{ entity.description|nl2br }}
        </div>
 
        <h4>How to apply?</h4>
 
        <p class="how_to_apply">{{ entity.howtoapply }}</p>
 
        <div class="meta">
            <small>posted on {{ entity.createdat|date('m/d/Y') }}</small>
        </div>
 
        <div style="padding: 20px 0">
            <a href="{{ path('ibw_job_edit', { 'id': entity.id }) }}">
                Edit
            </a>
        </div>
    </div>
{% endblock %}
```

![](imgs/04-02.png)

## Job页面的Action ##

Job页面是通过*show*这个动作生成的，*showAction()*被定义在JobController：

```PHP
// src/Ibw/JobeetBundle/Controller/JobController.php
public function showAction($id)
{
    $em = $this->getDoctrine()->getManager();
 
    $entity = $em->getRepository('IbwJobeetBundle:Job')->find($id);
 
    if (!$entity) {
        throw $this->createNotFoundException('Unable to find Job entity.');
    }
 
    $deleteForm = $this->createDeleteForm($id);
 
    return $this->render('IbwJobeetBundle:Job:show.html.twig', array(
        'entity' => $entity,
        'delete_form' => $deleteForm->createView(),
    ));
}
```

就像在*indexAction()*中一样，*IbwJobeetBundle repository*是用来检索一个指定的Job数据，但这次使用的是*find()*方法。*find()*方法的参数是能一个够唯一标识Job的标识符，即**主键**。在下面的内容中我们会探索*showAction()*是怎么样得到Job数据的主键*$id*的值。

如果用户需要访问的Job页面不存在，`throw $this->createNotFoundException()`将会把我们重定向到*404页面*。

异常（excpetions）在*production*环境和*devlopment*环境下显示的页面会不同。

![](imgs/04-03.png)

![](imgs/04-04.png)

今天我们就先到这啦，明天我们来将会讲解路由（*routing*）功能。

# 许可证 #

如果您需要转载的话，请尊重原作者的知识产权，您可以通过把如下链接放到您转载文章中的头部或者尾部，谢谢。

原文链接：<http://www.intelligentbee.com/blog/2013/08/10/symfony2-jobeet-day-4-the-controller-and-the-view/>

您可以在以下链接查看该许可证的全文：

![](../imgs/license.png)

<http://creativecommons.org/licenses/by-nc/3.0/legalcode>
