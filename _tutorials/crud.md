---
layout: page
name: CRUD operations
---

# CRUD operations

This tutorial is designed to help the user to make the first steps using UD

### Create a group of users

The first thing we need to do is to create a group of users.
Each application has grups of users with different access level to the different features of the application. We need to define those groups.
For each group we need to define the default action, that is the action triggered once we go trough the login process and the menu this specific group is going to use in order to navigate the application.

{% highlight json %}
{
  "name": "author",
  "metadata": { "type":"group", "version": "1" },
  "defaultaction": "articles",
  "home": { "label":"My app", "action":"#" },
  "menu": [
    {
      "label":"Articles", "icon":"asset",
      "submenu": [
        { "label":"Articles", "resource":"articlestable" },
        { "label":"New article", "resource":"newarticleform" }
      ]
    }
  ]
}
{% endhighlight %}

In this case we define just one menu item with two sub items:

* Articles: a link the a table that shows all aticles in the system
* New article: a link to a form that allows this group to insert a new article in the system

For more information about the group file syntax please check out <a href="{{site.baseurl}}/docs/group">group docs page</a>.

### Create a list of articles

{% highlight json %}
{
  "name": "articlestable",
  "metadata": { "type":"table", "version": "1" },
  "allowedgroups": [ "author" ],
  "get": {
    "request": {
      "parameters": []
    },
    "query": {
      "sql": "SELECT * FROM articles;",
      "parameters":[]
    },
    "table": {
      "title": "My articles",
      "topactions": [
        { "label": "New", "resource": "newarticle"] }
      ]
      "fields": [
        {"headline": "Title", "sqlfield": "title"},
        {"headline": "Description", "sqlfield": "description"},
        {"headline": "Date", "sqlfield": "created"}
      ],
      "actions": [
        {"label": "Edit", "action": "editarticleform", "parameters":[{"name": "id", "sqlfield": "id"}] },
        {"label": "Delete", "action": "deletearticlelogic", "parameters":[{"name": "id", "sqlfield": "id"}] }
      ]
    }
  }
}
{% endhighlight %}

### Create a form to insert a new article

{% highlight json %}
{
  "name": "newarticleform",
  "metadata": { "type":"form", "version": "1" },
  "allowedgroups": [ "author" ],
  "get": {
    "form": {
      "title": "My new article",
      "fields": [
        { "type":"textfield", "name":"title", "label":"Title", "placeholder":"title", "width":"12", "row":"1" },
        { "type":"textarea", "name":"description", "label":"Description", "placeholder":"Description", "width":"12", "row":"2" },
		{ "type": "submitbutton", "width":"2", "row":"3", "name": "Save", "constantparameter": "Save" },
      ]
    }
  },
  "post": {
    "request": {
      "postparameters": [
        { "name":"title", "validation":"required|max_len,250" },
        { "name":"description", "validation":"alpha_numeric" }
      ]
    },
    "transactions": [
      {
        "sql":"INSERT INTO articles ( title, description, created ) VALUES ( :title, :description, NOW() );",
        "parameters":[
          { "type":"string", "placeholder": ":title", "postparameter": "title" },
          { "type":"long", "placeholder": ":description", "postparameter": "description" }
        ]
      }
    ]
  }
}
{% endhighlight %}

### Create a form to update an article

{% highlight json %}
{
  "name": "editarticleform",
  "metadata": { "type":"form", "version": "1" },
  "allowedgroups": [ "author" ],
  "get": {
    "request": {
      "parameters": [
        { "type":"long", "validation":"required|numeric", "name":"id" }
      ]
    },
    "query": {
      "sql": "SELECT title, description FROM articles WHERE id = :id;",
      "parameters":[
        { "type":"long", "placeholder": ":id", "getparameter": "id" }
      ]
    },
    "form": {
      "title": "Edit article",
      "fields": [
        { "type":"textfield", "name":"title", "label":"Title", "placeholder":"title", "width":"12", "row":"1" },
        { "type":"textarea", "name":"description", "label":"Description", "placeholder":"Description", "width":"12", "row":"2" },
		{ "type":"hidden", "name":"id", "sqlfield":"id", "row":"3" },
		{ "type": "submitbutton", "width":"2", "row":"3", "name": "Save", "constantparameter": "Save" },
      ]
    }
  },
  "post": {
    "request": {
      "postparameters": [
        { "name":"title", "validation":"required|max_len,250" },
        { "name":"description", "validation":"alpha_numeric" },
        { "name":"id", "validation":"required|integer" }
      ]
    },
    "transactions": [
      {
        "sql":"UPDATE articles SET title = :title, description = :description WHERE id=:id;",
        "parameters":[
          { "type":"string", "placeholder": ":title", "postparameter": "title" },
          { "type":"string", "placeholder": ":description", "postparameter": "description" },
          { "type":"long", "placeholder": ":id", "postparameter": "id" }
        ]
      }
    ],
  }
}
{% endhighlight %}

### Create a logic to delete an article

{% highlight json %}
{
  "name": "deletearticlelogic",
  "metadata": { "type":"transaction", "version": "1" },
  "allowedgroups": [ "author" ],
  "get": {
    "request": {
      "parameters": [
        { "type":"long", "validation":"required|numeric", "name":"id" }
      ]
    },
    "transactions": [
      {
        "sql":"DELETE FROM article WHERE id=:id;",
        "parameters":[
          { "placeholder": ":id", "getparameter": "id" }
        ]
      }
    ],
    "redirect": {
      "internal": { "type": "onepageback" }
    }
  }
}
{% endhighlight %}
