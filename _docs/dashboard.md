---
layout: page
name: Dashboard
---

# Dashboard

A dashboard is a container of resources that allows the programmer to compose a page with more then one resource.
The page is designed using bootstrap framework and for each resource we need to specify the row where we want to put it and how many columns we want the resource to take.

A dashboard contains an array of **panels**

For each panel we need to specify:

* title: is going to be shown in the interface
* resource: the identifier of the json resource in the system
* row: which row cotains the specific panel
* width: how many columns we want the panel to extend

We do not need to provide parameter to this specific resource as the GET parameters are calculated recoursively from the contained resources. We expect the action that link this dashboard to provide all parameters required by all resources contained in it

### Complete example


{% highlight json %}
{
  "name": "centralheadofficedashboard",
  "metadata": { "type":"dashboard", "version": "1" },
  "allowedgroups": [ "centralheadofficegroup" ],
  "title":"Manager dashboard",
  "get": {
    "request": {
      "parameters": []
    }
  },
  "panels":[
    { "title":"My chart", "resource":"selectriskcenterform", "row":"1", "width":"12" },
    { "title":"My chart", "resource":"top10risks", "row":"2", "width":"6" },
    { "title":"My table", "resource":"top10risksbyriskrating", "row":"2", "width":"6" },
    { "title":"My table", "resource":"top10rootcauses", "row":"3", "width":"6" },
    { "title":"My table", "resource":"top10consequences", "row":"3", "width":"6" },
    { "title":"My table", "resource":"unitheatmap", "row":"4", "width":"6" }
  ]
}
{% endhighlight %}
