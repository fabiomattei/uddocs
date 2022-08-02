---
layout: page
name: Use Case
---

Sometimes the result of a POST call is not just the execution of a bounch of SQL Queries we can put in the <a href="{{site.baseurl}}/docs/transaction">transactions</a> section of a json resource. Sometimes we need to implement some logic and iin those cases we need the power and the freedom given by a programming language.

In order to do that we need to subclass *BaseUseCase* and implement our logic in the *performAction* method.

{% highlight php %}
namespace myproject;

class MyUseCase extends BaseUseCase {
    function performAction() {
        // Load all parameters sent with the json structure
        $this->loadParameters();

        // assign database handler and logger to local variable in order to use it later
        $dbh = $this->pageStatus->getDbconnection()->getDBH();
        $logger = $this->applicationBuilder->getLogger();
		
        // here you can implement your logic using parameter(s) sent from the json resource that called the use case
        // you can address the parameters using the object property $this->useCaseParameters.
        // in this case we are receving two parameter:
        //    { "type":"long", "name":"articleid", "postparameter":"articleid" },
        //    { "type":"long", "name":"authorid", "postparameter":"authorid" }
        $this->useCaseParameters['articleid'];
        $this->useCaseParameters['authorid'];
		
        // at the end you can give some positive feedback
        $this->pageStatus->addSuccess( "Article linked to selected author" );
		
        // or you can give some negative feedback
        $this->pageStatus->addError( "Ehy there is an error here" );
        $this->pageStatus->addWarning( "Adding some warning" );
    }
}
{% endhighlight %}

In order to have the application to execute the use case you can add this to the post section of a json resource.

{% highlight json %}
"usecases" : [
  { "name": "MyUseCase", "parameters": [
    { "type":"long", "name":"articleid", "postparameter":"articleid" },
    { "type":"long", "name":"authorid", "postparameter":"authorid" }
  ] }
]
{% endhighlight %}

As you can see we can link not one but many usecases to a resource post section, what we need to do is name the class name we want to call and pass the parameters. The parameters listed will be passed to the class in the useCaseParameters assocative array.

## Let's make an example:

