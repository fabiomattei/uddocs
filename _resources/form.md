---
layout: page
name: Form
---

![Form](images/tut02-form.png){:class="aside-image"}

## Structure

A form resource allows the programmer to insert a form in an application. 
A form is a complex structure. A form often need to be filled in advance with data coming from the database.
Then it send POST data through a POST request and after that some logic is executed in order to update the 
data.

## GET section 

### Request

Sometimes we need to pass some parameter to a query that feels the form.

Example: parentId = 2302

{% highlight json %}
"parameters": [
  { "type":"integer", "validation":"required|integer", "name":"parentId" }
]
{% endhighlight %}
If you want to know about the <a href="{{site.baseurl}}/docs/validation">Validation</a> check out the related page.

### Query

Oftentimes we need to pre-fill the form with some data that comes from the database in order to allow the user to edit that data.
In order to do that we need to write a query.
We can write the query in plain SQL and eventually connect the paratameters needed to get paramenters.

{% highlight json %}
"query": {
  "sql": "select id, typeid, name, description FROM mytable WHERE parentid = :parentid;",
  "parameters":[
    { "type":"long", "placeholder": ":parentid", "getparameter": "parentId" }
  ]
}
{% endhighlight %}

As you can see the SQL parameter is inserted in the query using a placeholder: *:parametername*
The SQL parameter is connected to the GET parameter using: "getparameter": "parentid"
We can insert as many paremeters as we need.

It is possible to send to the query a session parameter using: "sessionparameter": "sessionparametername"

It is possible to send to the query a constant parameter using: "constantparameter": "constantvalue"

If you want to know more about SQL paramenters check out the <a href="{{site.baseurl}}/docs/query">Query</a> page.

### Dummy data

Sometimes we need to design an interface not being sure yet about the shape of the database. 
In this case it is possible to replace the query section of the resource with a **dummydata** section.
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

### Form fields

A form is made by different fields. These fields will allow the user to insert information. 
All fields are listed in a list named **fields** that is part of the **form** object.

{% highlight json %}
"form": {
  "title": "",
  "fields": [
    { "type":"textarea", "name":"name", "label":"Name", "placeholder":"Name", "sqlfield":"name", "width":"6", "row":"1" },
    { "type":"currency", "name":"amount", "label":"Amount", "placeholder":"10.0", "sqlfield":"amount", "width":"6", "row":"1" },
    { "type":"date", "name":"duedate", "label":"Due date", "sqlfield":"duedate", "width":"12", "row":"2" },
    { "type":"sqldropdown", "name":"categoryid", "label":"Category", "sqlfield":"c_name", "width":"6", "row":"2",
      "query": {
        "sql": "SELECT c_id, c_name FROM categories;",
        "parameters":[]
      },
      "valuesqlfield": "c_id",
      "labelsqlfield": "c_name"
    },
    { "type":"dropdown", "name":"tag", "label":"Tag", "sqlfield":"r_tag", "width":"6", "row":"4", "options": [
      { "value": "0", "label":"(Not set)" },
      { "value": "1", "label":"1: Easy to user" },
      { "value": "2", "label":"2: Easy enough" },
      { "value": "3", "label":"3: Medium" },
      { "value": "4", "label":"4: Medium high" },
      { "value": "5", "label":"5: Difficult to use" }
      ]
    },
    { "type":"hidden", "name":"assetid", "getparameter":"assetid", "row":"5" },
	{ "type":"submitbutton", "width":"2", "row":"5", "name": "Save", "constantparameter": "Save" },
  ]
}
{% endhighlight %}

### Field properties

As you can see a field is a complicated structure, we need to define many things in order to set them up properly.
The properties of the **field** object are:

* label: the descriptive text associated to the field the user can read
* type: the type of the field [textfield, numeric, textarea, currency, date, dropdown, sqldropdown, hidden, submitbutton]
* name: html name attrbute associated to the field
* placeholder: html placeholder attrbute associated to the field
* sqlfield: used in case we need to load data in a field that comes from a query
* sessionparameter: used in case we need to load data in a field that comes from a session parameter
* getparameter: used in case we need to load data in a field that comes from a getparameter parameter
* postparameter: used in case we need to load data in a field that comes from a postparameter parameter
* constantparameter: used in case we need to load data in a field that comes from a constantparameter parameter
* width: the width of the field in bootstrap terms
* row: the row number where the field is located in bootstrap terms
* options: used in case of dropdown field a list of possible option objects Ex: { "value": "0", "label":"(Not set)" }
* query: used by sqldropdown
* valuesqlfield: used by sqldropdown
* labelsqlfield: used by sqldropdown

## POST section

### Request

What a form usually does is to pass POST parameters using a POST request and then to process that parameters
We need to give a description of the post parameters and of the validation we need to apply to them.

{% highlight json %}
"request": {
  "postparameters": [
    { "name":"name", "validation":"required|max_len,250" },
    { "name":"amount", "validation":"numeric" },
    { "name":"duedate", "validation":"max_len,20" },
    { "name":"categoryid", "validation":"numeric" },
    { "name":"tag", "validation":"numeric" },
    { "name":"assetid", "validation":"required|integer" },
  ]
}
{% endhighlight %}

If you want to know more about the <a href="{{site.baseurl}}/docs/validation">Validation</a> check out the related page.

### Transactions

Once data passed the post validation stage we usually need to save that data.
The trasactions phase will do that. 
A transaction is a list of queries that are going to change the status of the database.
A commit is set in order to make all changes permanent only if all queries succeed.

