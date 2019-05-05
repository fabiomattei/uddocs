---
layout: page
name: Group
---

# Structure

A group is a set of users that can access to a set of resourcers. We need to define groups in order to allow the users to perform the operations they can and to avoid the possibility of they accessing to operations they should not have access to.

Each application has gruops of users with different access level to the different features of the application. We need to define those groups.

For each group we need to define:

* default action: the action triggered after the user passed the login process, it is basically the first resource or interface of the application the user cas see.
* menu: the menu the person that belongs to a certain group is going to use in order to navigate the application. A menu is basically a set of links or we should say a set of <a href="{{site.baseurl}}/docs/action">actions</a> in UD terms.

This is a complete example of a group file:

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

As you can see this group has been named **author**. The group has, as a dafault action, the <a href="{{site.baseurl}}/docs/resource">resource</a> named **articles**.

In this case we define just one menu item with two sub items:

* Articles: a link the a table that shows all aticles in the system
* New article: a link to a form that allows this group to insert a new article in the system
