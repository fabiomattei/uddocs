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
