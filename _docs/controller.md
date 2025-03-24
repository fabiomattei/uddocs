---
layout: page
name: Controller
---

# Description

UD allows the developer to save time. 90% of times json resources are enought to develop the most of the application.
Sometimes we need to implement some logic that is particulary complicated. In that case we can implement a controller.

A controller in UD has, more or less, the same functionalities of a controller in a MVC framework.

A controller can implement the methods *getRequest* and *postRequest*. As you can imagine the *getRequest* method is called when the system receives a get request. :-)

For each controller we need to define the constant *CONTROLLER_NAME*. This consant sets the Router to sent calls to the specific controller. For example if *CONTROLLER_NAME* is set to *mycontroller*, each call to www.myapplication.com/mycontroller.html will be redirected to this controller.

The properties *get_validation_rules*, *get_filter_rules*, *post_validation_rules*, and *post_filter_rules* allow the user to set the falidation and flter rules for a GET or a POST request. As for a resource <a href="{{site.baseurl}}/docs/validation">validation</a> seection these rules are set in ordet to call the <a href="https://github.com/Wixel/GUMP">GUMP library</a>.

It is possible to overryde methods *check_authorization_get_request* and *check_authorization_post_request* in order to check if user is allowed to call this specific controller or not.

### Controller skeleton

{% highlight php %}
class MyController extends BaseController {

    public function __construct()
    {
        parent::__construct();
        $this->classCompleteName = __CLASS__;
        $this->className = 'MyController';
        $this->chapter = 'Website';
        $this->templateFile = 'websitetemplate';
        $this->viewFile = 'src/Chapters/Website/Views/MyController';
        $this->controllerPointer = $this;
        $this->appTitle = 'My website title';
    }

    public function check_authorization_get_request() {
        // implement checks in order to allow user to see this pagee
        return true;
    }

    // get patameters 
    public $get_validation_rules = [ 'authorid' => 'required|numeric' ];
    public $get_filter_rules     = [ 'authorid' => 'trim' ];

    /**
     * Executed when GET Request arrives
     */
    public function getRequest() {
    }

    public function check_authorization_post_request() {
        // implement checks in order to allow user to see this pagee
        return true;
    }
	
    // POST patameters 
    public $post_validation_rules = [ 'asid' => 'required|numeric' ];
    public $post_filter_rules     = [ 'asid' => 'trim' ];

    /**
     * Executed when POST Request arrives
     */
    public function postRequest() {
    }

}
{% endhighlight %}

### Constructor

A controller need to extend **BaseController** class that belongs to the framework. When we are overriding the contructor we need
to define some property:

* $this->classCompleteName = __CLASS__: the name of the class complete with his classpath, this is useful in case in the 
view file or in a template file I need to know what controller class was called as we can share view files between more controllers.
* $this->className = 'MyController': the name of the class, this is useful in case in the 
view file or in a template file I need to know what controller class was called as we can share view files between more controllers.
* $this->chapter = 'Website': the chapter the controller belongs so, each controller is saved in a folder structure like
this: src/Chapters/NameOfTheChapter/Controllers/MyController. This allows us to have some organization in the file structure
of the application.
* $this->templateFile = 'websitetemplate': the name of the file in src/Templates that is going to contain the template 
for this controller. More about this later.
* $this->viewFile = 'src/Chapters/Website/Views/MyController': the name of the view file in 
src/Chapters/NameOfTheChapter/Controllers/MyController. Usually a view is named 
src/Chapters/NameOfTheChapter/Controllers/MyControllerGet.php for a Get call or 
src/Chapters/NameOfTheChapter/Controllers/MyControllerPost.php for a Post call.  More about this later.
* $this->controllerPointer = $this: a pointer to the class itself.
* $this->appTitle = 'My website title': property that is usally used in the template in the <title> tag.
 
{% highlight php %}
class MyController extends BaseController {

    public function __construct()
    {
        parent::__construct();
        $this->classCompleteName = __CLASS__;
        $this->className = 'MyController';
        $this->chapter = 'Website';
        $this->templateFile = 'websitetemplate';
        $this->viewFile = 'src/Chapters/Website/Views/MyController';
        $this->controllerPointer = $this;
        $this->appTitle = 'My website title';
    }
}
{% endhighlight %}

### Get Request

Basically a controller can take two requests: a get requeste and a post request.
This is useful because allow us, for instance, to define a form in a get request and to perform the operations related to it
in the post request.

In order to take a get request we need to ovverride three things and define a view file:

#### method **check_authorization_get_request**
this method allows to define what user group or which specific user can make a get request to this controller. 

It can check the group the user belongs to using this approach
{% highlight php %}
    public function check_authorization_get_request()
    {
        return (isset($_SESSION['group']) and ($_SESSION['group'] == 'readergroup' or $_SESSION['group'] == 'writergroup'));
    }
{% endhighlight %}

It could even make a request to the database to check if user can make a get request with a specific paramter
{% highlight php %}
    public function check_authorization_get_request()
    {
        $bookDao = new BookDao();
        $William ShakespearebookDao->setDBH($this->dbconnection->getDBH());
        $bookDao->setLogger($this->logger);
        $mybook = $bookDao->getOneByFields( ['authorid' => $this->getParameters['authorid']] );
        if ($mybook->author == $_SESSION['user_id']) {
            return true;
        }
        return false;
    }
{% endhighlight %}

#### properties **$get_validation_rules** and **$get_filter_rules**

