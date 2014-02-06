# 第九天：功能测试 #

*这一系列文章来源于Fabien Potencier，基于Symfony1.4编写的[Jobeet Tutirual](http://symfony.com/legacy/doc/jobeet?orm=Doctrine)。

功能测试是一个很好的端到端（end to end）测试工具，端到端就是浏览器发出请求并接受服务器响应的过程。功能测试用于测试应用程序的所有层：*路由（the routing）*，*模型（the model）*，*行为（the actions）*和*模板（the templates）*。功能测试和你曾经手动做过的测试十分相似：每次当你为网站修改或者添加一个行为的的时候，你会去浏览器中检查被渲染页面中的链接或者页面元素是否和预期的结果一致。换句话说，你的手动测试其实就是对网站功能在适当的情境下进行测试。

因为手动测试的过程是乏味的而且容易出错。每次你修改了代码，你必须对所有的情景一步步地进行调试以确保你做的修改是正确的。这样的繁复的手动测试会让人疯掉的。Symfony中的功能测试提供了一种简单的方式来描述（decribe）情景（scenarios）。每个场景都能够一遍又一遍地再现模拟用户在浏览器中的操作。功能测试和单元测试一样，它能够让你对编写好的代码更加有信心。

功能测试用一个明确的工作流：

* 发出请求
* 测试响应
* 点击一个链接或者提交一个表单
* 测试响应
* Rinse and repeat

## 第一个功能测试 ##

功能测试通常是一个存放在*Tests/Controller*目录下的一个*PHP*文件。如果你想要测试被*CategoryController*类处理的页面，那么就需要创建*CategoryControllerTest*类，让它继承*WebTestCase*类：

```PHP
// src/Ibw/JobeetBundle/Tests/Controller/CategoryControllerTest.php
namespace Ibw\JobeetBundle\Tests\Controller;
 
use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;
use Symfony\Bundle\FrameworkBundle\Console\Application;
use Symfony\Component\Console\Output\NullOutput;
use Symfony\Component\Console\Input\ArrayInput;
use Doctrine\Bundle\DoctrineBundle\Command\DropDatabaseDoctrineCommand;
use Doctrine\Bundle\DoctrineBundle\Command\CreateDatabaseDoctrineCommand;
use Doctrine\Bundle\DoctrineBundle\Command\Proxy\CreateSchemaDoctrineCommand;
 
class CategoryControllerTest extends WebTestCase
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
 
    public function testShow()
    {
        $client = static::createClient();
 
        $crawler = $client->request('GET', '/category/index');
        $this->assertEquals('Ibw\JobeetBundle\Controller\CategoryController::showAction', $client->getRequest()->attributes->get('_controller'));
        $this->assertTrue(200 === $client->getResponse()->getStatusCode());
    }
}
```

想要学些更多关于*crawler*的知识，可以通过[Symfony文档](http://symfony.com/doc/current/book/testing.html#the-crawler)进行学习。

## 运行功能测试 ##

和单元测试一样，我们可以通过*PHPUnit*命令来运行功能测试：

    phpunit -c app/ src/Ibw/JobeetBundle/Tests/Controller/CategoryControllerTest

运行测试不能通过，因为测试的URL */category/index*在*Jobeet*中是无效的。


    PHPUnit 3.7.22 by Sebastian Bergmann.
 
    Configuration read from /var/www/jobeet/app/phpunit.xml.dist
 
    F
 
    Time: 2 seconds, Memory: 25.25Mb
 
    There was 1 failure:
 
    1) Ibw\JobeetBundle\Tests\Controller\CategoryControllerTest::testShow
    Failed asserting that false is true.

## 编写功能测试 ##

编写单元测试就像是在浏览器中进行情景模拟。我们在第二天中已经把所有的场景都已经描述出来了，我需要测试它们。

首先，我们来编辑*JobControllerTest*类来测试*Jobeet*的首页。我们会一个个地对这些功能进行测试：

### 过期的*job*不应该被显示出来 ###

```PHP
// src/Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php
namespace Ibw\JobeetBundle\Tests\Controller;
 
use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;
use Symfony\Bundle\FrameworkBundle\Console\Application;
use Symfony\Component\Console\Output\NullOutput;
use Symfony\Component\Console\Input\ArrayInput;
use Doctrine\Bundle\DoctrineBundle\Command\DropDatabaseDoctrineCommand;
use Doctrine\Bundle\DoctrineBundle\Command\CreateDatabaseDoctrineCommand;
use Doctrine\Bundle\DoctrineBundle\Command\Proxy\CreateSchemaDoctrineCommand;
 
class JobControllerTest extends WebTestCase
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
 
    public function testIndex()
    {
        $client = static::createClient();
        $crawler = $client->request('GET', '/');
 
        $this->assertEquals('Ibw\JobeetBundle\Controller\JobController::indexAction', $client->getRequest()->attributes->get('_controller'));
        $this->assertTrue($crawler->filter('.jobs td.position:contains("Expired")')->count() == 0);
    }
}
```

为了验证过期的*job*不能在首页中显示出来，我们通过对服务器返回的HTML页面进行匹配，检查是否有能和*css*选择器*.jobs td.position:contains("Expired")*匹配的HTML内容（还记得在*fixtures*吗？我们只在过期*job*的*position*中包含有“Expired”）。

### 一个*category*只能显示N行*job* ###

把下面的代码添加到*testIndex()*函数后面。为了在功能测试中得到*app/config/config.yml*中的自定义参数，我们会使用到*内核（kernel）*:

```PHP
// src/Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php
public function testIndex()
{
    //...
    $kernel = static::createKernel();
    $kernel->boot();
    $max_jobs_on_homepage = $kernel->getContainer()->getParameter('max_jobs_on_homepage');
    $this->assertTrue($crawler->filter('.category_programming tr')->count() <= $max_jobs_on_homepage );
}
```

为了能进行测试，我们需要为*Job/index.html.twig*中的每个*category*添加适当的*css类*（这样做才能够选择出每个*category*并计算被显示出*job*的数量）：


```HTML
<!-- src/Ibw/JobeetBundle/Resources/views/Job/index.html.twig -->
<!-- ... -->
 
    {% for category in categories %}
        <div class="category_{{ category.slug }}">
           <div class="category">
<!-- ... -->
```

### 只有当*category*中有过多的*job*时才有*and...more...* ###

```PHP
// src/Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php
public function testIndex()
{
    //...
    $this->assertTrue($crawler->filter('.category_design .more_jobs')->count() == 0);
    $this->assertTrue($crawler->filter('.category_programming .more_jobs')->count() == 1);
}
```

在这个测试中，我们检查*design*分类没有*"more jobs"*链接（即*\.category\_design \.more\_jobs*不存在），检查*programming*有*"more jobs"*链接（即*\.category\_design \.more\_jobs*存在）。

### *job*按照日期排序 ###

为了检查*job*是否按照日期来排序，我们需要检查第一行*job*是否和我们所预期的一样。通过检查URL中是否包含有预期*job*的主键来实现这个测试。主键可能在运行期间被改变，我们首先要从数据库中得到*Doctrine*对象。

```PHP
// src/Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php
public function testIndex()
{    
    // ...
    $em = $kernel->getContainer()->get('doctrine.orm.entity_manager');
 
    $query = $em->createQuery('SELECT j from IbwJobeetBundle:Job j LEFT JOIN j.category c WHERE c.slug = :slug AND j.expires_at > :date ORDER BY j.created_at DESC');
    $query->setParameter('slug', 'programming');
    $query->setParameter('date', date('Y-m-d H:i:s', time()));
    $query->setMaxResults(1);
    $job = $query->getSingleResult();
 
    $this->assertTrue($crawler->filter('.category_programming tr')->first()->filter(sprintf('a[href*="/%d/"]', $job->getId()))->count() == 1);
}
```

即使测试只在测试阶段才运行，但是我们也需要对代码进行重构，比如像获得*programming*分类中的第一个*job*的功能，使得这个功能在其它的地方也能够被重用。我们不会把这些代码移到*Model*层里，因为这些代码只用在编写测试中。相反地，我们会把代码放在测试类里的*getMostRecentProgrammingJob()*函数中：

```PHP
// src/Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php
// ...
 
public function getMostRecentProgrammingJob()
{
    $kernel = static::createKernel();
    $kernel->boot();
    $em = $kernel->getContainer()->get('doctrine.orm.entity_manager');

    $query = $em->createQuery('SELECT j from IbwJobeetBundle:Job j LEFT JOIN j.category c WHERE c.slug = :slug AND j.expires_at > :date ORDER BY j.created_at DESC');
    $query->setParameter('slug', 'programming');
    $query->setParameter('date', date('Y-m-d H:i:s', time()));
    $query->setMaxResults(1);

    return $query->getSingleResult();
}
 
// ...
```

现在用下面的代码替换之前的代码：

```PHP
// src/Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php
// ...
 
$this->assertTrue($crawler->filter('.category_programming tr')->first()->filter(sprintf('a[href*="/%d/"]', $this->getMostRecentProgrammingJob()->getId()))->count() == 1);
 
//...
```

### 在首页中的每个*job*都是可以点击的 ###

为了测试首页中*job*的链接，我们在*“Web Developer”*文本上进行模拟点击。首页中有很多的*job*，我们指定要求浏览器点击第一个*job*。

测试URL中的每个请求参数，以确保点击链接后的路由能够正确地工作。

```PHP
// src/Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php 
public function testIndex() 
{
    // ...
 
    $job = $this->getMostRecentProgrammingJob();
    $link = $crawler->selectLink('Web Developer')->first()->link();
    $crawler = $client->click($link);
    $this->assertEquals('Ibw\JobeetBundle\Controller\JobController::showAction', $client->getRequest()->attributes->get('_controller'));
    $this->assertEquals($job->getCompanySlug(), $client->getRequest()->attributes->get('company'));
    $this->assertEquals($job->getLocationSlug(), $client->getRequest()->attributes->get('location'));
    $this->assertEquals($job->getPositionSlug(), $client->getRequest()->attributes->get('position'));
    $this->assertEquals($job->getId(), $client->getRequest()->attributes->get('id'));
}
 
// ...
```

### 学习例子 ###

在这一部分中包含有测试*job*和*category*页面的所需的代码。请仔细阅读代码，你也许能够从中学到一些小技巧：

```PHP
// src/Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php
namespace Ibw\JobeetBundle\Tests\Controller;
 
use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;
use Symfony\Bundle\FrameworkBundle\Console\Application;
use Symfony\Component\Console\Output\NullOutput;
use Symfony\Component\Console\Input\ArrayInput;
use Doctrine\Bundle\DoctrineBundle\Command\DropDatabaseDoctrineCommand;
use Doctrine\Bundle\DoctrineBundle\Command\CreateDatabaseDoctrineCommand;
use Doctrine\Bundle\DoctrineBundle\Command\Proxy\CreateSchemaDoctrineCommand;
 
class JobControllerTest extends WebTestCase
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
 
    public function getMostRecentProgrammingJob()
    {
        $kernel = static::createKernel();
        $kernel->boot();
        $em = $kernel->getContainer()->get('doctrine.orm.entity_manager');
 
        $query = $em->createQuery('SELECT j from IbwJobeetBundle:Job j LEFT JOIN j.category c WHERE c.slug = :slug AND j.expires_at > :date ORDER BY j.created_at DESC');
        $query->setParameter('slug', 'programming');
        $query->setParameter('date', date('Y-m-d H:i:s', time()));
        $query->setMaxResults(1);
 
        return $query->getSingleResult();
    }
 
    public function getExpiredJob()
    {
        $kernel = static::createKernel();
        $kernel->boot();
        $em = $kernel->getContainer()->get('doctrine.orm.entity_manager');
 
        $query = $em->createQuery('SELECT j from IbwJobeetBundle:Job j WHERE j.expires_at < :date');             
        $query->setParameter('date', date('Y-m-d H:i:s', time()));
        $query->setMaxResults(1);
 
        return $query->getSingleResult();
    }
 
    public function testIndex()
    {
        // get the custom parameters from app config.yml
        $kernel = static::createKernel();
        $kernel->boot();
        $max_jobs_on_homepage = $kernel->getContainer()->getParameter('max_jobs_on_homepage');
 
        $client = static::createClient();
 
        $crawler = $client->request('GET', '/');
        $this->assertEquals('Ibw\JobeetBundle\Controller\JobController::indexAction', $client->getRequest()->attributes->get('_controller'));
 
        // expired jobs are not listed
        $this->assertTrue($crawler->filter('.jobs td.position:contains("Expired")')->count() == 0);
 
        // only $max_jobs_on_homepage jobs are listed for a category
        $this->assertTrue($crawler->filter('.category_programming tr')->count()<= $max_jobs_on_homepage); 
        $this->assertTrue($crawler->filter('.category_design .more_jobs')->count() == 0);
        $this->assertTrue($crawler->filter('.category_programming .more_jobs')->count() == 1);
 
        // jobs are sorted by date
        $this->assertTrue($crawler->filter('.category_programming tr')->first()->filter(sprintf('a[href*="/%d/"]', $this->getMostRecentProgrammingJob()->getId()))->count() == 1);
 
        // each job on the homepage is clickable and give detailed information
        $job = $this->getMostRecentProgrammingJob();
        $link = $crawler->selectLink('Web Developer')->first()->link();
        $crawler = $client->click($link);
        $this->assertEquals('Ibw\JobeetBundle\Controller\JobController::showAction', $client->getRequest()->attributes->get('_controller'));
        $this->assertEquals($job->getCompanySlug(), $client->getRequest()->attributes->get('company'));
        $this->assertEquals($job->getLocationSlug(), $client->getRequest()->attributes->get('location'));
        $this->assertEquals($job->getPositionSlug(), $client->getRequest()->attributes->get('position'));
        $this->assertEquals($job->getId(), $client->getRequest()->attributes->get('id'));
 
        // a non-existent job forwards the user to a 404
        $crawler = $client->request('GET', '/job/foo-inc/milano-italy/0/painter');
        $this->assertTrue(404 === $client->getResponse()->getStatusCode());
 
        // an expired job page forwards the user to a 404
        $crawler = $client->request('GET', sprintf('/job/sensio-labs/paris-france/%d/web-developer', $this->getExpiredJob()->getId()));
        $this->assertTrue(404 === $client->getResponse()->getStatusCode());
    }
}
```

```PHP
// src/Ibw/JobeetBundle/Tests/Controller/CategoryControllerTest.php
namespace Ibw\JobeetBundle\Tests\Controller;
 
use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;
use Symfony\Bundle\FrameworkBundle\Console\Application;
use Symfony\Component\Console\Output\NullOutput;
use Symfony\Component\Console\Input\ArrayInput;
use Doctrine\Bundle\DoctrineBundle\Command\DropDatabaseDoctrineCommand;
use Doctrine\Bundle\DoctrineBundle\Command\CreateDatabaseDoctrineCommand;
use Doctrine\Bundle\DoctrineBundle\Command\Proxy\CreateSchemaDoctrineCommand;
 
class CategoryControllerTest extends WebTestCase
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
 
    public function testShow()
    {
        $kernel = static::createKernel();
        $kernel->boot();
 
        // get the custom parameters from app/config.yml
        $max_jobs_on_category = $kernel->getContainer()->getParameter('max_jobs_on_category');
        $max_jobs_on_homepage = $kernel->getContainer()->getParameter('max_jobs_on_homepage');
 
        $client = static::createClient();
 
        $categories = $this->em->getRepository('IbwJobeetBundle:Category')->getWithJobs();
 
        // categories on homepage are clickable
        foreach($categories as $category) {
            $crawler = $client->request('GET', '/');
 
            $link = $crawler->selectLink($category->getName())->link();
            $crawler = $client->click($link);
 
            $this->assertEquals('Ibw\JobeetBundle\Controller\CategoryController::showAction', $client->getRequest()->attributes->get('_controller'));
            $this->assertEquals($category->getSlug(), $client->getRequest()->attributes->get('slug'));
 
            $jobs_no = $this->em->getRepository('IbwJobeetBundle:Job')->countActiveJobs($category->getId()); 
 
            // categories with more than $max_jobs_on_homepage jobs also have a "more" link                 
            if($jobs_no > $max_jobs_on_homepage) {
                $crawler = $client->request('GET', '/');
                $link = $crawler->filter(".category_" . $category->getSlug() . " .more_jobs a")->link();
                $crawler = $client->click($link);
 
                $this->assertEquals('Ibw\JobeetBundle\Controller\CategoryController::showAction', $client->getRequest()->attributes->get('_controller'));
                $this->assertEquals($category->getSlug(), $client->getRequest()->attributes->get('slug'));
            }
 
            $pages = ceil($jobs_no/$max_jobs_on_category);
 
            // only $max_jobs_on_category jobs are listed 
            $this->assertTrue($crawler->filter('.jobs tr')->count() <= $max_jobs_on_category);
            $this->assertRegExp("/" . $jobs_no . " jobs/", $crawler->filter('.pagination_desc')->text());
 
            if($pages > 1) {
                $this->assertRegExp("/page 1\/" . $pages . "/", $crawler->filter('.pagination_desc')->text());
 
                for ($i = 2; $i <= $pages; $i++) {
                    $link = $crawler->selectLink($i)->link();
                    $crawler = $client->click($link);
 
                    $this->assertEquals('Ibw\JobeetBundle\Controller\CategoryController::showAction', $client->getRequest()->attributes->get('_controller'));
                    $this->assertEquals($i, $client->getRequest()->attributes->get('page'));
                    $this->assertTrue($crawler->filter('.jobs tr')->count() <= $max_jobs_on_category);
                    if($jobs_no >1) {
                        $this->assertRegExp("/" . $jobs_no . " jobs/", $crawler->filter('.pagination_desc')->text());
                    }
                    $this->assertRegExp("/page " . $i . "\/" . $pages . "/", $crawler->filter('.pagination_desc')->text());
                }
            }     
        }
    }
}
```

今天我们就先到这啦，明天我们会讲解表单（forms），那么明天见！

# 许可证 #

如果您需要转载的话，请尊重原作者的知识产权，您可以通过把如下链接放到您转载文章中的头部或者尾部，谢谢。

原文链接：<http://www.intelligentbee.com/blog/2013/08/15/symfony2-jobeet-day-9-the-functional-tests/>

您可以在以下链接查看该许可证的全文：

![](../imgs/license.png)

<http://creativecommons.org/licenses/by-nc/3.0/legalcode>