{% highlight json %}
"transactions": [
  {
	"label": "firstquery"
    "sql":"UPDATE mytable SET name = :name, amount = :amount, duedate = :duedate, categoryid= :categoryid, tag = :tag WHERE id=:assetid;",
    "parameters":[
      { "type":"string", "placeholder": ":name", "postparameter": "name" },
      { "type":"string", "placeholder": ":amount", "postparameter": "amount" },
      { "type":"string", "placeholder": ":duedate", "postparameter": "duedate" },
      { "type":"string", "placeholder": ":categoryid", "postparameter": "categoryid" },
      { "type":"string", "placeholder": ":tag", "postparameter": "tag" },
      { "type":"long", "placeholder": ":assetid", "postparameter": "assetid" }
    ]
  },
  {
    "sql":"UPDATE asset SET name = :name WHERE id=:technicalassetid;",
    "parameters":[
      { "type":"string", "placeholder": ":name", "postparameter": "name" },
      { "type":"long", "placeholder": ":assetid", "postparameter": "assetid" }
    ]
  }
]
{% endhighlight %}

As you can see the SQL parameter is inserted in the query using a placeholder: *:parametername*
The SQL parameter is connected to the GET parameter using: **"postparameter": "parentid"**
We can insert as many paremeters as we need.

It is possible to send to the query a session parameter using: **"sessionparameter": "sessionparametername"**

It is possible to send to the query a constant parameter using: **"constantparameter": "constantvalue"**

### Redirection

Once all queries listed in a transaction are completed it is possible to redirect the user to a new page.
All we need to do is to add a redirect block to the page.

{% highlight json %}
    "redirect": {
      "action": { "resource": "asset-list" }
    }
{% endhighlight %}

Sometimes we need to do a redirection with **parameters returned by the transactions**.
What we need to do is refer the the **query label** using the **returnedid** property.

{% highlight json %}
    "redirect": {
      "action": { "resource": "siti-immersione-edit", "parameters":[{"name": "id", "returnedid": "insert-new-immersione"}] }
    }
{% endhighlight %}

## Complete example

{% highlight json %}
{
  "name": "sampleform",
  "metadata": { "type":"form", "version": "1" },
  "allowedgroups": [ "administrationgroup", "teachergroup", "managergroup" ],
  "get": {
    "request": {
      "parameters": [
        { "type":"long", "validation":"required|numeric", "name":"assetid" }
      ]
    },
    "query": {
      "sql": "SELECT id, name, amount, duedate, c_name, r_tag, assetid FROM mytable WHERE id = :id;",
      "parameters":[
        { "type":"long", "placeholder": ":id", "getparameter": "assetid" }
      ]
    },
    "form": {
      "title": "My form",
      "action": { "resource":"sampleform" },
      "method": "POST",
	  "fields": [
	    { "type":"textarea", "name":"name", "label":"Name", "placeholder":"Name", "sqlfield":"name", "width":"6", "row":"1" },
	    { "type":"currency", "name":"amount", "label":"Amount", "placeholder":"10.0", "sqlfield":"amount", "width":"6", "row":"1" },
	    { "type":"date", "name":"duedate", "label":"Due date", "sqlfield":"duedate", "width":"12", "row":"2" },
	    { "type":"sqldropdown", "name":"categoryid", "label":"Category", "sqlfield":"c_name", "width":"6", "row":"2",
	      "query": {
	        "sql": "SELECT c_id, c_name FROM categories;",
	        "parameters":[]
	      },
	      "valuesqlfield": "c_id",
	      "labelsqlfield": "c_name"
	    },
	    { "type":"dropdown", "name":"tag", "label":"Tag", "sqlfield":"r_tag", "width":"6", "row":"4", "options": [
	      { "value": "0", "label":"(Not set)" },
	      { "value": "1", "label":"1: Easy to user" },
	      { "value": "2", "label":"2: Easy enough" },
	      { "value": "3", "label":"3: Medium" },
	      { "value": "4", "label":"4: Medium high" },
	      { "value": "5", "label":"5: Difficult to use" }
	      ]
	    },
	    { "type":"hidden", "name":"assetid", "getparameter":"assetid", "row":"5" },
		{ "type":"submitbutton", "width":"2", "row":"5", "name": "Save", "constantparameter": "Save" },
	  ]
    }
  },
  "post": {
    "request": {
	  "postparameters": [
	    { "name":"name", "validation":"required|max_len,250" },
	    { "name":"amount", "validation":"numeric" },
	    { "name":"duedate", "validation":"max_len,20" },
	    { "name":"categoryid", "validation":"numeric" },
	    { "name":"tag", "validation":"numeric" },
	    { "name":"assetid", "validation":"required|integer" },
	  ]
    },
	"transactions": [
	  {
		"label": "firstquery"
	    "sql":"UPDATE mytable SET name = :name, amount = :amount, duedate = :duedate, categoryid= :categoryid, tag = :tag WHERE id=:assetid;",
	    "parameters":[
	      { "type":"string", "placeholder": ":name", "postparameter": "name" },
	      { "type":"string", "placeholder": ":amount", "postparameter": "amount" },
	      { "type":"string", "placeholder": ":duedate", "postparameter": "duedate" },
	      { "type":"string", "placeholder": ":categoryid", "postparameter": "categoryid" },
	      { "type":"string", "placeholder": ":tag", "postparameter": "tag" },
	      { "type":"long", "placeholder": ":assetid", "postparameter": "assetid" }
	    ]
	  },
	  {
	    "sql":"UPDATE asset SET name = :name WHERE id=:technicalassetid;",
	    "parameters":[
	      { "type":"string", "placeholder": ":name", "postparameter": "name" },
	      { "type":"long", "placeholder": ":assetid", "postparameter": "assetid" }
	    ]
	  }
	],
    "redirect": {
      "action": { "resource": "asset-list" }
    }
  }
}
{% endhighlight %}

