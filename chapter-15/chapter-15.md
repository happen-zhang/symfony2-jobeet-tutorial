# 第十五天：Web Services #

*这一系列文章来源于Fabien Potencier，基于Symfony1.4编写的[Jobeet Tutirual](http://symfony.com/legacy/doc/jobeet?orm=Doctrine)。

在昨天的内容中，我们给*Jobeet*加上了订阅功能之后，用户就可以实时地接收到最新发布的信息了。

现在我们试着站在发布者的角度来思考，当发布者发布一个*Job*信息之后，发布者想要让尽可能多的人能够了解到这条信息。如果我们能把这些信息放在很多小网站上，这样一来看的人越多，那么发布者就将有更大几率能够找到适合这个工作的人选了。这就是所谓的[长尾效应（long tail）](http://en.wikipedia.org/wiki/The_Long_Tail)。*Affiliates*能够在他们的网站上发布最新的*Job*信息，这些都需要*web services*的支持，那么我们今天就来实现*web services*。

## Affiliates ##

就像我们在第二天中内容说的那样，一个*Affiliate*能够得到所有当前已激活的*Job*列表。

### The fixtures ###

我们为*Affiliates*创建一个新的*Fixture*文件：

```PHP
// src/Ibw/JobeetBundle/DataFixtures/ORM/LoadAffiliateData.php
namespace Ibw\JobeetBundle\DataFixtures\ORM;
 
use Doctrine\Common\Persistence\ObjectManager;
use Doctrine\Common\DataFixtures\AbstractFixture;
use Doctrine\Common\DataFixtures\OrderedFixtureInterface;
use Ibw\JobeetBundle\Entity\Affiliate;
 
class LoadAffiliateData extends AbstractFixture implements OrderedFixtureInterface
{
    public function load(ObjectManager $em)
    {
        $affiliate = new Affiliate();
 
        $affiliate->setUrl('http://sensio-labs.com/');
        $affiliate->setEmail('address1@example.com');
        $affiliate->setToken('sensio-labs');
        $affiliate->setIsActive(true);
        $affiliate->addCategorie($em->merge($this->getReference('category-programming')));
 
        $em->persist($affiliate);
 
        $affiliate = new Affiliate();
 
        $affiliate->setUrl('/');
        $affiliate->setEmail('address2@example.org');
        $affiliate->setToken('symfony');
        $affiliate->setIsActive(false);
        $affiliate->addCategorie($em->merge($this->getReference('category-programming')), $em->merge($this->getReference('category-design')));
 
        $em->persist($affiliate);
        $em->flush();
 
        $this->addReference('affiliate', $affiliate);
    }
 
    public function getOrder()
    {
        return 3; // This represents the order in which fixtures will be loaded
    }
}
```

现在运行下面的命令，我们把定义在*Fixture*文件中的数据持久化到数据库中：

    php app/console doctrine:fixtures:load

在*Fixture*文件中，我们可以看到*token*是硬编码写上去的，这里的目的是为了方便测试。但当在实际的运行时，当用户需要申请为*Affiliate*时，这个*token*将会被自动生成。我们在*Affiliate*类中创建一个方法来生成*token*。我们先在*ORM*文件中的*lifecycleCallbacks*部分添加*setTokenValue*方法：

```YAML
# src/Ibw/JobeetBundle/Resources/config/doctrine/Affiliate.orm.yml
# ... 
    lifecycleCallbacks:
        prePersist: [ setCreatedAtValue, setTokenValue ]
```

运行下面的命令会后，*Affiliate*类中将会生成*setTokenValue()*方法：

    php app/console doctrine:generate:entities IbwJobeetBundle

现在我们来修改这个方法：

```PHP
// src/Ibw/JobeetBundle/Entity/Affiliate.php
public function setTokenValue()
{
    if(!$this->getToken()) {
        $token = sha1($this->getEmail().rand(11111, 99999));
        $this->token = $token;
    }

    return $this;
}
```

重新加载数据：

    php app/console doctrine:fixtures:load

## The Job Web Service ##

和以前一样，我们每次创建新资源的的时候，第一件要做的事情就是定义路由，这是个好习惯：

```YAML
# src/Ibw/JobeetBundle/Resources/config/routing.yml
IbwJobeetBundle_api:
    pattern: /api/{token}/jobs.{_format}
    defaults: {_controller: "IbwJobeetBundle:Api:list"}
    requirements:
        _format: xml|json|yaml
```

通常我们修改过路由文件之后，我们都需要去清除*cache*：

    php app/console cache:clear --env=dev
    php app/console cache:clear --env=prod

我们的下一步是创建*api*动作和相应的模板，它们会共享相同的动作。我们新建一个控制器文件，并把它命名为“*ApiController*”：

```PHP
// src/Ibw/JobeetBundle/Controller/ApiController.php
namespace Ibw\JobeetBundle\Controller;
 
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Ibw\JobeetBundle\Entity\Affiliate;
use Ibw\JobeetBundle\Entity\Job;
use Ibw\JobeetBundle\Repository\AffiliateRepository;
 
class ApiController extends Controller
{
    public function listAction(Request $request, $token)
    {
        $em = $this->getDoctrine()->getManager();
 
        $jobs = array();
 
        $rep = $em->getRepository('IbwJobeetBundle:Affiliate');
        $affiliate = $rep->getForToken($token);
 
        if(!$affiliate) { 
            throw $this->createNotFoundException('This affiliate account does not exist!');
        }
 
        $rep = $em->getRepository('IbwJobeetBundle:Job');
        $active_jobs = $rep->getActiveJobs(null, null, null, $affiliate->getId());
 
        foreach ($active_jobs as $job) {
            $jobs[$this->get('router')->generate('ibw_job_show', array('company' => $job->getCompanySlug(), 'location' => $job->getLocationSlug(), 'id' => $job->getId(), 'position' => $job->getPositionSlug()), true)] = $job->asArray($request->getHost());
        }
 
        $format = $request->getRequestFormat();
        $jsonData = json_encode($jobs);
 
        if ($format == "json") {
            $headers = array('Content-Type' => 'application/json'); 
            $response = new Response($jsonData, 200, $headers);
 
            return $response;
        }
 
        return $this->render('IbwJobeetBundle:Api:jobs.' . $format . '.twig', array('jobs' => $jobs));  
    }
}
```

为了能够通过*token*获得*Affiliate*的信息，我们需要创建*getForToken()*方法。这个方法同样会验证*Affiliate*是否是已激活的，所以我们这里就不需要再判断*Affiliate*是否已被激活。到现在为止我们还尚未使用过*AffiliateRepository*，因为它还不存在。我们现在就来创建它，先修改*ORM*文件：

```YAML
# src/Ibw/JobeetBundle/Resources/config/doctrine/Affiliate.orm.yml
Ibw\JobeetBundle\Entity\Affiliate:
    type: entity
    repositoryClass: Ibw\JobeetBundle\Repository\AffiliateRepository
    # ...
```

运行下面的代码：

    php app/console doctrine:generate:entities IbwJobeetBundle

创建完毕后，我们马上就可以使用它了：

```PHP
// src/Ibw/JobeetBundle/Repository/AffiliateRepository.php
namespace Ibw\JobeetBundle\Repository;
 
use Doctrine\ORM\EntityRepository;
 
/**
 * AffiliateRepository
 *
 * This class was generated by the Doctrine ORM. Add your own custom
 * repository methods below.
 */
class AffiliateRepository extends EntityRepository
{
    public function getForToken($token)
    {
        $qb = $this->createQueryBuilder('a')
            ->where('a.is_active = :active')
            ->setParameter('active', 1)
            ->andWhere('a.token = :token')
            ->setParameter('token', $token)
            ->setMaxResults(1)
        ;
 
        try{
            $affiliate = $qb->getQuery()->getSingleResult();
        } catch(\Doctrine\Orm\NoResultException $e){
            $affiliate = null;
        }
 
        return $affiliate;
    }
}
```

当通过*token*取出对应的*Affiliate*之后，我们会调用*getActiveJobs()*方法返回属于某个分类的所有的*Job*信息给*Affiliate*。如果你打开*JobRepository.php*文件，可以看到*getActiveJobs()*方法没有提供任何的参数选项给*Affiliate*使用。为了能够重用这个方法，我们需要对它做一些修改：

```PHP
// src/Ibw/JobeetBundle/Repository/JobRepository.php
// ...
 
    public function getActiveJobs($category_id = null, $max = null, $offset = null, $affiliate_id = null)
    {
        $qb = $this->createQueryBuilder('j')
            ->where('j.expires_at > :date')
            ->setParameter('date', date('Y-m-d H:i:s', time()))
            ->andWhere('j.is_activated = :activated')
            ->setParameter('activated', 1)
            ->orderBy('j.expires_at', 'DESC');
 
        if($max) {
            $qb->setMaxResults($max);
        }
 
        if($offset) {
            $qb->setFirstResult($offset);
        }
 
        if($category_id) {
            $qb->andWhere('j.category = :category_id')
                ->setParameter('category_id', $category_id);
        }
        // j.category c, c.affiliate a
        if($affiliate_id) {
            $qb->leftJoin('j.category', 'c')
               ->leftJoin('c.affiliates', 'a')
               ->andWhere('a.id = :affiliate_id')
               ->setParameter('affiliate_id', $affiliate_id)
            ;
        }
 
        $query = $qb->getQuery();
 
        return $query->getResult();
    }
 
// ...
```

就如你所看到的，我们调用了*asArray()*函数来填充*jobs*数组。我们来定义它：

```PHP
// src/Ibw/JobeetBundle/Entity/Job.php
public function asArray($host)
{
    return array(
        'category'     => $this->getCategory()->getName(),
        'type'         => $this->getType(),
        'company'      => $this->getCompany(),
        'logo'         => $this->getLogo() ? 'http://' . $host . '/uploads/jobs/' . $this->getLogo() : null,
        'url'          => $this->getUrl(),
        'position'     => $this->getPosition(),
        'location'     => $this->getLocation(),
        'description'  => $this->getDescription(),
        'how_to_apply' => $this->getHowToApply(),
        'expires_at'   => $this->getCreatedAt()->format('Y-m-d H:i:s'),
    );
}
```

### *XML*格式 ###

支持*XML*格式很简单，只需要创建一个模板即可：

```XML
<!-- src/Ibw/JobeetBundle/Resources/views/Api/Jobs.xml.twig -->
<?xml version="1.0" encoding="utf-8"?>
<jobs>
{% for url, job in jobs %}
    <job url="{{ url }}">
{% for key,value in job %}
        <{{ key }}>{{ value }}</{{ key }}>
{% endfor %}
    </job>
{% endfor %}
</jobs>
```

### *JSON*格式 ###

支持*JSON*格式也是类似的：

```JSON
// src/Ibw/JobeetBundle/Resources/views/Api/jobs.json.twig
{% for url, job in jobs %}
{% i = 0, count(jobs), ++i %}
[
    "url":"{{ url }}",
{% for key, value in job %} {% j = 0, count(key), ++j %}
    "{{ key }}":"{% if j == count(key)%} {{ json_encode(value) }}, {% else %} {{ json_encode(value) }}
                 {% endif %}"
{% endfor %}]
{% endfor %}
```

### *YAML*格式 ###

```YAML
# src/Ibw/JobeetBundle/Resources/views/Api/jobs.yaml.twig
{% for url,job in jobs %}
    Url: {{ url }}
{% for key, value in job %}
        {{ key }}: {{ value }}
{% endfor %}
{% endfor %}
```

如果你想通过一个无效的*token*来调用*web service*，那么不论你请求的是什么格式的内容，你都将会得到一个**404**响应。如果你想看到我们努力的成果，你可以试试访问下面的链接：<http://jobeet.local/app_dev.php/api/sensio-labs/jobs.xml>或者是<http://jobeet.local/app_dev.php/api/symfony/jobs.xml>。你可以修改*URL*中的扩展名来得到指定格式的内容。

## 测试*Web Service* ##

```PHP
// src/Ibw/JobeetBundle/Tests/Controller/ApiControllerTest.php
namespace Ibw\JobeetBundle\Tests\Controller;
 
use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;
use Symfony\Bundle\FrameworkBundle\Console\Application;
use Symfony\Component\Console\Output\NullOutput;
use Symfony\Component\Console\Input\ArrayInput;
use Doctrine\Bundle\DoctrineBundle\Command\DropDatabaseDoctrineCommand;
use Doctrine\Bundle\DoctrineBundle\Command\CreateDatabaseDoctrineCommand;
use Doctrine\Bundle\DoctrineBundle\Command\Proxy\CreateSchemaDoctrineCommand;
use Symfony\Component\DomCrawler\Crawler;
use Symfony\Component\HttpFoundation\HttpExceptionInterface;
 
class ApiControllerTest extends WebTestCase
{
    private $em;
 
    private $application;
 
    public function setUp()
    {
        static::$kernel = static::createKernel();
        static::$kernel->boot();
 
        $this->application = new Application(static::$kernel);
 
        // drop the database
        $command = new DropDatabaseDoctrineCommand();
        $this->application->add($command);
        $input = new ArrayInput(array(
            'command' => 'doctrine:database:drop',
            '--force' => true
        ));
        $command->run($input, new NullOutput());
 
        // we have to close the connection after dropping the database so we don't get "No database selected" error
        $connection = $this->application->getKernel()->getContainer()->get('doctrine')->getConnection();
        if ($connection->isConnected()) {
            $connection->close();
        }
 
        // create the database
        $command = new CreateDatabaseDoctrineCommand();
        $this->application->add($command);
        $input = new ArrayInput(array(
            'command' => 'doctrine:database:create',
        ));
        $command->run($input, new NullOutput());
 
        // create schema
        $command = new CreateSchemaDoctrineCommand();
        $this->application->add($command);
        $input = new ArrayInput(array(
            'command' => 'doctrine:schema:create',
        ));
        $command->run($input, new NullOutput());
 
        // get the Entity Manager
        $this->em = static::$kernel->getContainer()
            ->get('doctrine')
            ->getManager();
 
        // load fixtures
        $client = static::createClient();
        $loader = new \Symfony\Bridge\Doctrine\DataFixtures\ContainerAwareLoader($client->getContainer());
        $loader->loadFromDirectory(static::$kernel->locateResource('@IbwJobeetBundle/DataFixtures/ORM'));
        $purger = new \Doctrine\Common\DataFixtures\Purger\ORMPurger($this->em);
        $executor = new \Doctrine\Common\DataFixtures\Executor\ORMExecutor($this->em, $purger);
        $executor->execute($loader->getFixtures());
    }
 
    public function testList()
    {
        $client = static::createClient();
        $crawler = $client->request('GET', '/api/sensio-labs/jobs.xml');
 
        $this->assertEquals('Ibw\JobeetBundle\Controller\ApiController::listAction', $client->getRequest()->attributes->get('_controller'));
        $this->assertTrue($crawler->filter('description')->count() == 32);
 
        $crawler = $client->request('GET', '/api/sensio-labs87/jobs.xml');
 
        $this->assertTrue(404 === $client->getResponse()->getStatusCode());
 
        $crawler = $client->request('GET', '/api/symfony/jobs.xml');
 
        $this->assertTrue(404 === $client->getResponse()->getStatusCode());
 
        $crawler = $client->request('GET', '/api/sensio-labs/jobs.json');
 
        $this->assertEquals('Ibw\JobeetBundle\Controller\ApiController::listAction', $client->getRequest()->attributes->get('_controller'));
        $this->assertRegExp('/"category"\:"Programming"/', $client->getResponse()->getContent());
 
        $crawler = $client->request('GET', '/api/sensio-labs87/jobs.json');
 
        $this->assertTrue(404 === $client->getResponse()->getStatusCode());
 
        $crawler = $client->request('GET', '/api/sensio-labs/jobs.yaml');
        $this->assertRegExp('/category\: Programming/', $client->getResponse()->getContent());
 
        $this->assertEquals('Ibw\JobeetBundle\Controller\ApiController::listAction', $client->getRequest()->attributes->get('_controller'));
 
        $crawler = $client->request('GET', '/api/sensio-labs87/jobs.yaml');
 
        $this->assertTrue(404 === $client->getResponse()->getStatusCode());
    }
}
```

## *Affiliate*申请表单 ##

*web service*已经可以使用了，现在我们来添加创建*Affiliate*的表单吧。为了实现这个功能，我们需要写HTML表单，为每个表单域实现验证规则，把表单域的值处理后保存到数据库中，当表单数据有错误时还需要显示出错误信息反馈给用户。

首先我们先创建控制器文件，把它命名为*AffiliateCotrolelr*：

```PHP
// src/Ibw/JobeetBundle/Tests/Controller/AffiliateController.php
namespace Ibw\JobeetBundle\Controller;
 
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Ibw\JobeetBundle\Entity\Affiliate;
use Ibw\JobeetBundle\Form\AffiliateType;
use Symfony\Component\HttpFoundation\Request;
use Ibw\JobeetBundle\Entity\Category;
 
class AffiliateController extends Controller
{
    // Your code goes here
}
```

然后修改*layout.html.twig*中的链接：

```HTML
<!-- src/Ibw/JobeetBundle/Resources/views/layout.html.twig -->
<!-- ... -->
    <li class="last"><a href="{{ path('ibw_affiliate_new') }}">Become an affiliate</a></li>
<!-- ... -->
```

现在创建一个*action*来匹配刚才修改链接的路由：

```PHP
// src/Ibw/JobeetBundle/Controller/AffiliateController.php
namespace Ibw\JobeetBundle\Controller;
 
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Ibw\JobeetBundle\Entity\Affiliate;
use Ibw\JobeetBundle\Form\AffiliateType;
use Symfony\Component\HttpFoundation\Request;
use Ibw\JobeetBundle\Entity\Category;
 
class AffiliateController extends Controller
{
    public function newAction()
    {
        $entity = new Affiliate();
        $form = $this->createForm(new AffiliateType(), $entity);
 
        return $this->render('IbwJobeetBundle:Affiliate:affiliate_new.html.twig', array(
            'entity' => $entity,
            'form'   => $form->createView(),
        ));
    }
}
```

我们已经有了路由的名字还有*action*，但我们还没有路由。所以我们来创建它：

```YAML
# src/Ibw/JobeetBundle/Resources/config/routing/affiliate.yml
ibw_affiliate_new:
    pattern:  /new
    defaults: { _controller: "IbwJobeetBundle:Affiliate:new" }
```

同样需要在*routing.yml*文件中加入下面的代码：

```YAML
# src/Ibw/JobeetBundle/Resources/config/routing.yml
# ...
 
IbwJobeetBundle_ibw_affiliate:
    resource: "@IbwJobeetBundle/Resources/config/routing/affiliate.yml"
    prefix:   /affiliate
```

表单类同样需要被创建出来。尽管*Affiliate*有很多的字段域，但我们没有必要把它们全部显示出来，因为有些字段域是不需要用来填写的。我们来创建*Affiliate*表单：

```PHP
// src/Ibw/JobeetBundle/Form/AffiliateType.php
namespace Ibw\JobeetBundle\Form;
 
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolverInterface;
use Ibw\JobeetBundle\Entity\Affiliate;
use Ibw\JobeetBundle\Entity\Category;
 
class AffiliateType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('url')
            ->add('email')
            ->add('categories', null, array('expanded'=>true))
        ;
    }
 
    public function setDefaultOptions(OptionsResolverInterface $resolver)
    {
        $resolver->setDefaults(array(
            'data_class' => 'Ibw\JobeetBundle\Entity\Affiliate',
        ));
    }
 
    public function getName()
    {
        return 'affiliate';
    }
}
```

现在我们需要验证提交上来的*Affiliate*表单对象中的数据是否有效。我们在*validation.yml*文件中添加下面的代码：

```YAML
# src/Ibw/JobeetBundle/Resources/config/validation.yml
# ...
 
Ibw\JobeetBundle\Entity\Affiliate:
    constraints:
        - Symfony\Bridge\Doctrine\Validator\Constraints\UniqueEntity: email
    properties:
        url:
            - Url: ~
        email:
            - NotBlank: ~
            - Email: ~
```

在上面的代码中，我们使用了一个新的验证器，叫做*UniqueEntity*。它能够验证*Doctrine*实体对象中的一个或者多个特殊字段域是否是唯一的。这个十分常用，例如，我们需要防止一个新注册用户的*email*地址和已存在用户的*email*地址重复。

添加了验证约束之后不要忘记清除*cache*！

最后一步需要创建表单试图：

```HTML
<!-- src/Ibw/JobeetBundle/Resources/views/Affiliate/affiliate_new.html.twig -->
{% extends 'IbwJobeetBundle::layout.html.twig' %}
 
{% set form_themes = _self %}
 
{% block form_errors %}
{% spaceless %}
    {% if errors|length > 0 %}
        <ul class="error_list">
            {% for error in errors %}
                <li>{{ error.messageTemplate|trans(error.messageParameters, 'validators') }}</li>
            {% endfor %}
        </ul>
    {% endif %}
{% endspaceless %}
{% endblock form_errors %}
 
{% block stylesheets %}
    {{ parent() }}
    <link rel="stylesheet" href="{{ asset('bundles/ibwjobeet/css/job.css') }}" type="text/css" media="all" />
{% endblock %}
 
{% block content %}
    <h1>Become an affiliate</h1>
        <form action="{{ path('ibw_affiliate_create') }}" method="post" {{ form_enctype(form) }}>
            <table id="job_form">
                <tfoot>
                    <tr>
                        <td colspan="2">
                            <input type="submit" value="Submit" />
                        </td>
                    </tr>
                </tfoot>
                <tbody>
                    <tr>
                        <th>{{ form_label(form.url) }}</th>
                        <td>
                            {{ form_errors(form.url) }}
                            {{ form_widget(form.url) }}
                        </td>
                    </tr>
                    <tr>
                        <th>{{ form_label(form.email) }}</th>
                        <td>
                            {{ form_errors(form.email) }}
                            {{ form_widget(form.email) }}
                        </td>
                    </tr>
                    <tr>
                        <th>{{ form_label(form.categories) }}</th>
                        <td>
                            {{ form_errors(form.categories) }}
                            {{ form_widget(form.categories) }}
                        </td>
                    </tr>
                </tbody>
            </table>
        {{ form_end(form) }}
{% endblock %}
```

当提交的表单数据如果是有效的，那么表单数据就会被保存到数据库中。我们为*AffiliateController*添加*create*动作：

```PHP
// src/Ibw/JobeetBundle/Controller/AffiliateController.php
class AffiliateController extends Controller 
{
    // ...    
 
    public function createAction(Request $request)
    {
        $affiliate = new Affiliate();
        $form = $this->createForm(new AffiliateType(), $affiliate);
        $form->bind($request);
        $em = $this->getDoctrine()->getManager();
 
        if ($form->isValid()) {
 
            $formData = $request->get('affiliate');
            $affiliate->setUrl($formData['url']);
            $affiliate->setEmail($formData['email']);
            $affiliate->setIsActive(false);
 
            $em->persist($affiliate);
            $em->flush();
 
            return $this->redirect($this->generateUrl('ibw_affiliate_wait'));
        }
 
        return $this->render('IbwJobeetBundle:Affiliate:affiliate_new.html.twig', array(
            'entity' => $affiliate,
            'form'   => $form->createView(),
        ));
    }
}
```

当表单一旦被提交，那么*createAction()*就会被执行，所以我们需要定义路由：

```YAML
# src/Ibw/JobeetBundle/Resources/config/routing/affiliate.yml
# ...
 
ibw_affiliate_create:
    pattern: /create
    defaults: { _controller: "IbwJobeetBundle:Affiliate:create" }
    requirements: { _method: post }
```

*Affiliate*注册之后，他将会被重定向到等待页面。我们来为定义这个动作或试图吧：

```PHP
// src/Ibw/JobeetBundle/Controller/AffiliateController.php
class AffiliateController extends Controller
{
    // ...
 
    public function waitAction()
    {
        return $this->render('IbwJobeetBundle:Affiliate:wait.html.twig');
    }
}
```

```HTML
<!-- src/Ibw/JobeetBundle/Resources/views/Affiliate/wait.html.twig -->
{% extends "IbwJobeetBundle::layout.html.twig" %}
 
{% block content %}
    <div class="content">
        <h1>Your affiliate account has been created</h1>
        <div style="padding: 20px">
            Thank you!
            You will receive an email with your affiliate token
            as soon as your account will be activated.
        </div>
    </div>
{% endblock %}
```

现在添加路由：

```YAML
# src/Ibw/JobeetBundle/Resources/config/routing/affiliate.yml
# ...
 
ibw_affiliate_wait:
    pattern: /wait
    defaults: { _controller: "IbwJobeetBundle:Affiliate:wait" }
```

定义好路由之后，我们需要清除*cache*。

现在你可以试着点击首页中的*Affiliates*链接，你会跳转到*Affiliate*表单页面。

### 测试 ###

最后一步是为新功能添加功能测试：

```PHP
// src/Ibw/JobeetBundle/Tests/Controller/AffiliateControllerTest.php
namespace Ibw\JobeetBundle\Tests\Controller;
 
use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;
use Symfony\Bundle\FrameworkBundle\Console\Application;
use Symfony\Component\Console\Output\NullOutput;
use Symfony\Component\Console\Input\ArrayInput;
use Doctrine\Bundle\DoctrineBundle\Command\DropDatabaseDoctrineCommand;
use Doctrine\Bundle\DoctrineBundle\Command\CreateDatabaseDoctrineCommand;
use Doctrine\Bundle\DoctrineBundle\Command\Proxy\CreateSchemaDoctrineCommand;
use Symfony\Component\DomCrawler\Crawler;
 
class AffiliateControllerTest extends WebTestCase
{
    private $em;
    private $application;
 
    public function setUp()
    {
        static::$kernel = static::createKernel();
        static::$kernel->boot();
 
        $this->application = new Application(static::$kernel);
 
        // drop the database
        $command = new DropDatabaseDoctrineCommand();
        $this->application->add($command);
        $input = new ArrayInput(array(
            'command' => 'doctrine:database:drop',
            '--force' => true
        ));
        $command->run($input, new NullOutput());
 
        // we have to close the connection after dropping the database so we don't get "No database selected" error
        $connection = $this->application->getKernel()->getContainer()->get('doctrine')->getConnection();
        if ($connection->isConnected()) {
            $connection->close();
        }
 
        // create the database
        $command = new CreateDatabaseDoctrineCommand();
        $this->application->add($command);
        $input = new ArrayInput(array(
            'command' => 'doctrine:database:create',
        ));
        $command->run($input, new NullOutput());
 
        // create schema
        $command = new CreateSchemaDoctrineCommand();
        $this->application->add($command);
        $input = new ArrayInput(array(
            'command' => 'doctrine:schema:create',
        ));
        $command->run($input, new NullOutput());
 
        // get the Entity Manager
        $this->em = static::$kernel->getContainer()
            ->get('doctrine')
            ->getManager();
 
        // load fixtures
        $client = static::createClient();
        $loader = new \Symfony\Bridge\Doctrine\DataFixtures\ContainerAwareLoader($client->getContainer());
        $loader->loadFromDirectory(static::$kernel->locateResource('@IbwJobeetBundle/DataFixtures/ORM'));
        $purger = new \Doctrine\Common\DataFixtures\Purger\ORMPurger($this->em);
        $executor = new \Doctrine\Common\DataFixtures\Executor\ORMExecutor($this->em, $purger);
        $executor->execute($loader->getFixtures());
    }
 
    public function testAffiliateForm()
    {
        $client = static::createClient();
        $crawler = $client->request('GET', '/affiliate/new');
 
        $this->assertEquals('Ibw\JobeetBundle\Controller\AffiliateController::newAction', $client->getRequest()->attributes->get('_controller'));
 
        $form = $crawler->selectButton('Submit')->form(array(
            'affiliate[url]' => 'http://sensio-labs.com/',
            'affiliate[email]' => 'jobeet@example.com'
        ));
 
        $client->submit($form);
        $this->assertEquals('Ibw\JobeetBundle\Controller\AffiliateController::createAction', $client->getRequest()->attributes->get('_controller'));
 
        $kernel = static::createKernel();
        $kernel->boot();
        $em = $kernel->getContainer()->get('doctrine.orm.entity_manager');
 
        $query = $em->createQuery('SELECT count(a.email) FROM IbwJobeetBundle:Affiliate a WHERE a.email = :email');
        $query->setParameter('email', 'jobeet@example.com');
        $this->assertEquals(1, $query->getSingleScalarResult());
 
        $crawler = $client->request('GET', '/affiliate/new');
        $form = $crawler->selectButton('Submit')->form(array(
            'affiliate[email]'        => 'not.an.email',
        ));
        $crawler = $client->submit($form);
 
        // check if we have 1 errors
        $this->assertTrue($crawler->filter('.error_list')->count() == 1);
        // check if we have error on affiliate_email field
        $this->assertTrue($crawler->filter('#affiliate_email')->siblings()->first()->filter('.error_list')->count() == 1);
    }
 
    public function testCreate()
    {
        $client = static::createClient();
        $crawler = $client->request('GET', '/affiliate/new');
        $form = $crawler->selectButton('Submit')->form(array(
            'affiliate[url]' => 'http://sensio-labs.com/',
            'affiliate[email]' => 'address@example.com'
        ));
 
        $client->submit($form);
        $client->followRedirect();
 
        $this->assertEquals('Ibw\JobeetBundle\Controller\AffiliateController::waitAction', $client->getRequest()->attributes->get('_controller'));
 
        return $client;
    }
 
    public function testWait()
    {
        $client = static::createClient();
        $crawler = $client->request('GET', '/affiliate/wait');
 
        $this->assertEquals('Ibw\JobeetBundle\Controller\AffiliateController::waitAction', $client->getRequest()->attributes->get('_controller'));
    }
}
```

## *Affiliate*后台管理 ##

对于后台，我们会使用*SonataAdminBundle*。就像我们之前说过的那样，*Affiliate*注册之后需要等待*admin*来激活他。所以，当*admin*访问*Affiliate*页面后，他只能看到激活或者不激活这两个操作，这样有利于提高工作效率嘛。

首先，你需要在*services.yml*中声明*Affiliate*服务：

```YAML
# src/Ibw/JobeetBundle/Resources/config/service.yml
# ...
    ibw.jobeet.admin.affiliate:
        class: Ibw\JobeetBundle\Admin\AffiliateAdmin
        tags:
            - { name: sonata.admin, manager_type: orm, group: jobeet, label: Affiliates }
        arguments:
            - ~
            - Ibw\JobeetBundle\Entity\Affiliate
            - 'IbwJobeetBundle:AffiliateAdmin'
```

然后创建*Admin*文件：

```PHP
// src/Ibw/JobeetBundle/Admin/AffiliateAdmin.php
namespace Ibw\JobeetBundle\Admin;
 
use Sonata\AdminBundle\Admin\Admin;
use Sonata\AdminBundle\Datagrid\ListMapper;
use Sonata\AdminBundle\Datagrid\DatagridMapper;
use Sonata\AdminBundle\Validator\ErrorElement;
use Sonata\AdminBundle\Form\FormMapper;
use Sonata\AdminBundle\Show\ShowMapper;
use Ibw\JobeetBundle\Entity\Affiliate;
 
class AffiliateAdmin extends Admin
{
    protected $datagridValues = array(
        '_sort_order' => 'ASC',
        '_sort_by' => 'is_active'
    );
 
    protected function configureFormFields(FormMapper $formMapper)
    {
        $formMapper
            ->add('email')
            ->add('url')
        ;
    }
 
    protected function configureDatagridFilters(DatagridMapper $datagridMapper)
    {
        $datagridMapper
            ->add('email')
            ->add('is_active');
    }
 
    protected function configureListFields(ListMapper $listMapper)
    {
        $listMapper
            ->add('is_active')
            ->addIdentifier('email')
            ->add('url')
            ->add('created_at')
            ->add('token')
        ;
    }
}
```

为辅助管理员工作，我们想要只显示出未激活的*Affiliate*。我们能够通过过滤*is_active*值为*false*的*Affiliate*来实现这个功能：

```PHP
// src/Ibw/JobeetBundle/Admin/AffiliateAdmin.php
// ...
    protected $datagridValues = array(
        '_sort_order' => 'ASC',
        '_sort_by' => 'is_active',
        'is_active' => array('value' => 2) // The value 2 represents that the displayed affiliate accounts are not activated yet
    );
 
// ...
```

现在我们创建*AffiliateAdminController*：

```PHP
// src/Ibw/JobeetBundle/Controller/AffiliateAdminController.php
namespace Ibw\JobeetBundle\Controller;
 
use Sonata\AdminBundle\Controller\CRUDController as Controller;
use Sonata\DoctrineORMAdminBundle\Datagrid\ProxyQuery as ProxyQueryInterface;
use Symfony\Component\HttpFoundation\RedirectResponse;
 
class AffiliateAdminController extends Controller
{
    // Your code goes here
}
```

我们来创建*activate*和*deactivate*批量操作：

```PHP
// src/Ibw/JobeetBundle/Controller/AffiliateAdminController.php
namespace Ibw\JobeetBundle\Controller;
 
use Sonata\AdminBundle\Controller\CRUDController as Controller;
use Sonata\DoctrineORMAdminBundle\Datagrid\ProxyQuery as ProxyQueryInterface;
use Symfony\Component\HttpFoundation\RedirectResponse;
 
class AffiliateAdminController extends Controller
{
    public function batchActionActivate(ProxyQueryInterface $selectedModelQuery)
    {
        if($this->admin->isGranted('EDIT') === false || $this->admin->isGranted('DELETE') === false) {
            throw new AccessDeniedException();
        }
 
        $request = $this->get('request');
        $modelManager = $this->admin->getModelManager();
 
        $selectedModels = $selectedModelQuery->execute();
 
        try {
            foreach($selectedModels as $selectedModel) {
                $selectedModel->activate();
                $modelManager->update($selectedModel);
            }
        } catch(\Exception $e) {
            $this->get('session')->getFlashBag()->add('sonata_flash_error', $e->getMessage());
 
            return new RedirectResponse($this->admin->generateUrl('list',$this->admin->getFilterParameters()));
        }
 
        $this->get('session')->getFlashBag()->add('sonata_flash_success',  sprintf('The selected accounts have been activated'));
 
        return new RedirectResponse($this->admin->generateUrl('list',$this->admin->getFilterParameters()));
    }
 
    public function batchActionDeactivate(ProxyQueryInterface $selectedModelQuery)
    {
        if($this->admin->isGranted('EDIT') === false || $this->admin->isGranted('DELETE') === false) {
            throw new AccessDeniedException();
        }
 
        $request = $this->get('request');
        $modelManager = $this->admin->getModelManager();
 
        $selectedModels = $selectedModelQuery->execute();
 
        try {
            foreach($selectedModels as $selectedModel) {
                $selectedModel->deactivate();
                $modelManager->update($selectedModel);
            }
        } catch(\Exception $e) {
            $this->get('session')->getFlashBag()->add('sonata_flash_error', $e->getMessage());
 
            return new RedirectResponse($this->admin->generateUrl('list',$this->admin->getFilterParameters()));
        }
 
        $this->get('session')->getFlashBag()->add('sonata_flash_success',  sprintf('The selected accounts have been deactivated'));
 
        return new RedirectResponse($this->admin->generateUrl('list',$this->admin->getFilterParameters()));
    }
}
```

为了能够让批量操作生效，我们把它添加到*Admin::getBatchActions()*方法中：

```PHP
// src/Ibw/JobeetBundle/Admin/AffiliateAdmin.php
class AffiliateAdmin extends Admin
{
    // ... 
 
    public function getBatchActions()
    {
        $actions = parent::getBatchActions();
 
        if($this->hasRoute('edit') && $this->isGranted('EDIT') && $this->hasRoute('delete') && $this->isGranted('DELETE')) {
            $actions['activate'] = array(
                'label'            => 'Activate',
                'ask_confirmation' => true
            );
 
            $actions['deactivate'] = array(
                'label'            => 'Deactivate',
                'ask_confirmation' => true
            );
        }
 
        return $actions;
    }
}
```

在此我们还需要给*Affiliate*实体添加两个方法：*activate()*和*deactivate()*：

```PHP
// src/Ibw/JobeetBundle/Entity/Affiliate.php
// ...
 
    public function activate()
    {
        if(!$this->getIsActive()) {
            $this->setIsActive(true);
        }
 
        return $this->is_active;
    }
 
    public function deactivate()
    {
        if($this->getIsActive()) {
            $this->setIsActive(false);
        }
 
        return $this->is_active;
    }
```

现在我们为每条*Affiliate*信息都创建两个独立的动作，*activate*和*deactivate*。首先我们先为它们创建路由，这就是为什么我们的*Admin*类中重写了*configureRoutes*函数：

```PHP
// src/Ibw/JobeetBundle/Admin/AffiliateAdmin.php
use Sonata\AdminBundle\Route\RouteCollection;
 
class AffiliateAdmin extends Admin
{
    // ...
 
    protected function configureRoutes(RouteCollection $collection) {
        parent::configureRoutes($collection);
 
        $collection->add('activate',
            $this->getRouterIdParameter().'/activate')
        ;
 
        $collection->add('deactivate',
            $this->getRouterIdParameter().'/deactivate')
        ;
    }
}
```

现在我们在*AdminController*中实现这两个动作：

```PHP
// src/Ibw/JobeetBundle/Controller/AffiliateAdminController.php
class AffiliateAdminController extends Controller
{
    // ...
 
    public function activateAction($id)
    {
        if($this->admin->isGranted('EDIT') === false) {
            throw new AccessDeniedException();
        }
 
        $em = $this->getDoctrine()->getManager();
        $affiliate = $em->getRepository('IbwJobeetBundle:Affiliate')->findOneById($id);
 
        try {
            $affiliate->setIsActive(true);
            $em->flush();
        } catch(\Exception $e) {
            $this->get('session')->getFlashBag()->add('sonata_flash_error', $e->getMessage());
 
            return new RedirectResponse($this->admin->generateUrl('list', $this->admin->getFilterParameters()));
        }
 
        return new RedirectResponse($this->admin->generateUrl('list',$this->admin->getFilterParameters()));
 
    }
 
    public function deactivateAction($id)
    {
        if($this->admin->isGranted('EDIT') === false) {
            throw new AccessDeniedException();
        }
 
        $em = $this->getDoctrine()->getManager();
        $affiliate = $em->getRepository('IbwJobeetBundle:Affiliate')->findOneById($id);
 
        try {
            $affiliate->setIsActive(false);
            $em->flush();
        } catch(\Exception $e) {
            $this->get('session')->getFlashBag()->add('sonata_flash_error', $e->getMessage());
 
            return new RedirectResponse($this->admin->generateUrl('list', $this->admin->getFilterParameters()));
        }
 
        return new RedirectResponse($this->admin->generateUrl('list',$this->admin->getFilterParameters()));
    }
}
```

现在为新添加的*action*创建模板：

```HTML
<!-- src/Ibw/JobeetBundle/Resources/views/AffiliateAdmin/list__action_activate.html.twig -->
{% if admin.isGranted('EDIT', object) and admin.hasRoute('activate') %}
    <a href="{{ admin.generateObjectUrl('activate', object) }}" class="btn edit_link" title="{{ 'action_activate'|trans({}, 'SonataAdminBundle') }}">
        <i class="icon-edit"></i>
        {{ 'activate'|trans({}, 'SonataAdminBundle') }}
    </a>
{% endif %}
```

```HTML
<!-- src/Ibw/JobeetBundle/Resources/views/AffiliateAdmin/list__action_deactivate.html.twig -->
{% if admin.isGranted('EDIT', object) and admin.hasRoute('deactivate') %}
    <a href="{{ admin.generateObjectUrl('deactivate', object) }}" class="btn edit_link" title="{{ 'action_deactivate'|trans({}, 'SonataAdminBundle') }}">
        <i class="icon-edit"></i>
        {{ 'deactivate'|trans({}, 'SonataAdminBundle') }}
    </a>
{% endif %}
```

在*AffiliateAdmin::configureListFields()*方法中添加新的action和button后，我们就能在页面上看到每条*Affiliate*信息都把它们显示出来：

```PHP
// src/Ibw/JobeetBundle/Admin/AffiliateAdmin.php
class AffiliateAdmin extends Admin
{
    // ...    

    protected function configureListFields(ListMapper $listMapper)
    {
        $listMapper
            ->add('is_active')
            ->addIdentifier('email')
            ->add('url')
            ->add('created_at')
            ->add('token')
            ->add('_action', 'actions', array( 'actions' => array('activate' => array('template' => 'IbwJobeetBundle:AffiliateAdmin:list__action_activate.html.twig'),
                'deactivate' => array('template' => 'IbwJobeetBundle:AffiliateAdmin:list__action_deactivate.html.twig'))))
        ;
    }
    /// ...
}
```

好的，马上清除掉*cache*来试试吧！

今天我们就到这了，明天我们会讲解发送邮件，当*Affiliate*被激活后将收到一封邮件。

# 许可证 #

如果您需要转载的话，请尊重原作者的知识产权，您可以通过把如下链接放到您转载文章中的头部或者尾部，谢谢。

原文链接：<http://www.intelligentbee.com/blog/2013/08/21/symfony2-jobeet-day-15-web-services/>

您可以在以下链接查看该许可证的全文：

![](../imgs/license.png)

<http://creativecommons.org/licenses/by-nc/3.0/legalcode>
