---
layout: page
name: Controller
---

# Description

UD allows the developer to save time. 90% of times json resources are enought to develop the most of the application.
Sometimes we need to implement some logic that is particulary complicated. In that case we can implement a controller.

A controller in UD has the same functionalities of a controller in a MVC framework.

A controller can implement the methods *getRequest* and *postRequest*. As you can imagine the *getRequest* method is called when the system receives a get request. :-)

For each controller we need to define the constant *CONTROLLER_NAME*. This consant sets the Router to sent calls to the specific controller. For example if *CONTROLLER_NAME* is set to *mycontroller*, each call to www.myapplication.com/mycontroller.html will be redirected to this controller.

The properties *get_validation_rules*, *get_filter_rules*, *post_validation_rules*, and *post_filter_rules* allow the user to set the falidation and flter rules for a GET or a POST request. As for a resource <a href="{{site.baseurl}}/docs/validation">validation</a> seection these rules are set in ordet to call the <a href="https://github.com/Wixel/GUMP">GUMP library</a>.

It is possible to overryde methods *check_authorization_get_request* and *check_authorization_post_request* in order to check if user is allowed to call this specific controller or not.

### Controller skeleton

{% highlight php %}
class MyController extends BaseController {

    const CONTROLLER_NAME = 'mycontroller';

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
        $this->templateFile = 'application';

        $menuresource = $this->applicationBuilder->getJsonloader()->loadResource( $this->pageStatus->getSessionWrapper()->getSessionGroup() );
        $this->menubuilder = new MenuJsonTemplate($this->applicationBuilder, $this->pageStatus);
        $this->menubuilder->setMenuStructure( $menuresource );

        // loading data from database using DAO objeects
        $bookDao  = new BookDao;
        $bookDao->setDBH($this->dbconnection->getDBH());
        $bookDao->setLogger($this->logger);
        $books = $bookDao->getByFields( [ 'authorid' => $this->getParameters['authorid'] ] );

        $this->menucontainer          = array( $this->menubuilder->createMenu() );
        $this->centralcontainer = array( new BookList( $books ) );
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

A controller need to extend **BaseController** class. 
It need to define **CONTROLLER_NAME** constant and it need to overwrite **getRequest** method.

