# 第十四天：订阅 #

*这一系列文章来源于Fabien Potencier，基于Symfony1.4编写的[Jobeet Tutirual](http://symfony.com/legacy/doc/jobeet?orm=Doctrine)。

如果你正在寻找工作，那么你可能十分想要第一时间就能了解到最新发布的*Job*信息。我们总不可能每时每刻都守在电脑面前，不断地刷新网站是否有发布新的*Job*信息吧，那样很不方便，所以我们会为*Jobeet*用户提供订阅（feeds）功能，这样用户就能实时地接受到最新发布的*Job*信息了。

## 模板格式 ##

模板是能把内容渲染成任意一种格式的通用方式，但我们在大多数情况下都是使用模板来渲染HTML内容。当然，模板也很容易生成Javascript，CSS，XML或者其它任意类型格式的内容。

我们来举个例子，比如说我们需要把相同的“资源”按照不同的格式渲染出来。为了把一个文章的索引页渲染成XML格式，我们只需简单地把把格式名包含进模板名中即可：

* XML模板名：AcmeArticleBundle:Article:index.xml.twig
* XML模板文件名：index.xml.twig

事实上，这仅仅只是一个命名约定，模板会基于它原本的格式，而并不会再被渲染成其它的格式了。.

在多数情况下，你可能希望只使用单个控制器来渲染用户期望得到的指定格式的内容。出于这个原因，我们通常的做法会是这样的：

```PHP
public function indexAction()
{
    $format = $this->getRequest()->getRequestFormat();
 
    return $this->render('AcmeBlogBundle:Blog:index.'.$format.'.twig');
}
```

*Request*对象的*getRequestFormat()*方法默认返回的值是*html*，但它能够基于用户请求的格式返回任意类型的格式。请求格式通常由路由进行管理，路由是可以配置的，所以*/contact*的请求格式是html（因为默认是html格式），而*/contact.xml*的请求格式是xml。

为了创建带有格式参数的链接，我们需要在参数列表中加入*_format*键：

```HTML
<a href="{{ path('article_show', {'id': 123, '_format': 'pdf'}) }}">
    PDF Version
</a>
```

## 订阅 ##

### 订阅最新的*Job*信息 ###

支持不同格式的内容就像创建模板一样简单。为了给最新发布的*Job*信息创建一个*Atom*格式的订阅，我们需要创建一个*index.atom.twig*模板：

```ATOM
<!-- src/Ibw/JobeetBundle/Resources/views/Job/index.atom.twig -->
<?xml version="1.0" encoding="utf-8"?>
<feed xmlns="http://www.w3.org/2005/Atom">
    <title>Jobeet</title>
    <subtitle>Latest Jobs</subtitle>
    <link href="" rel="self"/>
    <link href=""/>
    <updated></updated>
    <author><name>Jobeet</name></author>
    <id>Unique Id</id>
 
    <entry>
        <title>Job title</title>
        <link href="" />
        <id>Unique id</id>
        <updated></updated>
        <summary>Job description</summary>
        <author><name>Company</name></author>
    </entry>
</feed>
```

在*Jobeet*页面的页脚部分更新订阅的链接：

```HTML
<!-- src/Ibw/JobeetBundle/Resources/views/layout.html.twig -->
<!-- ... -->
 
<li class="feed"><a href="{{ path('ibw_job', {'_format': 'atom'}) }}">Full feed</a></li>
 
<!-- ... -->
```

在*layout*的*<head>*标签部分添加一个*<link>*标签，这让浏览器能够自动察觉到我们的订阅链接：

```HTML
<!-- src/Ibw/JobeetBundle/Resources/views/layout.html.twig -->
<!-- ... -->
 
<link rel="alternate" type="application/atom+xml" title="Latest Jobs" href="{{ url('ibw_job', {'_format': 'atom'}) }}" />
 
<!-- ... -->
```

修改*JobController::indexAction()*方法能够按照*_format*参数来渲染模板：

```PHP
// src/Ibw/JobeetBundle/Controller/JobController.php
// ...
 
$format = $this->getRequest()->getRequestFormat();
 
return $this->render('IbwJobeetBundle:Job:index.'.$format.'.twig', array(
    'categories' => $categories
));
 
// ...
```

把*Atom*模板的头部分替换成下面的代码：

```ATOM
<!-- src/Ibw/JobeetBundle/Resources/views/Job/index.atom.twig -->
<!-- ... -->
 
    <title>Jobeet</title>
    <subtitle>Latest Jobs</subtitle>
    <link href="{{ url('ibw_job', {'_format': 'atom'}) }}" rel="self"/>
    <link href="{{ url('ibw_jobeet_homepage') }}"/>
    <updated>{{ lastUpdated }}</updated>
    <author><name>Jobeet</name></author>
    <id>{{ feedId }}</id>
 
<!-- ... -->
```

在*JobController::indexAction()*方法中，我们需要把*lastUpdated*和*feedId*传递给模板：

```PHP
// src/Ibw/JobeetBundle/Controller/JobController.php
// ...
 
        $latestJob = $em->getRepository('IbwJobeetBundle:Job')->getLatestPost();
 
        if($latestJob) {
            $lastUpdated = $latestJob->getCreatedAt()->format(DATE_ATOM);
        } else {
            $lastUpdated = new \DateTime();
            $lastUpdated = $lastUpdated->format(DATE_ATOM);
        }
 
        $format = $this->getRequest()->getRequestFormat();
        return $this->render('IbwJobeetBundle:Job:index.'.$format.'.twig', array(
               'categories' => $categories,
               'lastUpdated' => $lastUpdated,
               'feedId' => sha1($this->get('router')->generate('ibw_job', array('_format'=> 'atom'), true)),
        ));
// ...
```

为了得到最新发布信息的日期，我们需要在*JobRepository*类中添加*getLatestPost()*方法：

```PHP
// src/Ibw/JobeetBundle/Repository/JobRepository.php
// ...
 
    public function getLatestPost($category_id = null)
    {
        $query = $this->createQueryBuilder('j')
            ->where('j.expires_at > :date')
            ->setParameter('date', date('Y-m-d H:i:s', time()))
            ->andWhere('j.is_activated = :activated')
            ->setParameter('activated', 1)
            ->orderBy('j.expires_at', 'DESC')
            ->setMaxResults(1);
 
        if($category_id) {
            $query->andWhere('j.category = :category_id')
                ->setParameter('category_id', $category_id);
        }
 
        try{
            $job = $query->getQuery()->getSingleResult();
        } catch(\Doctrine\Orm\NoResultException $e){
            $job = null;
        }
 
        return $job;    
    }
// ...
```

订阅中的*entity*可以通过下面的代码生成：

```ATOM
<!-- src/Ibw/JobeetBundle/Resources/views/Job/index.atom.twig -->
{% for category in categories %}
    {% for entity in category.activejobs %}
        <entry>
            <title>{{ entity.position }} ({{ entity.location }})</title>
            <link href="{{ url('ibw_job_show', { 'id': entity.id, 'company': entity.companyslug, 'location': entity.locationslug, 'position': entity.positionslug }) }}" />
            <id>{{ entity.id }}</id>
            <updated>{{ entity.createdAt.format(constant('DATE_ATOM')) }}</updated>
            <summary type="xhtml">
                <div xmlns="http://www.w3.org/1999/xhtml">
                    {% if entity.logo %}
                        <div>
                            <a href="{{ entity.url }}">
                                <img src="http://{{ app.request.host }}/uploads/jobs/{{ entity.logo }}" alt="{{ entity.company }} logo" />
                            </a>
                        </div>
                    {% endif %}
                    <div>
                        {{ entity.description|nl2br }}
                    </div>
                    <h4>How to apply?</h4>
                    <p>{{ entity.howtoapply }}</p>
                </div>
            </summary>
            <author><name>{{ entity.company }}</name></author>
        </entry>
    {% endfor %}
{% endfor %}
```

### 订阅分类中最新的*Job*信息 ###

*Jobeet*的目标之一就是帮助人们找到期望的工作，所以我们需要提供不同类型的*Job*信息订阅。

首先我们来更新模板中分类订阅的链接：

```HTML
<!-- src/Ibw/JobeetBundle/Resources/views/Job/index.html.twig -->
<div class="feed">
    <a href="{{ path('IbwJobeetBundle_category', { 'slug': category.slug, '_format': 'atom' }) }}">Feed</a>
</div>
```

```HTML
<!-- src/Ibw/JobeetBundle/Resources/views/Category/show.html.twig -->
<div class="feed">
    <a href="{{ path('IbwJobeetBundle_category', { 'slug': category.slug, '_format': 'atom' }) }}">Feed</a>
</div>
```

修改*CategoryController::showAction()*方法渲染适当的模板：

```PHP
// src/Ibw/JobeetBundle/Controller/CategoryController.php
// ...
    public function showAction($slug, $page)
    {
        $em = $this->getDoctrine()->getManager();
 
        $category = $em->getRepository('IbwJobeetBundle:Category')->findOneBySlug($slug);
 
        if (!$category) {
            throw $this->createNotFoundException('Unable to find Category entity.');
        }
 
        $latestJob = $em->getRepository('IbwJobeetBundle:Job')->getLatestPost($category->getId());
 
        if($latestJob) {
            $lastUpdated = $latestJob->getCreatedAt()->format(DATE_ATOM); 
        } else {
            $lastUpdated = new \DateTime();
            $lastUpdated = $lastUpdated->format(DATE_ATOM);
        }
 
        $total_jobs = $em->getRepository('IbwJobeetBundle:Job')->countActiveJobs($category->getId());
        $jobs_per_page = $this->container->getParameter('max_jobs_on_category');
        $last_page = ceil($total_jobs / $jobs_per_page);
        $previous_page = $page > 1 ? $page - 1 : 1;
        $next_page = $page < $last_page ? $page + 1 : $last_page; 
        $category->setActiveJobs($em->getRepository('IbwJobeetBundle:Job')->getActiveJobs($category->getId(), $jobs_per_page, ($page - 1) * $jobs_per_page));
 
        $format = $this->getRequest()->getRequestFormat();
 
        return $this->render('IbwJobeetBundle:Category:show.' . $format . '.twig', array(
            'category' => $category,
            'last_page' => $last_page,
            'previous_page' => $previous_page,
            'current_page' => $page,
            'next_page' => $next_page,
            'total_jobs' => $total_jobs,
            'feedId' => sha1($this->get('router')->generate('IbwJobeetBundle_category', array('slug' => $category->getSlug(), 'format' => 'atom'), true)),
            'lastUpdated' => $lastUpdated    
        ));
    }
```

最后，我们创建*show.atom.twig*模板：

```ATOM
<!-- src/Ibw/JobeetBundle/Resources/views/Category/show.atom.twig -->
<?xml version="1.0" encoding="utf-8"?>
<feed xmlns="http://www.w3.org/2005/Atom">
    <title>Jobeet ({{ category.name }})</title>
    <subtitle>Latest Jobs</subtitle>
    <link href="{{ url('IbwJobeetBundle_category', { 'slug': category.slug, '_format': 'atom' }) }}" rel="self" />
    <updated>{{ lastUpdated }}</updated>
    <author><name>Jobeet</name></author>
    <id>{{ feedId }}</id>
 
    {% for entity in category.activejobs %}
        <entry>
            <title>{{ entity.position }} ({{ entity.location }})</title>
            <link href="{{ url('ibw_job_show', { 'id': entity.id, 'company': entity.companyslug, 'location': entity.locationslug, 'position': entity.positionslug }) }}" />
            <id>{{ entity.id }}</id>
            <updated>{{ entity.createdAt.format(constant('DATE_ATOM')) }}</updated>
            <summary type="xhtml">
                <div xmlns="http://www.w3.org/1999/xhtml">
                    {% if entity.logo %}
                        <div>
                            <a href="{{ entity.url }}">
                                <img src="http://{{ app.request.host }}/uploads/jobs/{{ entity.logo }}" alt="{{ entity.company }} logo" />
                            </a>
                        </div>
                    {% endif %}
                    <div>
                        {{ entity.description|nl2br }}
                    </div>
                    <h4>How to apply?</h4>
                    <p>{{ entity.howtoapply }}</p>
                </div>
            </summary>
            <author><name>{{ entity.company }}</name></author>
        </entry>
    {% endfor %}
</feed>
```

# 许可证 #

如果您需要转载的话，请尊重原作者的知识产权，您可以通过把如下链接放到您转载文章中的头部或者尾部，谢谢。

原文链接：<http://www.intelligentbee.com/blog/2013/08/20/symfony2-jobeet-day-14-feeds/>

您可以在以下链接查看该许可证的全文：

![](../imgs/license.png)

<http://creativecommons.org/licenses/by-nc/3.0/legalcode>
