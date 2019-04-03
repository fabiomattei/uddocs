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
        { "label":"Articles", "resource":"articles" },
        { "label":"New article", "resource":"newarticle" }
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
        {"label": "Edit", "action": "articleform", "parameters":[{"name": "id", "sqlfield": "id"}] },
        {"label": "Delete", "action": "articledelete", "parameters":[{"name": "id", "sqlfield": "id"}] }
      ]
    }
  }
}
{% endhighlight %}

### Create a form to insert a new article

### Create a form to update an article

### Create a logic to delete an article

