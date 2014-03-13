# 第十一天：对你的表单进行测试 #

*这一系列文章来源于Fabien Potencier，基于Symfony1.4编写的[Jobeet Tutirual](http://symfony.com/legacy/doc/jobeet?orm=Doctrine)。

在第十天的内容中，我们使用*Symfony 2.3*创建了我们的第一个表单。现在用户能够在*Jobeet*上发布*Job*信息了，但是我们还没来得及给它做测试呢。别担心，我们会沿着这种（边开发边测试的）开发模式进行下去的。

## 提交表单 ##

打开*JobControllerTest.php*，为*Job*信息的创建和表单验证添加功能测试。我们在文件末尾加入下面的代码，测试是否能够正确访问到创建*Job*的页面：

```PHP
// src/Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php
// ...
 
public function testJobForm()
{
    $client = static::createClient();
 
    $crawler = $client->request('GET', '/job/new');
    $this->assertEquals('Ibw\JobeetBundle\Controller\JobController::newAction', $client->getRequest()->attributes->get('_controller'));
}
```

为了让功能测试能够模拟点击按钮来提交表单，我们会使用*selectButton()*方法。这个方法能够选择*button*标签和*submit*类型的*input*标签。只要你有一个表示*button*的*Crawler*，你就可以调用*form()*方法得到包含这个*button*结点的表单的*Form*实例。

```PHP
$form = $crawler->selectButton('Submit Form')->form();
```

> 上面的例子中选择了属性值为*Submit Form*的*submit*类型的*input*标签。

我们可以在调用*form()*方法的时候给它传递一个数组类型的参数来重载默认的调用：

```PHP
$form = $crawler->selectButton('submit')->form(array(
    'name' => 'Fabien',
    'my_form[subject]' => 'Symfony Rocks!'
));
```

现在我们来实际操作一下选择表单和传递参数的过程：

```PHP
// src/Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php
// ...
 
public function testJobForm()
{
    $client = static::createClient();
 
    $crawler = $client->request('GET', '/job/new');
    $this->assertEquals('Ibw\JobeetBundle\Controller\JobController::newAction', $client->getRequest()->attributes->get('_controller'));
 
    $form = $crawler->selectButton('Preview your job')->form(array(
        'job[company]'      => 'Sensio Labs',
        'job[url]'          => 'http://www.sensio.com/',
        'job[file]'         => __DIR__.'/../../../../../web/bundles/ibwjobeet/images/sensio-labs.gif',
        'job[position]'     => 'Developer',
        'job[location]'     => 'Atlanta, USA',
        'job[description]'  => 'You will work with symfony to develop websites for our customers.',
        'job[how_to_apply]' => 'Send me an email',
        'job[email]'        => 'for.a.job@example.com',
        'job[is_public]'    => false,
    ));
 
    $client->submit($form);
    $this->assertEquals('Ibw\JobeetBundle\Controller\JobController::createAction', $client->getRequest()->attributes->get('_controller'));
}
```

我们也可以模拟浏览器中的文件上传，我们只要把上传文件的绝对路径传递给它就行了。

表单提交之后，我们需要测试处理提交表单的*action*是否为*create*。

## 测试表单 ##

如果表单的值是有效的，那么*Job*信息应该就可以创建成功，然后用户会被重定向到*preview*页面：

```PHP
// src/Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php
public function testJobForm()
{
    // ...
    $client->followRedirect();
    $this->assertEquals('Ibw\JobeetBundle\Controller\JobController::previewAction', $client->getRequest()->attributes->get('_controller'));
}
```

## 测试数据库记录 ##

最后，我们需要测试数据库中是否有刚才创建的*Job*数据，还需要测试当用户未公布*Job*信息时*is_activated*列是否被设置成*false*：

```PHP
// src/Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php
public function testJobForm()
{
    // ...
    $kernel = static::createKernel();
    $kernel->boot();
    $em = $kernel->getContainer()->get('doctrine.orm.entity_manager');
 
    $query = $em->createQuery('SELECT count(j.id) from IbwJobeetBundle:Job j WHERE j.location = :location AND j.is_activated IS NULL AND j.is_public = 0');
    $query->setParameter('location', 'Atlanta, USA');
    $this->assertTrue(0 < $query->getSingleScalarResult());
}
```

## 测试错误的表单 ##

正确的*Job*表单信息提交后的结果和我们预期的结果一样。现在我们来为错误的*Job*信息表单做测试：

```PHP
// src/Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php
public function testJobForm()
{
    // ...
    $crawler = $client->request('GET', '/job/new');
    $form = $crawler->selectButton('Preview your job')->form(array(
        'job[company]'      => 'Sensio Labs',
        'job[position]'     => 'Developer',
        'job[location]'     => 'Atlanta, USA',
        'job[email]'        => 'not.an.email',
    ));
    $crawler = $client->submit($form);
 
    // check if we have 3 errors
    $this->assertTrue($crawler->filter('.error_list')->count() == 3);
 
    // check if we have error on job_description field
    $this->assertTrue($crawler->filter('#job_description')->siblings()->first()->filter('.error_list')->count() == 1);
 
    // check if we have error on job_how_to_apply field
    $this->assertTrue($crawler->filter('#job_how_to_apply')->siblings()->first()->filter('.error_list')->count() == 1);
 
    // check if we have error on job_email field
    $this->assertTrue($crawler->filter('#job_email')->siblings()->first()->filter('.error_list')->count() == 1);
}
```

现在我们来测试*preview*页面中的*admin*栏。当一条*Job*信息还未被激活时，我们可以对它进行*edit*，*delete*或者*publish*操作。为了测试这三个*action*，我们首先需要创建一条*Job*信息。为了避免过多的复制和粘贴，我们给*JobControllerTest*类添加一个*createJob()*的方法：

```PHP
// src/Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php
// ...
 
public function createJob($values = array())
{
    $client = static::createClient();
    $crawler = $client->request('GET', '/job/new');
    $form = $crawler->selectButton('Preview your job')->form(array_merge(array(
        'job[company]'      => 'Sensio Labs',
        'job[url]'          => 'http://www.sensio.com/',
        'job[position]'     => 'Developer',
        'job[location]'     => 'Atlanta, USA',
        'job[description]'  => 'You will work with symfony to develop websites for our customers.',
        'job[how_to_apply]' => 'Send me an email',
        'job[email]'        => 'for.a.job@example.com',
        'job[is_public]'    => false,
  ), $values));
 
    $client->submit($form);
    $client->followRedirect();
 
    return $client;
}
```

*createJob()*方法创建一个*Job*对象，然后进行页面跳转（转到*preview*页面）。我们可以给*createJob()*方法传递一个数组，这个数组的值会覆盖默认的表单值。测试*Public*现在就变得很简单了：

```PHP
// src/Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php
public function testPublishJob()
{
    $client = $this->createJob(array('job[position]' => 'FOO1'));
    $crawler = $client->getCrawler();
    $form = $crawler->selectButton('Publish')->form();
    $client->submit($form);
 
    $kernel = static::createKernel();
    $kernel->boot();
    $em = $kernel->getContainer()->get('doctrine.orm.entity_manager');
 
    $query = $em->createQuery('SELECT count(j.id) from IbwJobeetBundle:Job j WHERE j.position = :position AND j.is_activated = 1');
    $query->setParameter('position', 'FOO1');
    $this->assertTrue(0 < $query->getSingleScalarResult());
}
```

测试*Delete*操作也很类似：

```PHP
// src/Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php
// ...
 
public function testDeleteJob()
{
    $client = $this->createJob(array('job[position]' => 'FOO2'));
    $crawler = $client->getCrawler();
    $form = $crawler->selectButton('Delete')->form();
    $client->submit($form);
 
    $kernel = static::createKernel();
    $kernel->boot();
    $em = $kernel->getContainer()->get('doctrine.orm.entity_manager');
 
    $query = $em->createQuery('SELECT count(j.id) from IbwJobeetBundle:Job j WHERE j.position = :position');
    $query->setParameter('position', 'FOO2');
    $this->assertTrue(0 == $query->getSingleScalarResult());
}
```

## 用测试作保障 ##

当一条*Job*信息被发布之后就不能再对它进行编辑了，尽管*Edit*链接已经不在*preview*页面中显示出来。我们来给它做个测试吧。

首先我们为*createJob()*方法添加另外一个参数，让*createJob()*方法能够自动发布*Job*信息。我们来添加一个*getJobByPosition()*方法，它能够按指定的*position*返回*Job*数据：

```PHP
// src/Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php
// ...
 
public function createJob($values = array(), $publish = false)
{
    $client = static::createClient();
    $crawler = $client->request('GET', '/job/new');
    $form = $crawler->selectButton('Preview your job')->form(array_merge(array(
        'job[company]'      => 'Sensio Labs',
        'job[url]'          => 'http://www.sensio.com/',
        'job[position]'     => 'Developer',
        'job[location]'     => 'Atlanta, USA',
        'job[description]'  => 'You will work with symfony to develop websites for our customers.',
        'job[how_to_apply]' => 'Send me an email',
        'job[email]'        => 'for.a.job@example.com',
        'job[is_public]'    => false,
  ), $values));
 
    $client->submit($form);
    $client->followRedirect();
 
    if($publish) {
      $crawler = $client->getCrawler();
      $form = $crawler->selectButton('Publish')->form();
      $client->submit($form);
      $client->followRedirect();
    }
 
  return $client;
}
 
public function getJobByPosition($position)
{
    $kernel = static::createKernel();
    $kernel->boot();
    $em = $kernel->getContainer()->get('doctrine.orm.entity_manager');
 
    $query = $em->createQuery('SELECT j from IbwJobeetBundle:Job j WHERE j.position = :position');
    $query->setParameter('position', $position);
    $query->setMaxResults(1);
    return $query->getSingleResult();
}
```

访问一个已发布*Job*信息的*edit*页面会返回**404**状态码：

```PHP
// src/Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php
// ...
 
public function testEditJob()
{
    $client = $this->createJob(array('job[position]' => 'FOO3'), true);
    $crawler = $client->getCrawler();
    $crawler = $client->request('GET', sprintf('/job/%s/edit', $this->getJobByPosition('FOO3')->getToken()));
    $this->assertTrue(404 === $client->getResponse()->getStatusCode());
}
```

如果你运行一下测试，你不会得到期望的结果，因为我们昨天忘了实现安全性。编写测试同样是一种发现*Bug*的好方法，就像你去考虑所有的测试边界值一样。

修改这个*Bug*很简单，我们只需为已激活的*Job*信息转向到**404页面**即可：

```PHP
// src/Ibw/JobeetBundle/Controller/JobController.php
// ...
 
public function editAction($token)
{
    $em = $this->getDoctrine()->getManager();
 
    $entity = $em->getRepository('IbwJobeetBundle:Job')->findOneByToken($token);
 
    if (!$entity) {
        throw $this->createNotFoundException('Unable to find Job entity.');
    }
 
    if ($entity->getIsActivated()) {
        throw $this->createNotFoundException('Job is activated and cannot be edited.');
    }
 
  // ...
}
```

## “穿越未来”的测试 ##

当一条*Job*信息过期天数少于5天，或者*Job*信息已经过期了，用户能够从当前日期开始延期该条*Job*信息多30天。在浏览器中做这个测试并不容易，因为当一个*Job*信息被创建出来后需要30天才过期，所以当访问这个刚创建的*Job*信息页面时，延期链接是不存在的。当然，你也可以修改数据库中该*Job*数据的过期时间，或者是调整模板让延期链接一直都显示出来，但是这些方法都很乏味而且容易出错。你现在已经可能猜到了，编写测试能够帮助我们解决问题。

一如既往，我们需要先为*extend*方法添加新路由：

```YAML
# src/Ibw/JobeetBundle/Resources/config/routing/Job.yml
# ...
 
ibw_job_extend:
    pattern:  /{token}/extend
    defaults: { _controller: "IbwJobeetBundle:Job:extend" }
    requirements: { _method: post }
```

然后修改*admin.html.twig*，用*extend*表单替换*Extend*链接：

```HTML
<!-- src/Ibw/JobeetBundle/Resources/views/Job/admin.html.twig -->
<!-- ... -->
 
{% if job.expiresSoon %}
    <form action="{{ path('ibw_job_extend', { 'token': job.token }) }}" method="post">
        {{ form_widget(extend_form) }}
        <button type="submit">Extend</button> for another 30 days
    </form>
{% endif %}
 
<!-- ... -->
```

然后再添加*extendAction()*方法和*createExtendForm()*方法：

```PHP
// src/Ibw/JobeetBundle/Controller/JobController.php
// ...

public function extendAction(Request $request, $token)
{
    $form = $this->createExtendForm($token);
    $request = $this->getRequest();

    $form->bind($request);

    if($form->isValid()) {
        $em=$this->getDoctrine()->getManager();
        $entity = $em->getRepository('IbwJobeetBundle:Job')->findOneByToken($token);

        if(!$entity){
            throw $this->createNotFoundException('Unable to find Job entity.');
        }

        if(!$entity->extend()){
            throw $this->createNodFoundException('Unable to extend the Job');
        }

        $em->persist($entity);
        $em->flush();

        $this->get('session')->getFlashBag()->add('notice', sprintf('Your job validity has been extended until %s', $entity->getExpiresAt()->format('m/d/Y')));
    }

    return $this->redirect($this->generateUrl('ibw_job_preview', array(
        'company' => $entity->getCompanySlug(),
        'location' => $entity->getLocationSlug(),
        'token' => $entity->getToken(),
        'position' => $entity->getPositionSlug()
    )));
}

private function createExtendForm($token)
{
    return $this->createFormBuilder(array('token' => $token))
        ->add('token', 'hidden')
        ->getForm();
}
```

同样地，为*previewAction()*添加*extend*表单：

```PHP
// src/Ibw/JobeetBundle/Controller/JobController.php
// ...
 
public function previewAction($token)
{
    $em = $this->getDoctrine()->getManager();
 
    $entity = $em->getRepository('IbwJobeetBundle:Job')->findOneByToken($token);
 
    if (!$entity) {
        throw $this->createNotFoundException('Unable to find Job entity.');
    }
 
    $deleteForm = $this->createDeleteForm($entity->getId());
    $publishForm = $this->createPublishForm($entity->getToken());
    $extendForm = $this->createExtendForm($entity->getToken());
 
    return $this->render('IbwJobeetBundle:Job:show.html.twig', array(
        'entity'      => $entity,
        'delete_form' => $deleteForm->createView(),
        'publish_form' => $publishForm->createView(),
        'extend_form' => $extendForm->createView(),
    ));
}
```

如果*Job*信息已经被*extend*了，那么*Job::extend()*方法返回*true*，否则返回*false*：

```PHP
// src/Ibw/JobeetBundle/Entity/Job.php
// ...
 
public function extend()
{
    if (!$this->expiresSoon())
    {
        return false;
    }
 
    $this->expires_at = new \DateTime(date('Y-m-d H:i:s', time() + 86400 * 30));
 
    return true;
}
```

最后我们来添加测试：

```PHP
// src/Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php
// ...
 
public function testExtendJob()
{
    // A job validity cannot be extended before the job expires soon
    $client = $this->createJob(array('job[position]' => 'FOO4'), true);
    $crawler = $client->getCrawler();
    $this->assertTrue($crawler->filter('input[type=submit]:contains("Extend")')->count() == 0);
 
    // A job validity can be extended when the job expires soon
 
    // Create a new FOO5 job
    $client = $this->createJob(array('job[position]' => 'FOO5'), true);
 
    // Get the job and change the expire date to today
    $kernel = static::createKernel();
    $kernel->boot();
    $em = $kernel->getContainer()->get('doctrine.orm.entity_manager');
    $job = $em->getRepository('IbwJobeetBundle:Job')->findOneByPosition('FOO5');
    $job->setExpiresAt(new \DateTime());
    $em->flush();
 
    // Go to the preview page and extend the job
    $crawler = $client->request('GET', sprintf('/job/%s/%s/%s/%s', $job->getCompanySlug(), $job->getLocationSlug(), $job->getToken(), $job->getPositionSlug()));
    $crawler = $client->getCrawler();
    $form = $crawler->selectButton('Extend')->form();
    $client->submit($form);
 
    // Reload the job from db
    $job = $this->getJobByPosition('FOO5');
 
    // Check the expiration date
    $this->assertTrue($job->getExpiresAt()->format('y/m/d') == date('y/m/d', time() + 86400 * 30));
}

```

## 维护任务 ##

尽管*Symfony*是一个*Web*框架，但是它也自带了一系列的命令行工具。我们已经使用过它的命令行工具来生成默认的应用程序包（bundle）目录结构和各种各样的*model*文件。在*Symfony*中添加自定义的命令也很简单。当用户创建了一条*Job*信息之后，用户必须去激活它让它上线，否则的话，久而久之数据库中就会有许多无用的（stale）*Job*数据。现在我们来自定义一个命令清除数据库中所有无用的*Job*数据，这个命令通常运行在计划任务（cron job）中。

```PHP
// src/Ibw/JobeetBundle/Command/JobeetCleanupCommand.php
namespace Ibw\JobeetBundle\Command;
 
use Symfony\Bundle\FrameworkBundle\Command\ContainerAwareCommand;
use Symfony\Component\Console\Input\InputArgument;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Input\InputOption;
use Symfony\Component\Console\Output\OutputInterface;
use Ibw\JobeetBundle\Entity\Job;
 
class JobeetCleanupCommand extends ContainerAwareCommand {
 
  protected function configure()
  {
      $this
          ->setName('ibw:jobeet:cleanup')
          ->setDescription('Cleanup Jobeet database')
          ->addArgument('days', InputArgument::OPTIONAL, 'The email', 90)
    ;
  }
 
  protected function execute(InputInterface $input, OutputInterface $output)
  {
      $days = $input->getArgument('days');
 
      $em = $this->getContainer()->get('doctrine')->getManager();
      $nb = $em->getRepository('IbwJobeetBundle:Job')->cleanup($days);
 
      $output->writeln(sprintf('Removed %d stale jobs', $nb));
  }
}
```

我们需要在*JobRepository*类中添加*cleanup()*方法：

```PHP
// src/Ibw/JobeetBundle/Repository/JobRepository.php
// ...
 
public function cleanup($days)
{
    $query = $this->createQueryBuilder('j')
        ->delete()
        ->where('j.is_activated IS NULL')
        ->andWhere('j.created_at < :created_at')    
        ->setParameter('created_at',  date('Y-m-d', time() - 86400 * $days))
        ->getQuery();
 
    return $query->execute();
}
```

在项目根目录下运行下面命令来执行：

    php app/console ibw:jobeet:cleanup

或者：

    php app/console ibw:jobeet:cleanup 10

上面命令将删除近10天内数据库中无用的*Job*数据。

# 许可证 #

如果您需要转载的话，请尊重原作者的知识产权，您可以通过把如下链接放到您转载文章中的头部或者尾部，谢谢。

原文链接：<http://www.intelligentbee.com/blog/2013/08/17/symfony2-jobeet-day-11-testing-your-forms/>

您可以在以下链接查看该许可证的全文：

![](../imgs/license.png)

<http://creativecommons.org/licenses/by-nc/3.0/legalcode>
