---
layout: page
name: Group
---

# Structure

A group is a set of users that can access to a set of resourcers. We need to define groups in order to allow the users
to perform the operations they can and to avoid they can access to operations they should not.

Each application has gruops of users with different access level to the different features of the application. We need to define those groups.
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
