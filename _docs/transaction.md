---
layout: page
name: Transaction
---

Transactions are SQL statments that make changes to the database. They can be *insert*, *update* or *delete*.

An example of the transaction is the following json object:

{% highlight json %}
{
  "sql":"DELETE from requestv1 WHERE id=:id;",
  "parameters":[
    { "placeholder": ":id", "getparameter": "id" }
  ]
}
{% endhighlight %}

As you can see we need to define a SQL statment and an array of parameters we need to pass to it.

A transaction is defined:

* label: a string that allows referrals to this statment from other following SQL statments, optional
* sql: SQL statment in plain SQL, mandatory
* paramters: array of parameter objects, optional

# SQL parameters

Each SQL statment can have one or many parameters.
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

Sometimes we have two SQL statments in one transaction. 
It is easy to implement that as the transaction array can host more than one SQL statment. If we need to create references between different SQL statments, for example when we are storing an id coming from a previous statment. In the following esample we are insertin a book in a database and an author having a FK key to the book entity. As you can see the label of the first SQL statment is used for reference in the second SQL statment.

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



