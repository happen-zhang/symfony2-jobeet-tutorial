# 第三天：数据模型 #

*这一系列文章来源于Fabien Potencier，基于Symfony1.4编写的[Jobeet Tutirual](http://symfony.com/legacy/doc/jobeet?orm=Doctrine)。

假若你现在极其渴望打开你的文本编辑器来开始写`PHP`代码，那么今天就满足你的心愿，我们会开始做一些开发了。我们会定义`Jobeet`中使用到的模型，并使用`ORM`来和数据库交互，并且还会创建应用的第一个模块（module）。由于`Symfony`已经为我们做了很多工作，所以我们不用写太多的`PHP`代码就能拥有一个功能齐全的模块。

## 关系模型 ##

在[第二天](https://github.com/happen-zhang/symfony2-jobeet-tutorial/blob/master/chapter-02/chapter-02.md)中的用户stories描述了`Jobeet`项目中的主要对象：职位（jobs），affiliates和分类（categories）。下面是它们之间的实体关系图：

![](imgs/03-01.png)

除了我们在stories中描述的列（columns）之外，我们还添加了`created_at`和`updated_at`列。一个对象被保存或者是被更新的时候，`created_at`和`updated_at`这两个列将被自动更新。

## 数据库 ##

为了让职位（jobs），affiliate和分类（categories）能存放在数据库里，`Symfony2.3.2`使用的[ORM](http://www.doctrine-project.org/projects/orm.html)工具是[Doctrine](http://www.doctrine-project.org/projects/orm.html)。为了能够连接数据库，我们需要编辑`app/config/parameters.yml`文件（这里使用的是MySQL）：

```Yml
# app/config/parameters.yml
parameters:
    database\_driver: pdo\_mysql
    database\_host: localhost
    database\_port: null
    database\_name: jobeet
    database\_user: root
    database\_password: password
    # ...
```

现在你能让`Doctrine`连接到你的数据库了。你能通过在终端输入下面的命令来让`Doctrine`帮助你生成数据库：

    php app/console doctrine:database:create

## 模式 ##

为了能让`Doctrine`能够了解我们的对象，我们需要创建“元数据”文件来描述对象在数据库中的字段域。我们先在`src/Ibw/JobeetBundle/Resources/config`目录下创建一个`doctrine`目录。`doctribe`目录会包含三个文件：`Category.orm.yml`，`Job.orm.yml`和`Affiliate.orm.yml`。

```Yml
# src/Ibw/JobeetBundle/Resources/config/doctrine/Category.orm.yml
Ibw\JobeetBundle\Entity\Category:
    type: entity
    table: category
    id:
        id:
            type: integer
            generator: { strategy: AUTO }
    fields:
        name:
            type: string
            length: 255
            unique: true
    oneToMany:
        jobs:
            targetEntity: Job
            mappedBy: category
    manyToMany:
        affiliates:
            targetEntity: Affiliate
            mappedBy: categories
```

```Yml
# src/Ibw/JobeetBundle/Resources/config/doctrine/Job.orm.yml
Ibw\JobeetBundle\Entity\Job:
    type: entity
    table: job
    id:
        id:
            type: integer
            generator: { strategy: AUTO }
    fields:
        type:
            type: string
            length: 255
            nullable: true
        company:
            type: string
            length: 255
        logo:
            type: string
            length: 255
            nullable: true
        url:
            type: string
            length: 255
            nullable: true
        position:
            type: string
            length: 255
        location:
            type: string
            length: 255
        description:
            type: text
        how_to_apply:
            type: text
        token:
            type: string
            length: 255
            unique: true
        is_public:
            type: boolean
            nullable: true
        is_activated:
            type: boolean
            nullable: true
        email:
            type: string
            length: 255
        expires_at:
            type: datetime
        created_at:
            type: datetime
        updated_at:
            type: datetime
            nullable: true
    manyToOne:
        category:
            targetEntity: Category
            inversedBy: jobs
            joinColumn:
                name: category_id
                referencedColumnName: id
    lifecycleCallbacks:
        prePersist: [ setCreatedAtValue ]
        preUpdate: [ setUpdatedAtValue ]
```

```Yml
# src/Ibw/JobeetBundle/Resources/config/doctrine/Affiliate.orm.yml
Ibw\JobeetBundle\Entity\Affiliate:
    type: entity
    table: affiliate
    id:
        id:
            type: integer
            generator: { strategy: AUTO }
    fields:
        url:
            type: string
            length: 255
        email:
            type: string
            length: 255
            unique: true
        token:
            type: string
            length: 255
        is_active:
            type: boolean
            nullable: true
        created_at:
            type: datetime
    manyToMany:
        categories:
            targetEntity: Category
            joinTable:
                name: category_affiliate
                joinColumns:
                    affiliate_id:
                        referencedColumnName: id
                inverseJoinColumns:
                    category_id:
                        referencedColumnName: id
    lifecycleCallbacks:
        prePersist: [ setCreatedAtValue ]
```

## ORM ##

现在在终端输入下面的命令，`Doctrine`能够生成我们定义好的对应的类了：

    php app/console doctrine:generate:entities IbwJobeetBundle

现在你可以在`src/Ibw/JobeetBundle`目录下看到一个`Entity`目录，在`Entity`目录中你可以看到新生成的文件：`Category.php`，`Job.php`和`Affiliate.php`。打开`Job.php`文件，把`created_at`和`updated_at`的值设置如下：

```PHP
// src/Ibw/JobeetBundle/Entity/Job.php
// ...
 
    /**
     * @ORM\PrePersist
     */
    public function setCreatedAtValue()
    {
        if(!$this->getCreatedAt()) {
            $this->created_at = new \DateTime();
        }
    }
 
    /**
     * @ORM\PreUpdate
     */
    public function setUpdatedAtValue()
    {
        $this->updated_at = new \DateTime();
    }
```

对`Affiliate`类中的`created_at`进行同样的修改：

```PHP
// src/Ibw/JobeetBundle/Entity/Affiliate.php
// ...
 
    /**
     * @ORM\PrePersist
     */
    public function setCreatedAtValue()
    {
        $this->created_at = new \DateTime();
    }
 
// ...
```

上面做的修改会让`Doctrine`在保存对象或者更新对象的时候更新`created_at`和`updated_at`的值。这些`Doctrine`行为被定义在`Affiliate.orm.yml`和`Job.orm.yml`文件的下面。接下来我们让Doctrine帮助我们生成数据表，输入下面的命令：

    php app/console doctrine:schema:update --force

> 这个任务（task）只能在开发环境（development）中使用。生产环境（production）中更新你的数据库，看以查看[Doctrine migrations](http://symfony.com/doc/current/bundles/DoctrineMigrationsBundle/index.html)。

现在数据表已经创建好了，但数据表中还没有任何的数据。对于一个Web应用，它们往往具有三类数据：`初始数据`（应用运行所需要的数据，在`Jobeet`中我们需要初始用的分类（categories）数据和管理员（admin）数据），`测试数据`（应用程序测试用的数据）和`用户数据`（用户在使用应用的时候创建的）

为了让我们的数据库中能有些`初始数据`，我们使用[DoctrineFixturesBundle](http://symfony.com/doc/current/bundles/DoctrineFixturesBundle/index.html)。我们来按下面的步骤来安装`DoctrineFixturesBundle`：

### 1、添加下面的代码到`composer.json`中的`require`区块中 ###

```Javascript
// composer.json
// ...
    "require": {
        // ...
        "doctrine/doctrine-fixtures-bundle": "dev-master",
        "doctrine/data-fixtures": "dev-master"
    },
 
// ...
```

### 2、更新`vender`库 ###

    php composer.phar update

### 3、在`app/AppKernel.php`中注册`DoctrineFixturesBundle` ###

```PHP
// app/AppKernel.php
// ...
 
public function registerBundles()
{
    $bundles = array(
        // ...
        new Doctrine\Bundle\FixturesBundle\DoctrineFixturesBundle()
    );
 
    // ...
}
```

现在我们已经准备好了，我们在`src/Ibw/JobeetBundle/DataFixtures/ORM`目录下生成一些新类来加载数据：

```PHP
// src/Ibw/JobeetBundle/DataFixtures/ORM/LoadCategoryData.php
<?php
namespace Ibw\JobeetBundle\DataFixtures\ORM;
 
use Doctrine\Common\Persistence\ObjectManager;
use Doctrine\Common\DataFixtures\AbstractFixture;
use Doctrine\Common\DataFixtures\OrderedFixtureInterface;
use Ibw\JobeetBundle\Entity\Category;
 
class LoadCategoryData extends AbstractFixture implements OrderedFixtureInterface
{
    public function load(ObjectManager $em)
    {
        $design = new Category();
        $design->setName('Design');
 
        $programming = new Category();
        $programming->setName('Programming');
 
        $manager = new Category();
        $manager->setName('Manager');
 
        $administrator = new Category();
        $administrator->setName('Administrator');
 
        $em->persist($design);
        $em->persist($programming);
        $em->persist($manager);
        $em->persist($administrator);
        $em->flush();
 
        $this->addReference('category-design', $design);
        $this->addReference('category-programming', $programming);
        $this->addReference('category-manager', $manager);
        $this->addReference('category-administrator', $administrator);
    }
 
    public function getOrder()
    {
        return 1; // the order in which fixtures will be loaded
    }
}
```

```PHP
// src/Ibw/JobeetBundle/DataFixtures/ORM/LoadJobData.php
<?php
namespace Ibw\JobeetBundle\DataFixtures\ORM;
 
use Doctrine\Common\Persistence\ObjectManager;
use Doctrine\Common\DataFixtures\AbstractFixture;
use Doctrine\Common\DataFixtures\OrderedFixtureInterface;
use Ibw\JobeetBundle\Entity\Job;
 
class LoadJobData extends AbstractFixture implements OrderedFixtureInterface
{
    public function load(ObjectManager $em)
    {
         $job_sensio_labs = new Job();
         $job_sensio_labs->setCategory($em->merge($this->getReference('category-programming')));
         $job_sensio_labs->setType('full-time');
         $job_sensio_labs->setCompany('Sensio Labs');
         $job_sensio_labs->setLogo('sensio-labs.gif');
         $job_sensio_labs->setUrl('http://www.sensiolabs.com/');
         $job_sensio_labs->setPosition('Web Developer');
         $job_sensio_labs->setLocation('Paris, France');
         $job_sensio_labs->setDescription('You\'ve already developed websites with symfony and you want to work with Open-Source technologies. You have a minimum of 3 years experience in web development with PHP or Java and you wish to participate to development of Web 2.0 sites using the best frameworks available.');
         $job_sensio_labs->setHowToApply('Send your resume to fabien.potencier [at] sensio.com');
         $job_sensio_labs->setIsPublic(true);
         $job_sensio_labs->setIsActivated(true);
         $job_sensio_labs->setToken('job_sensio_labs');
         $job_sensio_labs->setEmail('job@example.com');
         $job_sensio_labs->setExpiresAt(new \DateTime('+30 days'));
         $job_extreme_sensio = new Job();
         $job_extreme_sensio->setCategory($em->merge($this->getReference('category-design')));
         $job_extreme_sensio->setType('part-time');
         $job_extreme_sensio->setCompany('Extreme Sensio');
         $job_extreme_sensio->setLogo('extreme-sensio.gif');
         $job_extreme_sensio->setUrl('http://www.extreme-sensio.com/');
         $job_extreme_sensio->setPosition('Web Designer');
         $job_extreme_sensio->setLocation('Paris, France');
         $job_extreme_sensio->setDescription('Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in.');
         $job_extreme_sensio->setHowToApply('Send your resume to fabien.potencier [at] sensio.com');
         $job_extreme_sensio->setIsPublic(true);
         $job_extreme_sensio->setIsActivated(true);
         $job_extreme_sensio->setToken('job_extreme_sensio');
         $job_extreme_sensio->setEmail('job@example.com');
         $job_extreme_sensio->setExpiresAt(new \DateTime('+30 days'));
 
         $em->persist($job_sensio_labs);
         $em->persist($job_extreme_sensio);
         $em->flush();
    }
 
    public function getOrder()
    {
        return 2; // the order in which fixtures will be loaded
    }
}
```

现在我们已经把fixtures写好了，我们可以使用下面的命令来加载这些数据：

    php app/console doctrine:fixtures:load

现在你可以去数据库看看，你可以看到数据表中已经有数据了。

## 查看浏览器 ##

运行下面的命令，它将会创建一个新的控制器（controller）`src/Ibw/JobeetBundle/Controllers/JobController.php`，这个控制器还带有CURD动作（Action）。
    
    php app/console doctrine:generate:crud --entity=IbwJobeetBundle:Job --route-prefix=ibw_job --with-write --format=yml

运行命令之后，选择默认的答案（answer）即可，这可能需要被要求做一些配置。

为了能在浏览器中浏览，我们需要导入（import）新生成的路由（routes）文件`src/Ibw/JobeetBundle/Resources/config/routing/job.yml`到主路由文件中：

```Yml
# src/Ibw/JobeetBundle/Resources/config/routing.yml
IbwJobeetBundle_job:
        resource: "@IbwJobeetBundle/Resources/config/routing/job.yml"
        prefix:   /job
 
# ...
```

同时我们需要在`Category`类中添加`__toString()`方法，这个能让`job`表单能够下拉选择类别（category）：

```PHP
// src/Ibw/JobeetBundle/Entity/Category.php
// ...
 
public function __toString()
{
    return $this->getName() ? $this->getName() : "";
}
 
// ...
```

清除缓存：

    php app/console cache:clear --env=dev
    php app/console cache:clear --env=prod

现在你可以在浏览器中测试一下了：`http://jobeet.local/job/`，或者在开发环境中的`http://jobeet.local/app_dev.php/job/`。

![](imgs/03-02.png)

现在你能够添加或者编辑职位（jobs）了。你可以试着让必填项留空或者输入无效的数据。是的，你可以看到Symfony已经创建了基本的验证规则，这些都有是按照数据表来定义的。

今天就这些了。我们勉强地写了一些PHP代码，同时也有了一个职位（job）模型和职位模块（module）。明天我们就来熟悉一下控制器（controller）和视图（view）吧，明天见。

# 许可证 #

如果您需要转载的话，请尊重原作者的知识产权，您可以通过把如下链接放到您转载文章中的头部或者尾部，谢谢。

原文链接：<http://www.intelligentbee.com/blog/2013/08/09/symfony2-jobeet-day-3-the-data-model/>

您可以在以下链接查看该许可证的全文：

![](../imgs/license.png)

<http://creativecommons.org/licenses/by-nc/3.0/legalcode>