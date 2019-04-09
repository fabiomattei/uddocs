---
layout: page
name: Query
---

# Query

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
* paramters: array of parameter objects, optional

# SQL parameters

Each query can have one or many parameters.
A parameter is a json object that contains two properties

The name property is mandatory:

* name: the parameter name

One of the following parameters is mandatory:

* sqlfield: the name of the field the value is taken from
* constantparameter: a constant parameter
* getparameter: the name of the get parameter the value is taken from
* postparemeter: the name of the post parameter the value is taken from
