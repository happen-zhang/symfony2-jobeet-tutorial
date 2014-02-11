# 第十三天：安全性 #

*这一系列文章来源于Fabien Potencier，基于Symfony1.4编写的[Jobeet Tutirual](http://symfony.com/legacy/doc/jobeet?orm=Doctrine)。

## 让应用程序更加安全 ##

应用程序的安全性是一个两步验证的过程，它的目标是阻止用户访问他/她不应该访问到的资源。实现安全性的第一步是*认证（authentication）*，安全认证系统会取得用户提交的一些标识符，然后通过这个标识符分辨出用户的身份。一旦系统分辨出用户的身份之后，下一步就是系统对用户进行*授权（authorization）*，这一步将决定用户能够访问哪些给定的资源（系统会检查用户是否有权限进行这个操作）。我们可以通过配置*app/config*目录下的*security.yml*文件来对应用程序的安全组件进行配置。为了实现我们应用的安全性，我们修改*scurity.yml*文件：

```YAML
# app/config/security.yml
security:
    role_hierarchy:
        ROLE_ADMIN:       ROLE_USER
        ROLE_SUPER_ADMIN: [ROLE_USER, ROLE_ADMIN, ROLE_ALLOWED_TO_SWITCH]
 
    firewalls:
        dev:
            pattern:  ^/(_(profiler|wdt)|css|images|js)/
            security: false
 
        secured_area:
            pattern:    ^/
            anonymous: ~
            form_login:
                login_path:  /login
                check_path:  /login_check
                default_target_path: ibw_jobeet_homepage
 
    access_control:
        - { path: ^/admin, roles: ROLE_ADMIN }
 
    providers:
        in_memory:
            memory:
                users:
                    admin: { password: adminpass, roles: 'ROLE_ADMIN' }
 
    encoders:
        Symfony\Component\Security\Core\User\User: plaintext
```

上面的配置会对网站的*/admin*部分（所有以*/admin*作为开头的url）进行安全性保护，并且只有*ROLE_ADMIN*角色的用户才能访问到*/admin*（可以查看*security.yml*中的*access_controller*部分）。在这个例子中，*admin*用户被定义在*security.yml*中（*providers*部分），而且*admin*用户的密码没有被编码（即没有经过加密算法处理过）（*encoders*部分）。对于用户认证，我们需要使用一个传统的登录表单，我们需要去实现它。首先我们需要创建两个路由：一个用来显示登录表单（例如：/login），另一个则用来处理提的交登录表单（例如：/login_check）：

```YAML
# src/Ibw/JobeetBundle/Resources/config/routing.yml
login:
    pattern:   /login
    defaults:  { _controller: IbwJobeetBundle:Default:login }
login_check:
    pattern:   /login_check
 
# ...
```

我们不需要为*/login_check* URL实现一个控制器，因为防火墙（firewall）会自动捕捉并处理所有提交到*/login_check* URL的表单。我们需要做的就是创建一个路由（*/login_check*），我们会在登录表单的模板中使用它，这样才能把登录表单提交到*/login_check*。

下一步我们来创建显示登录表单的*action*：

```PHP
// src/Ibw/JobeetBundle/Controller/DefaultController.php
namespace Ibw\JobeetBundle\Controller;
 
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\Security\Core\SecurityContext;
 
class DefaultController extends Controller
{
    // ...
 
    public function loginAction()
    {
        $request = $this->getRequest();
        $session = $request->getSession();
 
        // get the login error if there is one
        if ($request->attributes->has(SecurityContext::AUTHENTICATION_ERROR)) {
            $error = $request->attributes->get(SecurityContext::AUTHENTICATION_ERROR);
        } else {
            $error = $session->get(SecurityContext::AUTHENTICATION_ERROR);
            $session->remove(SecurityContext::AUTHENTICATION_ERROR);
        }
 
        return $this->render('IbwJobeetBundle:Default:login.html.twig', array(
            // last username entered by the user
            'last_username' => $session->get(SecurityContext::LAST_USERNAME),
            'error'         => $error,
        ));
    }
}
```

当用户提交了表单，安全系统会自动处理提交的表单。如果用户提交了一个无效的用户名或者密码，那么这个操作就会从系统中读取出提交表单的错误信息，并把这个错误信息反馈给用户。我们的工作仅仅只有显示出一个登录表单和可能出现的登录错误信息，而安全系统则要通过用户提交上来的用户名和密码对用户进行验证。

最后，我们来创建模板：

```HTML
<!-- src/Ibw/JobeetBundle/Resources/views/Default/login.html.twig -->
{% if error %}
    <div>{{ error.message }}</div>
{% endif %}
 
<form action="{{ path('login_check') }}" method="post">
    <label for="username">Username:</label>
    <input type="text" id="username" name="_username" value="{{ last_username }}" />
 
    <label for="password">Password:</label>
    <input type="password" id="password" name="_password" />
 
    <button type="submit">login</button>
</form>
```

现在，如果你去访问<http://jobeet.local/app_dev.php/admin/dashboard> URL，你会看到一个登录表单，你需要输入定义在*security.yml*中的用户名和密码（*admin/adminpass*）才能进入*admin*部分。

## User Providers ##

在认证过程中，用户提交了一组登录凭证（通常是用户名和密码）。认证系统的工作就是在用户列表中逐个匹配登录凭证。那么用户列表从何而来呢？

在Symfony2中，用户列表可以存放在任何地方，一个配置文件，一个数据库表，一个web service接口，或者其它你能想得到的地方。为认证系统提供一个或者多个用户的类或接口就叫做“*user provider*”。Symfony2 自带了两种最常见的*user provider*：一种是使用配置文件，另一种是使用数据库表。

在上面的例子中，我们把用户列表存放在配置文件里：

```YAML
# app/config/security.yml
# ...
 
providers:
    in_memory:
        memory:
            users:
                admin: { password: adminpass, roles: 'ROLE_ADMIN' }
 
# ...
```

但我们更多时候是需要把用户信息存放在数据表中。为了做到这点，我们需要在数据库中创建一个*user*表。首先我们来为*user*表创建orm文件：

```YAML
# src/Ibw/JobeetBundle/Resources/config/doctrine/User.orm.yml
Ibw\JobeetBundle\Entity\User:
    type: entity
    table: user
    id:
        id:
            type: integer
            generator: { strategy: AUTO }
    fields:
        username:
            type: string
            length: 255
        password:
            type: string
            length: 255
```

现在运行命令生成*User*实体类：

    php app/console doctrine:generate:entities IbwJobeetBundle

然后更新数据库：

    php app/console doctrine:schema:update --force

这里需要为用户类（User）实现*UserInterface*接口，这就意味着“用户”可是任何的东西，只要它实现了*UserInterface*接口。打开并修改*User.php*：

```PHP
// src/Ibw/JobeetBundle/Entity/User.php
namespace Ibw\JobeetBundle\Entity;
 
use Symfony\Component\Security\Core\User\UserInterface;
use Doctrine\ORM\Mapping as ORM;
 
/**
 * User
 */
class User implements UserInterface
{
    /**
     * @var integer
     */
    private $id;
 
    /**
     * @var string
     */
    private $username;
 
    /**
     * @var string
     */
    private $password;
 
    /**
     * Get id
     *
     * @return integer 
     */
    public function getId()
    {
        return $this->id;
    }
 
    /**
     * Set username
     *
     * @param string $username
     * @return User
     */
    public function setUsername($username)
    {
        $this->username = $username;
 
    }
 
    /**
     * Get username
     *
     * @return string 
     */
    public function getUsername()
    {
        return $this->username;
    }
 
    /**
     * Set password
     *
     * @param string $password
     * @return User
     */
    public function setPassword($password)
    {
        $this->password = $password;
 
    }
 
    /**
     * Get password
     *
     * @return string 
     */
    public function getPassword()
    {
        return $this->password;
    }
 
    public function getRoles()
    {
        return array('ROLE_ADMIN');
    }
 
    public function getSalt()
    {
        return null;
    }
 
    public function eraseCredentials()
    {
 
    }
 
    public function equals(User $user)
    {
        return $user->getUsername() == $this->getUsername();
    }        
}
```

我们需要为*User*类实现*UserInterface*接口的抽象方法：*getRoles*，*getSalt*，*eraseCredentials*和*equals*。

接下来配置一个*user provider*实体，它指向*User*类：

```YAML
# app/config/security.yml
...
 
    providers:
        main:
            entity: { class: Ibw\JobeetBundle\Entity\User, property: username }
 
    encoders:
        Ibw\JobeetBundle\Entity\User: sha512
```

我们同样改变了*encoder*部分，这样就能够使用*sha512*算法对*User*类的*password*属性进行加密了。

所有准备都已经做好了，现在我们需要先创建一个用户。为了做到这点，我们会创建一个新的Symfony命令：

```PHP
// src/Ibw/JobeetBundle/Command/JobeetUsersCommand.php
namespace Ibw\JobeetBundle\Command;
 
use Symfony\Bundle\FrameworkBundle\Command\ContainerAwareCommand;
use Symfony\Component\Console\Input\InputArgument;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Input\InputOption;
use Symfony\Component\Console\Output\OutputInterface;
use Ibw\JobeetBundle\Entity\User;
 
class JobeetUsersCommand extends ContainerAwareCommand
{
    protected function configure()
    {
        $this
            ->setName('ibw:jobeet:users')
            ->setDescription('Add Jobeet users')
            ->addArgument('username', InputArgument::REQUIRED, 'The username')
            ->addArgument('password', InputArgument::REQUIRED, 'The password')
        ;
    }
 
    protected function execute(InputInterface $input, OutputInterface $output)
    {
        $username = $input->getArgument('username');
        $password = $input->getArgument('password');
 
        $em = $this->getContainer()->get('doctrine')->getManager();
 
        $user = new User();
        $user->setUsername($username);
        // encode the password
        $factory = $this->getContainer()->get('security.encoder_factory');
        $encoder = $factory->getEncoder($user);
        $encodedPassword = $encoder->encodePassword($password, $user->getSalt());
        $user->setPassword($encodedPassword);
        $em->persist($user);
        $em->flush();
 
        $output->writeln(sprintf('Added %s user with password %s', $username, $password));
    }
}
```

使用命令创建第一个用户：

    php app/console ibw:jobeet:users admin admin

上面的命令会创建一个用户名和密码都是“admin”的用户。你可以使用它登录*admin*部分。

## 登出（Logout） ##

防火墙（firewall）能够自动处理登出。我们需要做的就是激活*logout*配置参数：

```YAML
# app/config/security.yml
security:
    firewalls:
        # ...
        secured_area:
            # ...
            logout:
                path:   /logout
                target: /
    # ...
```

你不需要为*/logout* URL实现控制器，防火墙（firewall）会自行处理。我们来创建*/logout*路由：

```YAML
# src/Ibw/JobeetBundle/Resources/config/routing.yml
# ...
 
logout:
    pattern:   /logout
 
# ...
```

一旦配置完成了，访问*/logout*后，当前用户就会被注销认证（un-authenticate）了。然后用户会被重定向到首页（这个值定义在*target*参数中）。

剩下没有做的事情就是为*admin*部分添加*logout*链接。我们会重载*SonataAdminBundle*的*user_block.html.twig*。在*app/Resources/SonataAdminBundle/views/Core*目录下创建*user_block.html.twig*文件：

```HTML
<!-- app/Resources/SonataAdminBundle/views/Core/user_block.html.twig -->
{% block user_block %}<a href="{{ path('logout') }}">Logout</a>{% endblock%}
```

现在，你访问*admin*部分（请先清除缓存），你会被要求填写用户名和密码，进入之后你可以在右上角看到一个*logout*链接。

## 用户Session ##

Symfony2提供了一个非常好用的session对象，用户的每个不同请求都可以访问到它里面保存的信息。Symfony2默认是使用原生的PHP session把信息保存到一个cookie中。

你可以在控制器中方便地取出session里保存的信息：

```PHP
$session = $this->getRequest()->getSession();
 
// store an attribute for reuse during a later user request
$session->set('foo', 'bar');
 
// in another controller for another request
$foo = $session->get('foo');
```

可惜的是，在我们Jobeet的用户stories中（第二天），并没有需要在用户session中保存信息的需求。所以我们来改改需求：为了方便*job*信息的浏览，我们会在菜单栏上显示用户刚刚浏览过的三条*job*信息的链接。

当用户访问一个*job*页面时，那么这个*job*对象会被添加到用户的浏览历史中，并且把它保存到session：

```PHP
// ...
 
public function showAction($id)
{
    $em = $this->getDoctrine()->getManager();
 
    $entity = $em->getRepository('IbwJobeetBundle:Job')->getActiveJob($id);
 
    if (!$entity) {
        throw $this->createNotFoundException('Unable to find Job entity.');
    }
 
    $session = $this->getRequest()->getSession();
 
    // fetch jobs already stored in the job history
    $jobs = $session->get('job_history', array());
 
    // store the job as an array so we can put it in the session and avoid entity serialize errors
    $job = array('id' => $entity->getId(), 'position' =>$entity->getPosition(), 'company' => $entity->getCompany(), 'companyslug' => $entity->getCompanySlug(), 'locationslug' => $entity->getLocationSlug(), 'positionslug' => $entity->getPositionSlug());
 
    if (!in_array($job, $jobs)) {
        // add the current job at the beginning of the array
        array_unshift($jobs, $job);
 
        // store the new job history back into the session
        $session->set('job_history', array_slice($jobs, 0, 3));
    }
 
    $deleteForm = $this->createDeleteForm($id);
 
    return $this->render('IbwJobeetBundle:Job:show.html.twig', array(
        'entity'      => $entity,
        'delete_form' => $deleteForm->createView(),
    ));
}
```

在*layout*文件的*#content* div标签之前添加下面的代码：

```HTML
<!-- src/Ibw/JobeetBundle/Resources/views/layout.html.twig -->
<!-- ... -->
 
<div id="job_history">
    Recent viewed jobs:
    <ul>
        {% for job in app.session.get('job_history') %}
            <li>
                <a href="{{ path('ibw_job_show', { 'id': job.id, 'company': job.companyslug, 'location': job.locationslug, 'position': job.positionslug }) }}">{{ job.position }} - {{ job.company }}</a>
            </li>
        {% endfor %}
    </ul>
</div>
 
<div id="content">
 
<!-- ... -->
```

## flash信息 ##

*flash*信息是存放在用户session中的简短信息（通常是用来作为反馈给用户的信息），它通常用在下一个请求中显示信息。它对表单很有用：当你想要重定向并请求一个新页面的同时，显示出一些指定的信息。我们已经在项目的发布*job*需求中使用过*flash*信息了：

```PHP
// src/Ibw/JobeetBundle/Controller/JobController.php
// ...
 
public function publishAction($token)
{
    // ...
 
    $this->get('session')->getFlashBag()->add('notice', 'Your job is now online for 30 days.');
 
    // ...
}
```

*getFlashBag()->add()*函数的第一个参数是用来标志*flash*的键，第二个参数是需要显示的信息。你可以为*flash*定义任何键名，但*notic*和*error*是最常用的两个。

为了显示*flash*信息，你需要在模板中把它们包含进来。我们已经在*layout.html.twig*中做过了：

```HTML
<!-- src/Ibw/JobeetBundle/Resources/views/layout.html.twig -->
<!-- ... -->
 
{% for flashMessage in app.session.flashbag.get('notice') %}
    <div>
        {{ flassMessage }}
    </div>
{% endfor %}
 
<!-- ... -->
```

# 许可证 #

如果您需要转载的话，请尊重原作者的知识产权，您可以通过把如下链接放到您转载文章中的头部或者尾部，谢谢。

原文链接：<http://www.intelligentbee.com/blog/2013/08/19/symfony2-jobeet-day-13-security/>

您可以在以下链接查看该许可证的全文：

![](../imgs/license.png)

<http://creativecommons.org/licenses/by-nc/3.0/legalcode>
