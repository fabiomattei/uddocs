---
layout: page
name: Dashboard
---

# Dashboard

A dashboard is a container of <a href="{{site.baseurl}}/docs/resource">resources</a> that allows the programmer to compose a page.
The page is designed using bootstrap framework and for each resource we need to specify the row where we want to put it and how many columns we want the resource to take.

A dashboard contains an array of **panels**

For each panel we need to specify:

* title: it is going to be shown in the interface
* resource: the identifier of the json resource in the system
* row: which row cotains the specific panel
* width: how many columns we want the panel to extend

We do not need to provide parameter to this specific resource as the GET parameters are calculated recoursively from the contained resources. We expect the action that link this dashboard to provide all parameters required by all resources contained in it

### Complete example


{% highlight json %}
{
  "name": "teacherdashboard",
  "metadata": { "type":"dashboard", "version": "1" },
  "allowedgroups": [ "teachergroup" ],
  "title":"Teacher dashboard",
  "get": {
    "request": {
      "parameters": []
    }
  },
  "panels":[
    { "title":"My chart", "resource":"mylargesearchform", "row":"1", "width":"12" },
    { "title":"My chart", "resource":"myfirstchart", "row":"2", "width":"6" },
    { "title":"My table", "resource":"myinfobox", "row":"2", "width":"6" },
    { "title":"My table", "resource":"mysecondchart", "row":"3", "width":"6" },
    { "title":"My table", "resource":"myimageuploadform", "row":"3", "width":"6" },
    { "title":"My table", "resource":"mythirdchart", "row":"4", "width":"6" }
  ]
}
{% endhighlight %}
