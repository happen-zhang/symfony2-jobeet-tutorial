# 第六天：更多的数据模型 #

*这一系列文章来源于Fabien Potencier，基于Symfony1.4编写的[Jobeet Tutirual](http://symfony.com/legacy/doc/jobeet?orm=Doctrine)。

## Doctrine查询对象 ##

在第二天中我们有这样的一个需求（requirements）：“在*job*首页显示最新和有效的（active）*job*列表”。我们现在的首页显示的是全部的*job*，而没有考虑*job*是否有效。

```PHP
// src/Ibw/JobeetBundle/Controller/JobController.php
// ...
 
class JobController extends Controller
{
    public function indexAction()
    {
        $em = $this->getDoctrine()->getManager();
 
        $entities = $em->getRepository('IbwJobeetBundle:Job')->findAll();
 
        return $this->render('IbwJobeetBundle:Job:index.html.twig', array(
            'entities' => $entities
        ));
 
 // ...
}
```

一个有效的*job*意味着这个*job*发布的日期不超过30天。`$entities = $em->getRepository('IbwJobeetBundle')->findAll()`这行代码是从数据库中取出所有的*job*数据，因为我们没有指定任何的查询条件。

我们来做些修改，使它只取出有效的（active）的*job*：

```PHP
// src/Ibw/JobeetBundle/Controller/JobController.php
public function indexAction()
{
    $em = $this->getDoctrine()->getManager();
 
    $query = $em->createQuery(
        'SELECT j FROM IbwJobeetBundle:Job j WHERE j.created_at > :date'
    )->setParameter('date', date('Y-m-d H:i:s', time() - 86400 * 30));
    $entities = $query->getResult();
 
    return $this->render('IbwJobeetBundle:Job:index.html.twig', array(
        'entities' => $entities
    ));
}
```

## 调试Doctrine生成的SQL ##

有时候看看*Doctrine*生成出来的SQL对我们还是很有帮助的，比如*Doctrine*的查询得不到我们期望结果的时候。在开发环境（development）下，多亏有*Symfony*的调试工具栏（浏览器页面的最下面的那条栏），对我们开发过程中有用的信息都能在那里找到，包括刚才说的*SQL*调试（<http://jobeet.local/app_dev.php>）。

![](imgs/06-01.png)

## 序列化对象 ##

尽管上面的代码可以运行了，但这离完美还差很远呢，因为我们还没有考虑到另一个需求：“一个用户能够重新激活或者延续*job*的期限多30天”。

我们上面的代码中只依赖了*created_at*的值来取出有效的*job*，由于*created_at*表示的是*job*的创建时间，如果还需要表示*job*的过期时间，我们还要另外一些列（columns）。

如果你还记得我们在[第三天](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-03/chapter-03.md)中描述的数据表结构的话，你会记起我们还定义了一个*expires_at*列。由于我们还没在*fixture*文件中设置*expires_at*的值，它们都是空的。当一个*job*被创建的时候，*expires_at*的值就会被设置成*created_at*值的30天之后。

当你需要在*Doctrine*对象被序列化到数据库中之前自动执行一些操作的时候，你可以添加一个新的动作（*action*）到*ORM*映射文件的*lifecycle callback*区块中，就像我们之前对*created_at*列进行的操作：

```YAML
# src/Ibw/JobeetBundle/Resources/config/doctrine/Job.orm.yml
# ...
    # ...
    lifecycleCallbacks:
        prePersist: [ setCreatedAtValue, setExpiresAtValue ]
        preUpdate: [ setUpdatedAtValue ]
```

现在我们需要重新生成实体类（entity class），这样*Doctrine*就会在*Job*类中添加一个*setExpiresAtValue()*函数：

    php app/console doctrine:generate:entities IbwJobeetBundle

打开*src/Ibw/JobeetBundle/Entity/Job.php*文件，我们来编辑*setExpiresAtValue()*函数：

```PHP
// src/Ibw/JobeetBundle/Entity/Job.php
// ...
 
class Job
{
    // ... 
 
    public function setExpiresAtValue()
    {
        if(!$this->getExpiresAt()) {
            $now = $this->getCreatedAt() ? $this->getCreatedAt()->format('U') : time();
            $this->expires_at = new \DateTime(date('Y-m-d H:i:s', $now + 86400 * 30));
        }
    }
}
```

好，现在让我们来用*expires_at*来替换*created_at*来取出有效的*job*：

```PHP
// src/Ibw/JobeetBundle/Controller/JobController.php
// ...
 
    public function indexAction()
    {
        $em = $this->getDoctrine()->getManager();
 
        $query = $em->createQuery(
            'SELECT j FROM IbwJobeetBundle:Job j WHERE j.expires_at > :date'
    )->setParameter('date', date('Y-m-d H:i:s', time()));
        $entities = $query->getResult();
 
        return $this->render('IbwJobeetBundle:Job:index.html.twig', array(
            'entities' => $entities
        ));
    }
 
// ...
```

## 加入更多的Fixtures ##

现在刷新浏览器中的*job*首页，你不会看到有任何的改变，因为我们之前加入到数据库中的*job*数据都才仅仅发布了几天，数据库中的*job*都是有效的。让我们在*fixture*中加入一些过期的（*expired*）的*job*数据吧：

```PHP
// src/Ibw/JobeetBundle/DataFixtures/ORM/LoadJobData.php
// ...
 
    public function load(ObjectManager $em)
    {
        $job_expired = new Job();
        $job_expired->setCategory($em->merge($this->getReference('category-programming')));
        $job_expired->setType('full-time');
        $job_expired->setCompany('Sensio Labs');
        $job_expired->setLogo('sensio-labs.gif');
        $job_expired->setUrl('http://www.sensiolabs.com/');
        $job_expired->setPosition('Web Developer Expired');
        $job_expired->setLocation('Paris, France');
        $job_expired->setDescription('Lorem ipsum dolor sit amet, consectetur adipisicing elit.');
        $job_expired->setHowToApply('Send your resume to lorem.ipsum [at] dolor.sit');
        $job_expired->setIsPublic(true);
        $job_expired->setIsActivated(true);
        $job_expired->setToken('job_expired');
        $job_expired->setEmail('job@example.com');
        $job_expired->setCreatedAt(new \DateTime('2005-12-01'));
 
        // ...
 
        $em->persist($job_expired);
        // ...
    }
 
// ...
```

现在重新加载*fixtures*，然后刷新你的浏览器，确保过期的*job*不被显示出来：

    php app/console doctrine:fixtures:load

## 重构代码 ##

尽管上面的代码已经能运行了，但还是不够好，你能发现其中有什么问题吗？

*Doctrine*的查询代码不应该属于*action*（*Controller*层），它应该属于*Model*层。在*MVC*模式中，*Model*定义的是业务逻辑，*Controller*则通过*Model*从数据库中取出数据。我们来把之前从数据库中获取*job*数据集合的代码从*Controller*层移到*Model*层吧。现在我们为*Job*实体类创建一个*repository*类。

打开*/src/Ibw/JobeetBundle/Resources/config/doctrine/Job.orm.yml*，添加下面的代码：

```YAML
Ibw\JobeetBundle\Entity\Job:
    type: entity
    repositoryClass: Ibw\JobeetBundle\Repository\JobRepository
    # ...
```

使用下面的命令，*Doctrine*能够帮你生成*repository*类：

    php app/console doctrine:generate:entities IbwJobeetBundle

下一步我们来给*JobRepository*类添加一个方法：*getActiveJobs()*。这个方法可以取出有效的*job*，并且按照*expires_at*的值排序（它还可以接受一个*$category_id*参数，它可以按分类取出*job*）。

```PHP
// src/Ibw/JobeetBundle/Repository/JobRepository.php
<?php

namespace Ibw\JobeetBundle\Repository;

use Doctrine\ORM\EntityRepository;

/**
 * JobRepository
 *
 * This class was generated by the Doctrine ORM. Add your own custom
 * repository methods below.
 */
class JobRepository extends EntityRepository
{
    public function getActiveJobs($category_id = null)
    {
        $qb = $this->createQueryBuilder('j')
                   ->where('j.expires_at > :date')
                   ->setParameter('date', date('Y-m-d H:i:s', time()))
                   ->orderBy('j.expires_at', 'DESC');

        if ($category_id) {
            $qb->andWhere('j.category = :category_id')
               ->setParameter('category_id', $category_id);
        }

        $query = $qb->getQuery();

        return $query->getResult();
    }
}
```

现在*action*里面就可以使用刚才添加的*getActiveJobs()*方法了。

```PHP
// src/Ibw/JobeetBundle/Controller/JobController.php
// ...
 
    public function indexAction()
    {
        $em = $this->getDoctrine()->getManager();
 
        $entities = $em->getRepository('IbwJobeetBundle:Job')->getActiveJobs();
 
        return $this->render('IbwJobeetBundle:Job:index.html.twig', array(
            'entities' => $entities
        ));
    }
 
// ...
```

上面重构后的代码比起之前的未重构代码有如下优点：

* 获取有效*job*的代码现在位于*Model*层
* *JobController::indexAction()*中代码更少了，可读性更高
* *getActiveJobs()*方法可以被重用
* 更加容易地对*model*的代码进行测试

## 首页中的Categories ##

根据我们第二天中的需求，*job*需要按照分类进行显示。直到现在我们都还没考虑到对*job*进行分类显示。按照第二天中的需求，我们在需要在首页中按分类来显示*job*。首先，我们需要获得包含有效*job*的所有分类。

为*Category*实体创建一个*repository*类：

```YAML
# src/Ibw/JobeetBundle/Resources/config/doctrine/Category.orm.yml
Ibw\JobeetBundle\Entity\Category:
    type: entity
    repositoryClass: Ibw\JobeetBundle\Repository\CategoryRepository
    #...
```

生成*repository*类：

    php app/console doctrine:generate:entities IbwJobeetBundle

打开*src/Ibw/JobeetBundle/Repository/CategoryRepository.php*文件，添加*getWithJobs()*方法：

```PHP
// src/Ibw/JobeetBundle/Repository/CategoryRepository.php
namespace Ibw\JobeetBundle\Repository;
 
use Doctrine\ORM\EntityRepository;
 
/**
 * CategoryRepository
 *
 * This class was generated by the Doctrine ORM. Add your own custom
 * repository methods below.
 */
class CategoryRepository extends EntityRepository
{
    public function getWithJobs()
    {
        $query = $this->getEntityManager()->createQuery(
            'SELECT c FROM IbwJobeetBundle:Category c LEFT JOIN c.jobs j WHERE j.expires_at > :date'
        )->setParameter('date', date('Y-m-d H:i:s', time()));
 
        return $query->getResult();
    }   
}
```

同时修改*indexAction()*：

```PHP
// src/Ibw/JobeetBundle/Controller/JobController.php
// ...
 
    public function indexAction()
    {
        $em = $this->getDoctrine()->getManager();
 
        $categories = $em->getRepository('IbwJobeetBundle:Category')->getWithJobs();
 
        foreach($categories as $category) {
            $category->setActiveJobs($em->getRepository('IbwJobeetBundle:Job')->getActiveJobs($category->getId()));
        }
 
        return $this->render('IbwJobeetBundle:Job:index.html.twig', array(
            'categories' => $categories
        ));
    }
 
// ...
```

我们在上面的代码可以看到*Category*有*setActiveJobs()*方法，那我们来修改这个方法：

```PHP
// src/Ibw/JobeetBundle/Entity/Category.php
class Category
{
    // ...
 
    private $active_jobs;
 
    // ...
 
    public function setActiveJobs($jobs)
    {
        $this->active_jobs = $jobs;
    }
 
    public function getActiveJobs()
    {
        return $this->active_jobs;
    }
}
```

在模板中，我们需要通过迭代*categories*来显示所有的*jobs*：

```HTML
<!-- src/Ibw/JobeetBundle/Resources/views/Job/index.html.twig -->
<!-- ... -->
{% block content %}
    <div id="jobs">
        {% for category in categories %}
            <div>
                <div class="category">
                    <div class="feed">
                        <a href="">Feed</a>
                    </div>
                    <h1>{{ category.name }}</h1>
                </div>
                <table class="jobs">
                    {% for entity in category.activejobs %}
                        <tr class="{{ cycle(['even', 'odd'], loop.index) }}">
                            <td class="location">{{ entity.location }}</td>
                            <td class="position">
                                <a href="{{ path('ibw_job_show', { 'id': entity.id, 'company': entity.companyslug, 'location': entity.locationslug, 'position': entity.positionslug }) }}">
                                    {{ entity.position }}
                                </a>
                            </td>
                             <td class="company">{{ entity.company }}</td>
                        </tr>
                    {% endfor %}
                </table>
            </div>
        {% endfor %}
    </div>
{% endblock %}
```

## 限制结果行数 ##

我们现在需要实现限制*job*列表中的数目为10行。这个非常简单，我们来给*JobRepository::getActiveJobs()*方法添加一个*$max*参数：

```PHP
// src/Ibw/JobeetBundle/Repository/JobRepository.php
public function getActiveJobs($category_id = null, $max = null)
{
    $qb = $this->createQueryBuilder('j')
        ->where('j.expires_at > :date')
        ->setParameter('date', date('Y-m-d H:i:s', time()))
        ->orderBy('j.expires_at', 'DESC');

    if($max) {
        $qb->setMaxResults($max);
    }

    if($category_id) {
        $qb->andWhere('j.category = :category_id')
            ->setParameter('category_id', $category_id);
    }

    $query = $qb->getQuery();

    return $query->getResult();
}
```

修改*indexAction()*中，带*$max*参数调用*getActiveJobs()*方法：

```PHP
// src/Ibw/JobeetBundle/Controller/JobController.php
// ...
 
public function indexAction()
{
    $em = $this->getDoctrine()->getManager();

    $categories = $em->getRepository('IbwJobeetBundle:Category')->getWithJobs();

    foreach($categories as $category)
    {
        $category->setActiveJobs($em->getRepository('IbwJobeetBundle:Job')->getActiveJobs($category->getId(), 10));
    }

    return $this->render('IbwJobeetBundle:Job:index.html.twig', array(
        'categories' => $categories
    ));
}
 
// ...
```

## 自定义配置 ##

在*JobController::indexAction()*方法中，我们对返回*job*的行数（*$max = 10*）进行了*硬编码*（*hardcode*），我们来让返回行数可配置。在Symfony中，你可以在*app/config/config.yml*文件中的*parameters*区块中自定义一些配置（如果*parameters*区块不存在，那么也可以创建它）。

```YAML
# app/config/config.yml
# ...
 
parameters:
    max_jobs_on_homepage: 10
```

定义好后我们就可以在*controller*中访问它们的值：

```PHP
// src/Ibw/JobeetBundle/Controller/JobController.php
// ...
 
    public function indexAction()
    {
        $em = $this->getDoctrine()->getManager();
 
        $categories = $em->getRepository('IbwJobeetBundle:Category')->getWithJobs();
 
        foreach($categories as $category) {
            $category->setActiveJobs($em->getRepository('IbwJobeetBundle:Job')->getActiveJobs($category->getId(), $this->container->getParameter('max_jobs_on_homepage')));
        }
 
        return $this->render('IbwJobeetBundle:Job:index.html.twig', array(
            'categories' => $categories
        ));
    }
 
// ...
```

## 动态Fixtures ##

对于上面的修改，我们还不能在首页中看到有什么变化，毕竟我们的数据中的只有很少量的*job*数据。现在我们需要在*fixture*中批量添加*job*数据。你可选择手动复制之前存在的代码来重复进行生成*job*，但我们有更好的办法。重复的代码让人感觉很差，甚至是*fixture*文件：

```PHP
// src/Ibw/JobeetBundle/DataFixtures/ORM/LoadJobData.php
// ...
 
public function load(ObjectManager $em)
{
    // ...
 
    for($i = 100; $i <= 130; $i++)
    {
        $job = new Job();
        $job->setCategory($em->merge($this->getReference('category-programming')));
        $job->setType('full-time');
        $job->setCompany('Company '.$i);
        $job->setPosition('Web Developer');
        $job->setLocation('Paris, France');
        $job->setDescription('Lorem ipsum dolor sit amet, consectetur adipisicing elit.');
        $job->setHowToApply('Send your resume to lorem.ipsum [at] dolor.sit');
        $job->setIsPublic(true);
        $job->setIsActivated(true);
        $job->setToken('job_'.$i);
        $job->setEmail('job@example.com');
 
        $em->persist($job);
    }
 
    // ... 
    $em->flush();
}
 
// ...
```

现在用`doctrine:fixtures:load`重新加载*fixture*，观察*Programming*分类下的*job*行数是否为10行：

![](imgs/06-02.png)

## 过期的Job页面 ##

如果一个*job*过期了，那么它将不再可以被访问，即使你知道URL。尝试着访问过期*job*的页面（获得过期*job*的id：`select id, token from job where expires_at < NOW()`，然后把获得的id替换下面URL中ID的值）:

    /app_dev.php/job/sensio-labs/paris-france/ID/web-developer-expired

访问过期的*job*，我们应该让用户重定向到*404页面*。我们来给*JobRepository*添加一个方法：

```PHP
// src/Ibw/JobeetBundle/Repository/JobRepository.php
// ...
 
public function getActiveJob($id)
{
    $query = $this->createQueryBuilder('j')
        ->where('j.id = :id')
        ->setParameter('id', $id)
        ->andWhere('j.expires_at > :date')
        ->setParameter('date', date('Y-m-d H:i:s', time()))
        ->setMaxResults(1)
        ->getQuery();

    try {
        $job = $query->getSingleResult();
    } catch (\Doctrine\Orm\NoResultException $e) {
        $job = null;
    }

    return $job;
}
```

> 如果没有结果被返回，*getSingleResult()*方法会抛出*Doctrine\ORM\NoResultException*异常；如果返回的结果不止一个，*getSingleResult()*方法会抛出*Doctrine\ORM\NonUniqueResultException*异常。如果你使用*getSingleResult()*方法，请确保用*try...catch*包含它，以至于返回的结果集只有一行数据。

现在修改*JobController::showAction()*方法：

```PHP
// src/Ibw/JobeetBundle/Controller/JobController.php
// ...
 
$entity = $em->getRepository('IbwJobeetBundle:Job')->getActiveJob($id);
 
// ...
```

现在你去访问一个过时的*job*页面，你会被转到*404页面*：

![](imgs/06-03.png)

好了，今天就到这吧。我们明天会开始实现*category*页面。

# 许可证 #

如果您需要转载的话，请尊重原作者的知识产权，您可以通过把如下链接放到您转载文章中的头部或者尾部，谢谢。

原文链接：<http://www.intelligentbee.com/blog/2013/08/12/symfony2-jobeet-day-6-more-with-the-model/>

您可以在以下链接查看该许可证的全文：

![](../imgs/license.png)

<http://creativecommons.org/licenses/by-nc/3.0/legalcode>