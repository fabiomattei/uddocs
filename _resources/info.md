---
layout: page
name: Info
---

This json file allows the user to create an info panel, a panel that basically queryes information from the database and write that information in a panel.
Possible fields are:

* textfield
* textarea
* currency
* date
* dropdown

{% highlight json %}
{
  "name": "mybookinfo",
  "metadata": { "type":"info", "version": "1" },
  "allowedgroups": [ "reader", "writer" ],
  "get": {
    "request": {
      "parameters": [
        { "type":"integer", "validation":"required|integer", "name":"assetid" }
      ]
    },
    "query": {
      "sql": "select * FROM asset WHERE ta_id = :id;",
      "parameters":[
        { "type":"long", "placeholder": ":id", "getparameter": "assetid" }
      ]
    },
    "info": {
      "title": "My new info panel",
      "fields": [
        { "type":"textarea", "validation":"max_len,2500", "name":"name", "label":"Name", "placeholder":"Name", "value":"name", "width":"6", "row":"1" },
        { "type":"currency", "validation":"required|numeric", "name":"amount", "label":"Amount", "placeholder":"10.0", "value":"amount", "width":"6", "row":"1" },
        { "type":"date", "validation":"max_len,10", "name":"duedate", "label":"Due date", "value":"duedate", "width":"12", "row":"2" }
      ]
    }
  }
}
{% endhighlight %}

### Dummy data

Sometimes we need to design an interface not being sure yet about the shape of the database. 
In this case it is possible to replace the query section of the resource with a *dummydata* section.
This will allow you to design all the interface and start to build a mockup of the application without the need to start
to shape the database on the first day when ideas are still unclear.

{% highlight json %}
"dummydata": {
  "id": "1",
  "typeid": "2",
  "name": "My dummy data name",
  "description": "My dummy data description"
}
{% endhighlight %}

The dummy data will simulate a row returned from the database and will populate the form fillng the *sqlfild* section in the
form section of the resource.

The final file will look like this:

{% highlight json %}
{
  "name": "mybookinfo",
  "metadata": { "type":"info", "version": "1" },
  "allowedgroups": [ "reader", "writer" ],
  "get": {
    "request": {
      "parameters": [
        { "type":"integer", "validation":"required|integer", "name":"bookid" }
      ]
    },
    "dummydata": {
      "id": "1",
      "name": "My book name",
      "amount": "1000",
      "duedate": "2025-03-09"
    },
    "info": {
      "title": "My new info panel",
      "fields": [
        { "type":"textarea", "validation":"max_len,2500", "name":"name", "label":"Name", "placeholder":"Name", "value":"name", "width":"6", "row":"1" },
        { "type":"currency", "validation":"required|numeric", "name":"amount", "label":"Amount", "placeholder":"10.0", "value":"amount", "width":"6", "row":"1" },
        { "type":"date", "validation":"max_len,10", "name":"duedate", "label":"Due date", "value":"duedate", "width":"12", "row":"2" }
      ]
    }
  }
}
{% endhighlight %}


### In line editing

It is possible to build an info panel having fields that are editable. This means that when the user click on the **editable field**
he has the possibility to edit the content and update the database accordingly.

For example if we want to make editable the name field created in the previous structure we need to do this: 

{% highlight json %}
{
  "name": "mybookinfo",
  "metadata": { "type":"info", "version": "1" },
  "allowedgroups": [ "reader", "writer" ],
  "get": {
    "request": {
      "parameters": [
        { "type":"integer", "validation":"required|integer", "name":"bookid" }
      ]
    },
    "dummydata": {
      "id": "1",
      "name": "My book name",
      "amount": "1000",
      "duedate": "2025-03-09"
    },
    "info": {
      "title": "My new info panel",
      "fields": [
        { "datatype":"textfield", "validation":"max_len,2500", "name":"name", "label":"Name", "placeholder":"Name", "value":"name", "width":"6", "row":"1",
		"type": "editable", "dataurl": { "controller": "mybookinfo", "parameters": [
		          {"name": "field", "constantparameter": "editablename"},
		          {"name": "id", "sqlfield": "id"},
		          {"name": "csrftoken", "sessionparameter": "csrftoken"}
		        ]
	    },
        { "type":"currency", "validation":"required|numeric", "name":"amount", "label":"Amount", "placeholder":"10.0", "value":"amount", "width":"6", "row":"1" },
        { "type":"date", "validation":"max_len,10", "name":"duedate", "label":"Due date", "value":"duedate", "width":"12", "row":"2" }
      ]
    }
  }
  "post": {
    "request": {
      "postparameters": [
        { "type":"string", "validation":"max_len,25000", "name":"field" },
        { "type":"string", "validation":"max_len,25000h", "name":"value" },
        { "type":"string", "validation":"alpha_numeric_dash", "name":"id" }
      ]
    },
    "inplaceeditor": {
      "switchparameter": "field",
      "updates": [
        {
          "fieldcontent": "editablename",
          "fieldvalue": { "postparameter":"field" },
          "updatequery": {
            "sql":"UPDATE mytable SET myfield = :name, btp_updated = NOW() WHERE id = :id;",
            "parameters":[
              { "type":"string", "placeholder": ":name", "postparameter": "value" },
              { "type":"string", "placeholder": ":id", "postparameter": "id"}
            ]
          },
          "reloadquery": {
            "sql":"SELECT myfield FROM mytable WHERE id = :id;",
            "parameters":[
              { "type":"string", "placeholder": ":prjid", "postparameter": "id"}
            ],
            "fieldtoload": "btp_name"
          }
        }
      ]
    }
  }
}
{% endhighlight %}




