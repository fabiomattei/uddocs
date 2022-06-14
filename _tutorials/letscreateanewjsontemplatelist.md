---
layout: page
title: Let's create a new JSON resource type
orderfield: 7
---

### The desired outcome

Let's say that we want to create a new json resource type.

We want to be able to write something like that:

* Edgar Allan Poe 1809 – 1849.
* Herman Melville 1819 – 1891.
* Walt Whitman 1819 - 1892.
* Mark Twain 1835 – 1910.

What we want is a list composed by data extracted from a database through a query and inserted in an unordered list.

### The Json resource I would like to write in order to have the desired outcome

The json resource should look like:

{% highlight json %}
{
  "name": "mylist",
  "metadata": { "type":"mylist", "version": "1" },
  "allowedgroups": [ "administrationgroup", "teachergroup", "managergroup" ],
  "get": {
    "query": {
      "sql": "select name, surname, birthyear, deathyear FROM authors;"
    }
    "list": {
      "rowfields": [
        { "sqlfield": "name" },
        { "sqlfield": "surname" },
        { "sqlfield": "birthyear" },
        { "sqlfield": "deathyear" }
      ]
    }
  }
}
{% endhighlight %}

Obviosly in the future we will be able to write as many **mylist** resources as we wish having the ability of passing parameters and changing the SQL Query.

### The HTMLBlock

We need now an HTML Block in order to implent this list. It could look like the following:

{% highlight php %}
use Fabiom\UglyDuckling\Common\Blocks\BaseHTMLBlock;

class HTMLBlockList extends BaseHTMLBlock {
	
    const HTML_BLOCK_NAME = 'basehtmlblocklist';	
    private string $listbody;
	
    function addLi(string $litext) {
        $this->listbody += '<li>'.$litext.'</li>';
    }

    function getHTML(): string {
        return '<ul>.'$this->listbody'.</ul>';
    }

}
{% endhighlight %}

### The json template file

Now we need a Json template in order to orchestrate everything:

{% highlight php %}
<?php
use Fabiom\UglyDuckling\Common\Json\JsonTemplates\JsonTemplate;
use Fabiom\UglyDuckling\Custom\HTMLBlocks\HTMLBlockExample;

class JsonTemplateExample extends JsonTemplate {

    const blocktype = 'mylist';
    
    /**
     * @return \Fabiom\UglyDuckling\Common\Blocks\EmptyHTMLBlock|HTMLBlockExample
     */
    public function createHTMLBlock() {
        $out = new HTMLBlockList;
        
        $queryExecutor = $this->pageStatus->getQueryExecutor();
        $queryExecutor->setQueryStructure( $this->resource->get->query );
        $entities = $queryExecutor->executeSql();
        
        foreach ($entities as $entity) {
            $this->pageStatus->setLastEntity($entity);
            
            $litext = '';
            foreach ( $this->resource->get->list->rowfields as $rowfield ) {
                $litext += $this->pageStatus->getValue( $rowfield );
            }
            $out->addLi($litext);
        }
        
        return $out;
    }

}
{% endhighlight %}

And this closes the circle. Now we are able to add to our software all the lists we need and all we need to do is to add a json resource to the code base where we are free to change the query and the showed fields.

### What now?

Now I am able to create another resource for a new list to add to another page and I do not need to create again HTMLBlock and Json template file.

Let's say I want to create another list maybe listing the books of a given author. The author id will be passed as GET parameter will be validated and eventually used in the following query in order to filter data.

{% highlight json %}
{
  "name": "mylist",
  "metadata": { "type":"mylist", "version": "1" },
  "allowedgroups": [ "administrationgroup", "teachergroup", "managergroup" ],
  "get": {
    "request": {
      "parameters": [ { "type":"integer", "validation":"required|integer", "name":"authorid" } ]
    },
    "query": {
      "sql": "select title FROM books WHERE authorid = :id;",
      "parameters":[
        { "type":"long", "placeholder": ":id", "getparameter": "authorid" }
      ]
    }
    "list": {
      "rowfields": [
        { "sqlfield": "title" }
      ]
    }
  }
}
{% endhighlight %}