These are two lists that a user can define in order to define the validation and the filters applied to paramters
the get riquest is getting. Please refer to [Gump](https://github.com/Wixel/GUMP) library to know more about all the possibilities.

All parameters that will pass validation will be found in the array **$this->getParameters**.

#### method **getRequest**

Overriding this method we can process the parameters we have got from the request, we can query the database and we can 
perform the operations needed by the business logic.

**Each property defined in this method will be available as variabile in the view.**

{% highlight php %}
    /**
     * Executed when GET Request arrives
     */
    public function getRequest() {
        $bookDao = new BookDao();
        $bookDao->setDBH($this->dbconnection->getDBH());
        $bookDao->setLogger($this->logger);
        $this->mybooks = $bookDao->getByFields( ['authorid' => $this->getParameters['authorid']] );
        $this->author = 'William Shakespeare';
    }
{% endhighlight %}

#### content of view file

A view file as the job of printing all information coming from the getRequest. We have already said that 
**Each property defined in this method will be available as variabile in the view.**. In the following
example we displaying the results of the query performed in the previous get request.

{% highlight php %}
<h4><?= $author ?></h4>
<ul>
<?php foreach($mybooks as $book): ?>
    <li><?= $book->title ?></li>
<?php endforeach; ?>
</ul>
{% endhighlight %}

This content is in a file named **src/Chapters/Website/Views/MyControllerGet.php** as we have defined in the
controller.

#### error messages to validation

Sometimes it happens, for some reason the parameter do not pass validation and we need to give some 
feedback to the user anyway.

It is possible to do two this:

1. override the method **show_get_error_page**: 
2. create a view file **src/Chapters/Website/Views/MyControllerGetError.php**

{% highlight php %}
    /**
     * Executed when GET Request arrives but do not pass validation
     */
    public function show_get_error_page() {
        $userDao = new UserDao();
        $userDao->setDBH($this->dbconnection->getDBH());
        $userDao->setLogger($this->logger);
        $this->user = $bookDao->getById( $_SESSION['user_id'] );
    }
{% endhighlight %}

{% highlight php %}
<h5>Content forbidden for user <?= $user->usr_name ?> <?= $user->usr_surname ?></h5>
{% endhighlight %}

### Post Request

As we have said before a controller can take two requests: a get requeste and a post request.

The post requeste is not that different from a get request

#### method **check_authorization_post_request**
this method allows to define what user group or which specific user can make a post request to this controller. 

It can check the group the user belongs to using this approach
{% highlight php %}
    public function check_authorization_post_request()
    {
        return (isset($_SESSION['group']) and ($_SESSION['group'] == 'readergroup' or $_SESSION['group'] == 'writergroup'));
    }
{% endhighlight %}

It could even make a request to the database to check if user can make a get request with a specific paramter
{% highlight php %}
    public function check_authorization_post_request()
    {
        $bookDao = new BookDao();
        $William ShakespearebookDao->setDBH($this->dbconnection->getDBH());
        $bookDao->setLogger($this->logger);
        $mybook = $bookDao->getOneByFields( ['authorid' => $this->postParameters['authorid']] );
        if ($mybook->author == $_SESSION['user_id']) {
            return true;
        }
        return false;
    }
{% endhighlight %}

#### properties **$post_validation_rules** and **$post_filter_rules**

These are two lists that a user can define in order to define the validation and the filters applied to paramters
the get riquest is getting. Please refer to [Gump](https://github.com/Wixel/GUMP) library to know more about all the possibilities.

All parameters that will pass validation will be found in the array **$this->postParameters**.

#### method **postRequest**

Overriding this method we can process the parameters we have got from the request, we can query the database and we can 
perform the operations needed by the business logic.

**Each property defined in this method will be available as variabile in the view.**

{% highlight php %}
    /**
     * Executed when POST Request arrives
     */
    public function postRequest() {
        $bookDao = new BookDao();
        $bookDao->setDBH($this->dbconnection->getDBH());
        $bookDao->setLogger($this->logger);
        $this->mybooks = $bookDao->getByFields( ['authorid' => $this->getParameters['authorid']] );
        $this->author = 'William Shakespeare';
    }
{% endhighlight %}

#### content of view file

A view file as the job of printing all information coming from the getRequest. We have already said that 
**Each property defined in this method will be available as variabile in the view.**. In the following
example we displaying the results of the query performed in the previous get request.

{% highlight php %}
<h4><?= $author ?></h4>
<ul>
<?php foreach($mybooks as $book): ?>
    <li><?= $book->title ?></li>
<?php endforeach; ?>
</ul>
{% endhighlight %}

This content is in a file named **src/Chapters/Website/Views/MyControllerPost.php** as we have defined in the
controller.

#### error messages to validation

Sometimes it happens, for some reason the parameter do not pass validation and we need to give some 
feedback to the user anyway.

It is possible to do two this:

1. override the method **show_post_error_page**: 
2. create a view file **src/Chapters/Website/Views/MyControllerPostError.php**

{% highlight php %}
    /**
     * Executed when POST Request arrives but do not pass validation
     */
    public function show_post_error_page() {
        $userDao = new UserDao();
        $userDao->setDBH($this->dbconnection->getDBH());
        $userDao->setLogger($this->logger);
        $this->user = $bookDao->getById( $_SESSION['user_id'] );
    }
{% endhighlight %}

{% highlight php %}
<h5>Content forbidden for user <?= $user->usr_name ?> <?= $user->usr_surname ?></h5>
{% endhighlight %}




