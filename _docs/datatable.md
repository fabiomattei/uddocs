---
layout: page
name: Datatable
---

#Description

A table is a structure that allows the developer to make a query and show to the user the results of the query.
Eventually the table is going to contain links that allows to user to access to different specific features in the system linked to a specific entity shown in the table.

How a table looks like:
(image here)

#The information needed in order to set up a table

### Get or Post parameters

Sometimes we need to pass to the query behind the table some parameter using a get or a post request

Example: parentId = 2302

```
#!json
"parameters": [
  { "type":"integer", "validation":"required|integer", "name":"parentId" }
]
```

### Query

We need to make a query to the tabase in order to populate our table.
The simplest thing to do is just to write the query in plain SQL and eventually connect the paratameters needed to the get or the post paramenters.

```
#!json
"query": {
  "sql": "select id, typeid, name, description FROM mytable WHERE parentid = :parentid;",
  "parameters":[
    { "type":"long", "placeholder": ":parentid", "getparameter": "parentId" }
  ]
}
```

As you can see the SQL parameter is inserted in the query using a placeholder: *:parametername*
The SQL parameter is connected to the GET parameter using: "getparameter": "parentid"
We can insert as many paremeters as we need.

If you want to know more about SQL paramenters check out the specific page.

### Table Structure

The table structure is pretty easy.

We give a title to the table.

We describe a set of buttons, at the top of the table linked to specific actions. This can be useful for instance in order to link a form to create a new entity.

We describe a set of fields giving to each of them a name and a connection to the SQL query: *{"headline": "Name", "sqlfield": "name"}*

We describe a set of actions liked to each item of the table. To see a better description of the actions check out this page.


```
#!json
"table": {
  "title": "My table",
  "topactions": [
    { "label": "New", "resource": "newentityform"] }
  ]
  "fields": [
    {"headline": "Name", "sqlfield": "name"},
    {"headline": "Description", "sqlfield": "description"}
  ],
  "actions": [
    { "icon":"level-down", "label": "Info", "resource": "entityinfo", "parameters":[{"name": "id", "sqlfield": "id"}] },
    { "label": "Edit", "resource": "editentityform", "parameters":[{"name": "id", "sqlfield": "id"}] }
  ]
}
```

# A complete example

Combining all the described information in a unique file we obtain something like this:

{% highlight json %}
{
  "name": "mytableresourcename",
  "metadata": { "type":"datatable", "version": "1" },
  "allowedgroups": [ "administrationgroup", "teachergroup", "managergroup" ],
  "get": {
    "request": {
	  "parameters": [
	    { "type":"integer", "validation":"required|integer", "name":"parentId" }
	  ]
    },
    "sql": "select id, typeid, name, description FROM mytable WHERE parentid = :parentid;",
    "parameters":[
      { "type":"long", "placeholder": ":parentid", "getparameter": "parentId" }
    ],
    "table": {
      "title": "My table",
      "topactions": [
        { "label": "New", "resource": "newentityform"] }
      ]
      "fields": [
        {"headline": "Name", "sqlfield": "rcu_name"},
        {"headline": "Description", "sqlfield": "rcu_description"}
      ],
      "actions": [
        {"icon":"level-down", "label": "Info", "action": "entityinfo", "resource": "inforequestv1", "parameters":[{"name": "id", "sqlfield": "rcu_id"}] },
        {"label": "Edit", "action": "entityform", "resource": "formrequestv1", "parameters":[{"name": "id", "sqlfield": "rcu_id"}] }
      ]
    }
  }
}
{% endhighlight %}
