---
layout: page
name: Query
---

Queries are a way to get from the database the information we need. They are mostly *select* statment

An example of a query is the following json object:

{% highlight json %}
"query": {
  "sql": "select id, typeid, name, description FROM mytable WHERE parentid = :parentid;",
  "parameters":[
    { "type":"long", "placeholder": ":parentid", "getparameter": "parentId" }
  ]
}
{% endhighlight %}

As you can see we need to define a SQL query and an array of parameters we need to pass to the query.

A query is defined:

* sql: query in plain SQL, mandatory
* parameters: optional, array of parameter objects
* debug: optional, shows information about the queries 

# SQL parameters

Each query can have one or many parameters.
A parameter is a json object that contains three properties

### type

It specifies the type of the parameter during the binding process. 

* long, int: PDO::PARAM_INT
* string, str: PDO::PARAM_STR
* bool, boolean: PDO::PARAM_BOOL
* float, decimal: PDO::PARAM_STR

### placeholder

Each parameter correspond to a placeholder in the query.

* placeholder: the placeholder name

### data

One of the following parameters is mandatory:

* sqlfield: the name of the field the value is taken from
* getparameter: the name of the get parameter the value is taken from
* postparemeter: the name of the post parameter the value is taken from
* sessionparameter: the name of the session paramenter the value is taken from
* constant: a constant parameter
* composite: used in in a column we want to put more than one information in a single parameter
* filter: the filter functions to apply to the parameeter

{% highlight json %}
{ "type":"string", "placeholder": ":description", "composite":"${placeholder1} ${placeholder2}", "parameters": [
      { "name":"${placeholder1}", "sqlfield": "name"  },
      { "name":"${placeholder2}", "sqlfield": "surname"  }
    ] 
}
{% endhighlight %}

### Value
This property allows the software developer to specify different sources of information for a single parameter. The sources
are evaluated on by one and the first not null result becomes the value of the field. A filter can be applyed to the not-null result of a value.

{% highlight json %}
{ "type":"string", "placeholder": ":description", "value": [
  { "type":"string", "sqlfield":"name", "filter":"substr,0,1" },
  { "type":"string", "constant":"" }
] }
{% endhighlight %}

In this example this column is filled with the information stored in the "name" sql field, result of a query. If that information is not null the filter substr is applyed. In case that information is null the second element in the list is evaluated, in this case it is the constant empty string.

# Debug

Sometimes we need help to fix queries.

{% highlight json %}
"query": {
  "sql": "select id, typeid, name, description FROM mytable WHERE parentid = :parentid;",
  "parameters":[
    { "type":"long", "placeholder": ":parentid", "getparameter": "parentId" }
  ],
  "debug":"on"
}
{% endhighlight %}

The debug parameter print on the page the query interpolated with all paramenters and shows the output of the command [PDOStatement::debugDumpParams](https://www.php.net/manual/en/pdostatement.debugdumpparams.php).
 


