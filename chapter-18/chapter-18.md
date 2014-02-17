# 第十八天：Ajax #

*这一系列文章来源于Fabien Potencier，基于Symfony1.4编写的[Jobeet Tutirual](http://symfony.com/legacy/doc/jobeet?orm=Doctrine)。

在昨天的教程中，我们使用了Zend Lucene库为Jobeet实现了一个功能强大的搜索引擎。而在今天的教程中，我们来提高搜索引擎的响应性，并且发挥Ajax的优点，把搜索变成实时响应的。现在我们需要给表单添加Javascript功能，但又不能往表单标签元素中嵌入Javascript，那么我们将会使用[unobtrusive JavaScript](http://en.wikipedia.org/wiki/Unobtrusive_JavaScript)来实现这个功能。在客户端代码中，如HTML，CSS和Javascript，使用unobtrusive JavaScript能够更好的遵循代码分离的概念。

## 安装jQuery ##

到[jQuery](http://jquery.com/)官方网站下载最新版本的jQuery，然后把*.js*文件放在*src/Ibw/JobeetBundle/Resources/public/js/*目录下。

把*.js*文件放在*js*目录下之后，运行下面的命令安装到Symfony资源中：

    php app/console assets:install web --symlink

## 包含jQuery ##

我们需要在所有的页面中使用到jQuery，因此我们需要修改layout文件：

```HTML
<!-- src/Ibw/JobeetBundle/Resources/views/layout.html.twig -->
<!-- ... -->
    {% block javascripts %}
        <script type="text/javascript" src="{{ asset('bundles/ibwjobeet/js/jquery-2.0.3.min.js') }}"></script>
    {% endblock %}
<!-- ... -->
```

## 添加行为 ##

一个实时的搜索功能就意味着每当用户在搜索框中输入字符时，系统就会触发一个到服务器的请求。服务器会返回请求的数据，并用这些数据更新页面中的一部分区域而不用重新刷新整个页面来加载这些请求数据。jQuery背后的原则是：等待页面加载完成之后才给DOM添加行为，而不是直接在HTML标签上添加on*()行为。如果你禁用了浏览器的Javascript支持，那么任何行为都不会被注册，尽管如此，表单还是能够和之前一样正常地工作。

我们的第一步是得到用户在输入框中输入的关键字：

```JAVASCRIPT
$('#search_keywords').keyup(function(key)
{
    if (this.value.length >= 3 || this.value == '')
    {
        // do something
    }
});
```

> 现在请不要把代码修改到文件中，因为我们还要对它进行大幅度的修改。在下面的教程中，我们会把最终的Javascript代码会在添加到layout文件里。

每当用户输入关键字时，jQuery就会执行上面代码定义的异步函数（anonymous function），但只有用户输入的字符超过三个或者不输入任何字符时才会执行里面的逻辑。

一种简单实现Ajax请求的方式就是使用DOM元素上的load()方法：

```JAVASCRIPT
$('#search_keywords').keyup(function(key)
{
    if (this.value.length >= 3 || this.value == '')
    {
        $('#jobs').load(
            $(this).parents('form').attr('action'), { query: this.value + '*' }
        );
    }
});
```

为了处理AJAX请求，我们将会在下面的部分中修改action。

最后但同样重要的是，如果Javascript可以使用，我们想要把search按钮给移除掉：

```JAVASCRIPT
$('.search input[type="submit"]').hide();
```

## 用户反馈 ##

在执行AJAX调用时，页面有时候并不会马上进行更新，浏览器会等待服务器把数据返还回来后才进行页面的更新。与此同时，我们需要做一些反馈给用户，告诉他们数据正在获取中。

在AJAX执行期间，给用户显示一个loading图标的用法已经成为了一种惯例。我们在layout中添加loader图片，并默认把它设成隐藏的：

```HTML
<!-- src/Ibw/JobeetBundle/Resources/views/layout.html.twig -->
<!-- ... -->
    <div class="search">
        <h2>Ask for a job</h2>
        <form action="{{ path('ibw_job_search') }}" method="get">
            <input type="text" name="query" value="{{ app.request.get('query') }}" id="search_keywords" />
            <input type="submit" value="search" />
            <img id="loader" src="{{ asset('bundles/ibwjobeet/images/loader.gif') }}" style="vertical-align: middle; display: none" />
            <div class="help">
                Enter some keywords (city, country, position, ...)
            </div>
        </form>
    </div>
<!-- ... -->
```

我们完成功能所需的HTML代码都已经有了，现在我们创建*search.js*文件，然后把之前我们解释过的代码添加到里面去：

```JAVASCRIPT
// src/Ibw/JobeetBundle/Resources/public/js/search.js
$(document).ready(function()
{
    $('.search input[type="submit"]').hide();
 
    $('#search_keywords').keyup(function(key)
    {
        if(this.value.length >= 3 || this.value == '') {
            $('#loader').show();
            $('#jobs').load(
                $(this).parent('form').attr('action'),
                { query: this.value ? this.value + '*' : this.value },
                function() {
                    $('#loader').hide();
                }
            );
        }
    });
});
```

运行下面的代码安装资源：

    php app/console assets:install web --symlink

我们同样需要修改layout文件来包含search.js：

```HTML
<!-- src/Ibw/JobeetBundle/Resources/views/layout.html.twig -->
<!-- ... -->
    {% block javascripts %}
        <script type="text/javascript" src="{{ asset('bundles/ibwjobeet/js/jquery-2.0.3.min.js') }}"></script>
        <script type="text/javascript" src="{{ asset('bundles/ibwjobeet/js/search.js') }}"></script>
    {% endblock %}
<!-- ... -->
```

## AJAX和Action ##

如果Javascript未被禁用，jQuery就会得到用户输入的关键字，然后请求*searchAction()*。如果Javascript被禁用了，用户可以点击search按钮来请求*searchAction()*。

那么*searchAction()*需要判断请求是否是通过AJAX发起的。不管什么时候调用AJAX，request对象的*isXmlHttpRequest()*方法都返回true。

```PHP
// src/Ibw/JobeetBundle/Controller/JobController.php
use Symfony\Component\HttpFoundation\Response;
 
class JobController extends Controller
{  
    // ...
 
    public function searchAction(Request $request)
    {
        $em = $this->getDoctrine()->getManager();
        $query = $this->getRequest()->get('query');
 
        if(!$query) {
            if(!$request->isXmlHttpRequest()) {
                return $this->redirect($this->generateUrl('ibw_job'));
            } else {
                return new Response('No results.');
            }
        }
 
        $jobs = $em->getRepository('IbwJobeetBundle:Job')->getForLuceneQuery($query);
 
        if($request->isXmlHttpRequest()) {
 
            return $this->render('IbwJobeetBundle:Job:list.html.twig', array('jobs' => $jobs));
        }
 
        return $this->render('IbwJobeetBundle:Job:search.html.twig', array('jobs' => $jobs));
    }
}
```

如果搜索的结果不存在，我们需要在页面上显示出一条信息，而不是空白的页面。

我们来返回一个简单的字符串：

```PHP
// src/Ibw/JobeetBundle/Controller/JobController.php
public function searchAction(Request $request)
{
    $em = $this->getDoctrine()->getManager();
    $query = $this->getRequest()->get('query');

    if(!$query) {
        if(!$request->isXmlHttpRequest()) {
            return $this->redirect($this->generateUrl('ibw_job'));
        } else {
            return new Response('No results.');
        }
    }

    $jobs = $em->getRepository('IbwJobeetBundle:Job')->getForLuceneQuery($query);

    if($request->isXmlHttpRequest()) {
        if('*' == $query || !$jobs || $query == '') {
            return new Response('No results.');
        }

        return $this->render('IbwJobeetBundle:Job:list.html.twig', array('jobs' => $jobs));
    }

    return $this->render('IbwJobeetBundle:Job:search.html.twig', array('jobs' => $jobs));
}
```

## 测试AJAX ##

因为Symfony不能够模拟AJAX请求，所以当测试AJAX调用时，我们需要给它一些帮助。这主要是意味着你需要手动在请求的header中添加X-Requested-With：

```PHP
// src/Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php
class JobControllerTest extends WebTestCase
{
    // ...
 
    public function testSearch()
    {
        $client = static::createClient();
 
        $crawler = $client->request('GET', '/job/search');
        $this->assertEquals('Ibw\JobeetBundle\Controller\JobController::searchAction', $client->getRequest()->attributes->get('_controller'));
 
        $crawler = $client->request('GET', '/job/search?query=sens*', array(), array(), array(
            'X-Requested-With' => 'XMLHttpRequest',
        ));
        $this->assertTrue($crawler->filter('tr')->count()== 2);
    }
}
```

在第十七天中，我们使用Zend Lucene库实现了搜索引擎。而今天我们使用jQuery让搜索引擎的响应性变得更好。Symfony提供了一些列基础工具，它使得构建一个MVC应用变得简单，而且它还能和其它第三方组件很好地集成在一起。总之，我们可以尝试着使用最佳的工具来服务我们的工作，提高工作效率。

# 许可证 #

如果您需要转载的话，请尊重原作者的知识产权，您可以通过把如下链接放到您转载文章中的头部或者尾部，谢谢。

原文链接：<http://www.intelligentbee.com/blog/2013/09/03/symfony2-jobeet-day-18-ajax/>

您可以在以下链接查看该许可证的全文：

![](../imgs/license.png)

<http://creativecommons.org/licenses/by-nc/3.0/legalcode>