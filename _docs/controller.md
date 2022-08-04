---
layout: page
name: Controller
---

# Description

UD allows the developer to save time. 90% of times json resources are enought to develop the most of the application.
Sometimes we need to implement some logic that is particulary complicated. In that case we can implement a controller.

### Controller skeleton

{% highlight php %}
class MyController extends Controller {

    const CONTROLLER_NAME = 'mycontroller';

    public function check_authorization_get_request() {
        // implement checks in order to allow user to see this pagee
        return true;
    }

    // get patameters 
    public $get_validation_rules = array( 'authorid' => 'required|numeric' );
    public $get_filter_rules     = array( 'authorid' => 'trim' );

    /**
     * Executed when GET Request arrives
     */
    public function getRequest() {
        $this->applicationBuilder->getJsonloader()->loadIndex();
        $dbh = $this->pageStatus->getDbconnection()->getDBH();
        $logger = $this->applicationBuilder->getLogger();
        $this->templateFile = 'application';

        $menuresource = $this->applicationBuilder->getJsonloader()->loadResource( $this->pageStatus->getSessionWrapper()->getSessionGroup() );
        $this->menubuilder = new MenuJsonTemplate($this->applicationBuilder, $this->pageStatus);
        $this->menubuilder->setMenuStructure( $menuresource );

        // loading data from database using DAO objeects
        $bookDao  = new BookDao;
        $bookDao->setDBH($dbh);
        $bookDao->setLogger( $logger );
        $books = $bookDao->getByFields( array( 'authorid' => $this->getParameters['authorid'] ) );

        $this->menucontainer          = array( $this->menubuilder->createMenu() );
        $this->centralcontainer = array( new BookList( $books ) );
    }
	
    // POST patameters 
    public $post_validation_rules = array( 'asid' => 'required|numeric' );
    public $post_filter_rules     = array( 'asid' => 'trim' );

    /**
     * Executed when POST Request arrives
     */
    public function postRequest() {
        
    }

}
{% endhighlight %}

A controller need to extend *Controller* class. It need to define *CONTROLLER_NAME* constant and it need to overwrite *getRequest* method.
