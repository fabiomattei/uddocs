---
layout: page
name: Use Case
---

Sometimes the desired result of a POST call is not just the execution of a bounch of SQL queries we can put in the <a href="{{site.baseurl}}/docs/transaction">transactions</a> section of a json resource. Sometimes we need to implement some logic and in those cases we need the power and the freedom given by a programming language.

In order to give this opportunity have created what I call a use case. A use case is a class containing a method that takes paramters as input, perform operations and gives back output.

A *use case* class is a subclass of *BaseUseCase* and implement the needed logic in the *performAction* method.

The base for implementation looks like the following code.

{% highlight php %}
class MyUseCase extends BaseUseCase {
    function performAction() {
        // Load all parameters sent using the json structure
        $this->loadParameters();

        // assign database handler and logger to local variables in order to use it later
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

In order to link th use case class to application we need to add this to the post section of a json resource.

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

We need to create a form that link an author, already in the database to a book, but befor to do that we need to be sure the author is not already linked.

This is the complete script:

{% highlight json %}
{
  "name": "bowtie-add-author-form",
  "metadata": { "type":"form", "version": "1" },
  "allowedgroups": [ "manager" ],
  "get": {
    "request": {
      "parameters": [
        { "type":"long", "validation":"required|integer", "name":"bookid" }
      ]
    },
    "form": {
      "formid": "lnk-author-form",
      "title": "Link an author",
      "fields": [
        { "type":"sqldropdown", "name":"authorid", "label":"", "width":"6", "row":"2",
          "query": {
            "sql": "SELECT authorid, name, surname FROM authors;",
            "parameters":[]
          },
          "valuesqlfield": "authorid",
          "labelsqlfields": ["name", "surname"]
        },
        { "type":"hidden", "name":"bookid", "getparameter":"bookid", "row":"2" }
      ]
      "bottombuttons": [
        {
          "type":"ajaxbutton",
          "cssclass":"udbuttonajaxrequest",
          "label":"Link author",
          "method": "POST",
          "dataudformid":"add-author-form",
          "dataudiddestination": "authorrowblock",
          "dataudurl":{
            "controller": "partial",
            "resource":"bowtie-add-author-form"
          }
        }
      ]
    }
  },
  "post" : {
    "request": {
      "postparameters": [
        { "type":"long", "validation":"required|numeric", "name":"authorid" },
        { "type":"long", "validation":"required|numeric", "name":"bookid" }
      ]
    },
    "usecases" : [
      { "name": "LinkAuthorToBookUseCase", "parameters": [
        { "type":"long", "name":"authorid", "postparameter":"authorid" },
        { "type":"long", "name":"bookid", "postparameter":"bookid" }
      ] }
    ],
    "ajaxreponses": [
      {
        "type": "appendurl",
        "destination": "#panel4",
        "method": "GET",
        "url": {
          "controller": "partial",
          "resource": "bowtie-causes-barriers-table",
          "parameters": [
            {"name": "btmid", "postparameter": "btmid"},
            {"name": "threatid", "postparameter": "threatid"}
          ]
        }
      },
      {
        "type": "empty",
        "destination":  "#threatbuttonbarcontainer"
      }
    ],
    "error": { "destination": "#panel4", "position": "afterbegin" }
  }
}
{% endhighlight %}

The form is implemented at the start as usual, but you can see that as use case is part of the post section passing tow paremeter received as POST parameters.

The use case class will be implemented as following:

{% highlight php %}
namespace myproject;

class LinkAuthorToBookUseCase extends BaseUseCase {
    function performAction() {
        // Load all parameters sent with the json structure
        $this->loadParameters();

        // assign database handler and logger to local variables in order to use it later
        $dbh = $this->pageStatus->getDbconnection()->getDBH();
        $logger = $this->applicationBuilder->getLogger();
		
        $authorsBookDao = new AuthorsBookDao;
        $authorsBookDao->setDBH( $dbh );
        $authorsBookDao->setLogger( $logger );
        
		$authorsBookNumber = $authorsBookDao->countByFields( ['authorid' => $this->useCaseParameters['authorid'] ] );

        if ( $authorsBookNumber == 0 ) {
            $authorsBookDao->insert( [ 'authorid' => $this->useCaseParameters['authorid'], 'bookid' => $this->useCaseParameters['bookid'] ] );
        } else {
            $this->pageStatus->addError( "This author is already linked to this book" );
        }
    }
}
{% endhighlight %}


