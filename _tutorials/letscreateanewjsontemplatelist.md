---
layout: page
title: Let's create a new JSON resource type
orderfield: 6
---

## The desired outcome

Let's say that we want to create a new json resource type.

We want to be able to write something like that:

* Edgar Allan Poe 1809 – 1849.
* Herman Melville 1819 – 1891.
* Walt Whitman 1819 - 1892.
* Mark Twain 1835 – 1910.

What we want is a list composed by data extracted from a database through a query and inserted in an unordered list.

The json resource should look like:

{% highlight json %}
{
  "name": "mylist",
  "metadata": { "type":"mylist", "version": "1" },
  "allowedgroups": [ "administrationgroup", "teachergroup", "managergroup" ],
  "get": {
    "sql": "select name, surname, birthyear, deathyear FROM authors;",
    "list": {
      "title": "Authors",
      "fields": [
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

We need now an HTML Block in order to implent this list. It could look like the following:




