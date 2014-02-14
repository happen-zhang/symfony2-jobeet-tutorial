# 第十六天：邮件 #

*这一系列文章来源于Fabien Potencier，基于Symfony1.4编写的[Jobeet Tutirual](http://symfony.com/legacy/doc/jobeet?orm=Doctrine)。

昨天我们为Jobeet添加一个只可读的（read-only）web service。现在用户可以申请*Affiliate*账户了，但是他们需要被管理员激活后才能够使用。为了*affiliate*能给拿到他们的*token*，管理员需要发送邮件来通知他们。这些就是我们今天要实现的功能。

## Swift Mailer ##

Symfony框架捆绑了最好的PHP邮件解决方案：[Swift Mailer](http://www.swiftmailer.org/)。当然，Symfony已经完全地集成了这个库，并且在它原有的功能上添加了一些很酷很好用的功能。我们开始工作吧，我们通过发送邮件给用户通知他们已被管理员激活了，并且告诉他们对应的*token*是多少。但在那之前，我们需要配置我们的环境：

```YAML
# app/config/parameters.yml
# ...
    # ...
    mailer_transport:  gmail
    mailer_host:       ~
    mailer_user:       address@example.com
    mailer_password:   your_password
    # ...
```

> 为了让上面的配置能够起作用，你应该把*mailer_user*和*mailer_password*修改成真实的值。

对*app/config/parameters_test.yml*文件做同样的修改。

修改完这两个文件之后清除*test*环境和*development*环境下的缓存：

    php app/console cache:clear --env=dev
    php app/console cache:clear --env=prod

因为我们把*mailer_transport*设置为gmail，所以你需要使用google的邮箱地址来作为*mailer_user*的值。

我们可以想想，平时发送一个邮件的时候，我们都会在邮箱客户端进行哪几步操作呢？我们会填写邮件主题，填写接收人，填写要发送的信息内容。

为了发送邮件，我们需要：

* 调用*Swift_message*的*newInstance()*方法（通过[Swift Mailer的官方文档](http://www.swiftmailer.org/docs)可以学习到这个对象）
* 通过*setFrom()*方法来设置发送者的地址
* 通过*setSubject()*方法来设置邮件的主题
* 通过*setTo()*，*setCc()*，*setBcc()*之一来设置接收者
* 通过*setBody()*方法设置邮件内容

用下面的代码替换*activateAction()*：

```PHP
// src/Ibw/JobeetBundle/Controller/AffiliateAdminController.php
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
 
            $message = \Swift_Message::newInstance()
                ->setSubject('Jobeet affiliate token')
                ->setFrom('address@example.com')
                ->setTo($affiliate->getEmail())
                ->setBody(
                    $this->renderView('IbwJobeetBundle:Affiliate:email.txt.twig', array('affiliate' => $affiliate->getToken())))
            ;
 
            $this->get('mailer')->send($message);
        } catch(\Exception $e) {
            $this->get('session')->setFlash('sonata_flash_error', $e->getMessage());
        }
 
        return new RedirectResponse($this->admin->generateUrl('list',$this->admin->getFilterParameters()));
    }
 
// ...
```

发送邮件很简单，我们只需给*mailer*实例的*send()*方法传递一个*message*对象作为参数即可。

对于邮件的内容，我们创建了一个*email.txt.twig*文件，它准确地包含了我们想通知给*affilate*的内容。

```TXT
Your affiliate account has been activated.
Your secret token is {{affiliate}}.
You can see the jobs list at the following addresses:
http://jobeet.local/app_dev.php/api/{{affiliate}}/jobs.xml
or http://jobeet.local/app_dev.php/api/{{affiliate}}/jobs.json
or http://jobeet.local/app_dev.php/api/{{affiliate}}/jobs.yaml
```

现在我们也为*batchActionActivate*添加发送邮件的功能，尽管一次性同时选择多个*affiliate*账户同时激活，他们也能够接收到邮件：

```PHP
// src/Ibw/JobeetBundle/Controller/AffiliateAdminController.php
// ... 
 
    public function batchActionActivate(ProxyQueryInterface $selectedModelQuery)
    {
        // ...
 
        try {
            foreach($selectedModels as $selectedModel) {
                $selectedModel->activate();
                $modelManager->update($selectedModel);
 
                $message = \Swift_Message::newInstance()
                    ->setSubject('Jobeet affiliate token')
                    ->setFrom('address@example.com')
                    ->setTo($selectedModel->getEmail())
                    ->setBody(
                        $this->renderView('IbwJobeetBundle:Affiliate:email.txt.twig', array('affiliate' => $selectedModel->getToken())))
                ;
 
                $this->get('mailer')->send($message);
            }
        } catch(\Exception $e) {
            $this->get('session')->setFlash('sonata_flash_error', $e->getMessage());
 
            return new RedirectResponse($this->admin->generateUrl('list',$this->admin->getFilterParameters()));
        }
 
        // ...
    }
 
// ...
```

## 测试 ##

我们已经看到了怎么样使用Symfony来发送邮件，现在我们来写一些功能测试以确保它们能正确地工作。

为了测试新功能，我们需要进行登录。为了登录，我们需要用户名和密码。我们来创建先的fixture文件，在这个文件中添加admin用户：

```PHP
// src/Ibw/JobeetBundle/DataFixtures/ORM/LoadUserData.php
namespace Ibw\JobeetBundle\DataFixtures\ORM;
 
use Doctrine\Common\Persistence\ObjectManager;
use Doctrine\Common\DataFixtures\AbstractFixture;
use Doctrine\Common\DataFixtures\FixtureInterface;
use Doctrine\Common\DataFixtures\OrderedFixtureInterface;
use Symfony\Component\DependencyInjection\ContainerAwareInterface;
use Symfony\Component\DependencyInjection\ContainerInterface;
use Ibw\JobeetBundle\Entity\User;
 
class LoadUserData implements FixtureInterface, OrderedFixtureInterface, ContainerAwareInterface
{
    /**
     * @var ContainerInterface
     */
    private $container;
 
    /**
     * {@inheritDoc}
     */
    public function setContainer(ContainerInterface $container = null)
    {
        $this->container = $container;
    }
 
    /**
     * @param \Doctrine\Common\Persistence\ObjectManager $em
     */
    public function load(ObjectManager $em)
    {
        $user = new User();
        $user->setUsername('admin');
        $encoder = $this->container
            ->get('security.encoder_factory')
            ->getEncoder($user)
        ;
 
        $encodedPassword = $encoder->encodePassword('admin', $user->getSalt());
        $user->setPassword($encodedPassword);
 
        $em->persist($user);
        $em->flush();
    }
 
    public function getOrder()
    {
        return 4; // the order in which fixtures will be loaded
    }
}
```

在测试中，我们会使用分析器（profiler）中为*swiftmailer*的*collector*来得到上一个请求中的邮件发送信息。现在我们来添加一些代码检查邮件是否正确发送：

```PHP
// src/Ibw/JobeetBundle/Tests/Controller/AffiliateAdminControllerTest.php
namespace Ibw\JobeetBundle\Tests\Controller;
 
use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;
use Symfony\Bundle\FrameworkBundle\Console\Application;
use Symfony\Component\Console\Output\NullOutput;
use Symfony\Component\Console\Input\ArrayInput;
use Doctrine\Bundle\DoctrineBundle\Command\DropDatabaseDoctrineCommand;
use Doctrine\Bundle\DoctrineBundle\Command\CreateDatabaseDoctrineCommand;
use Doctrine\Bundle\DoctrineBundle\Command\Proxy\CreateSchemaDoctrineCommand;
 
class AffiliateAdminControllerTest extends WebTestCase
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
 
    public function testActivate()
    {
        $client = static::createClient();
 
        // Enable the profiler for the next request (it does nothing if the profiler is not available)
        $client->enableProfiler();
        $crawler = $client->request('GET', '/login');
 
        $form = $crawler->selectButton('login')->form(array(
            '_username'      => 'admin',
            '_password'      => 'admin'
        ));
 
        $crawler = $client->submit($form);
        $crawler = $client->followRedirect();
 
        $this->assertTrue(200 === $client->getResponse()->getStatusCode());
 
        $crawler = $client->request('GET', '/admin/ibw/jobeet/affiliate/list');
 
        $link = $crawler->filter('.btn.edit_link')->link();
        $client->click($link);
 
        $mailCollector = $client->getProfile()->getCollector('swiftmailer');
 
        // Check that an e-mail was sent
        $this->assertEquals(1, $mailCollector->getMessageCount());
 
        $collectedMessages = $mailCollector->getMessages();
        $message = $collectedMessages[0];
 
        // Asserting e-mail data
        $this->assertInstanceOf('Swift_Message', $message);
        $this->assertEquals('Jobeet affiliate token', $message->getSubject());
        $this->assertRegExp(
            '/Your secret token is symfony/',
            $message->getBody()
        );
    }
}
```

如果你现在运行测试将会得到错误。为了防止错误发生，我们需要确保*config_test.yml*文件中的*profiler*在测试环境里是开启的。如果它被设成fasle，请设成true：

```YAML
# app/config/config_test.yml
# ...
 
framework:
    test: ~
    session:
        storage_id: session.storage.mock_file
    profiler:
        enabled: true
 
# ...
```

清除缓存后运行测试命令，哈哈，好好享受*green bar*吧：

    phpunit -c app src/Ibw/JobeetBundle/Tests/Controller/AffiliateAdminControllerTest

# 许可证 #

如果您需要转载的话，请尊重原作者的知识产权，您可以通过把如下链接放到您转载文章中的头部或者尾部，谢谢。

原文链接：<http://www.intelligentbee.com/blog/2013/08/24/symfony2-jobeet-day-16-the-mailer/>

您可以在以下链接查看该许可证的全文：

![](../imgs/license.png)

<http://creativecommons.org/licenses/by-nc/3.0/legalcode>