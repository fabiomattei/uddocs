---
layout: page
name: Transaction
---

# Transactions

Transactions are queries that make changes to the database. They can be *insert*, *update* or *delete*.

An example of the transaction is the following json object:

{% highlight json %}
{
  "sql":"DELETE from requestv1 WHERE id=:id;",
  "parameters":[
    { "placeholder": ":id", "getparameter": "id" }
  ]
}
{% endhighlight %}

As you can see we need to define a SQL query and an array of parameters we need to pass to the query.

A transaction is defined:

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
* sessionparameter: the nome of a session variable
* returnedid: the id returned from another query

# Transactions with dependent parameters

Sometimes we have two queries in one transaction. It is easy to implement that as the transaction array can host more than one query. Sometimes we need to create references between different queries, for example when we are storing an id. In the following esample we are insertin a book in a database and an author having a FK key to the book entity. As you can see the label of the first SQL statment is used for reference in the second SQL statment.

{% highlight json %}
{
  {
    "label": "bookinsert",
    "sql":"INSERT INTO books (title) VALUES (:title);",
    "parameters":[
      { "type":"string", "placeholder": ":title", "postparameter": "title" }
    ]
  },
  {
    "label": "secondinsert",
    "sql": "INSERT INTO author (name, surname, bookid, creationdate) VALUES (:name, :surname, :bookid, NOW());",
    "parameters":[
      { "type":"long", "placeholder": ":name", "sessionparameter": "name" },
      { "type":"long", "placeholder": ":surname", "postparameter": "surname" },
      { "type":"long", "placeholder": ":bookid", "returnedid": "bookinsert" }
    ]
  }
}
{% endhighlight %}



