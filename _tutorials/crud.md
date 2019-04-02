---
layout: page
name: CRUD operations
---

# CRUD operations

This tutorial is designed to help the user to make the first steps using UD

### Create a group of users

{% highlight json %}
{
  "name": "author",
  "metadata": { "type":"group", "version": "1" },
  "defaultdashboard": "articles",
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


### Create a list of articles

{% highlight json %}
{
  "name": "unitstable",
  "metadata": { "type":"table", "version": "1" },
  "allowedgroups": [ "administrationgroup", "assetspecialistgroup", "centralheadofficegroup", "riskmanagergroup", "operatorgroup" ],
  "get": {
    "request": {
      "parameters": [
        { "type":"long", "placeholder": ":unitid", "getparameter": "unitid" }
      ]
    },
    "query": {
      "sql": "SELECT id, type, name, description FROM mytable WHERE typeid = :typeid AND unitid=:unitid;",
      "parameters":[
        { "type":"long", "placeholder": ":typeid", "getparameter": "unitid" }
        { "type":"long", "placeholder": ":typeid", "sessionparameter": "typeid" }
      ]
    },
    "table": {
      "title": "My table",
      "topactions": [
        { "label": "New", "resource": "newentityform"] }
      ]
      "fields": [
        {"headline": "Name", "sqlfield": "name"},
        {"headline": "Description", "sqlfield": "description"},
        {"headline": "Field with constant value", "constant": ""}
      ],
      "actions": [
        {"icon":"level-down", "label": "Info", "action": "entityinfo", "resource": "inforequestv1", "parameters":[{"name": "id", "sqlfield": "id"}] },
        {"label": "Edit", "action": "entityform", "resource": "formrequestv1", "parameters":[{"name": "id", "sqlfield": "id"}] }
      ]
    }
  }
}
{% endhighlight %}

### Create a form to insert a new article

### Create a form to update an article

### Create a logic to delete an article

