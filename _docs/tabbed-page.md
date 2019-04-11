---
layout: page
name: Tabbed page
---

# Tabbed page

A tabbed-page is a container of resources that allows the programmer to compose a page with more tabs containing each one a different resource.
The page is designed using bootstrap framework.

A tabbed-page contains an array of **panels**

For each panel we need to spcify:

* title: is going to be shown in the interface
* resource: the identifier of the json resource in the system

We do not need to provide parameter to this specific resource as the GET parameters are calculated recoursively from the contained resources. We expect the action that link this page to provide all parameters required by all resources contained in it

### Complete example

{% highlight json %}
{
  "name": "myexampletabbedpage",
  "metadata": { "type":"tabbedpage", "version": "1" },
  "allowedgroups": [ "administrationgroup", "operatorgroup" ],
  "title":"My tabbed page",
  "get": {
    "request": {
    }
  },
  "panels":[
    { "title":"My first panel", "resource":"myfirstpanel" },
    { "title":"My second panel", "resource":"mysecondpanel" },
    { "title":"My third panel", "resource":"mythirdpanel" }
  ]
}
{% endhighlight %}
