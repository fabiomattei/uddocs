---
layout: page
name: Index
---

# Index

Json files are located in a tree of directories the programmer is free to compose.
It is important for the programmer to show to UD how to locate the <a href="{{site.baseurl}}/docs/resource">resources</a> files that compose the system.
The **index.json** files does exactly that.

Files are listed in a **scripts** array.
Each file has three fields:

* path: where the resource is located
* type: the type of resource, possible values are: [index, table, datatable, chartjs, form, info, group, dashboard, logic, titlebar, tabbedpage]
* name: this field is unique by resource, it allows the system to identify a resource between all resources that are part of the system. Index reources do not need a name.

It is important to rember that from the **root index.json** file it is possible to link more index files. This way we try to divide indexes by context and to avoid to have a single enormous index file.

Indexes are followed recoursively so UD is able to create an index based on the **name** field in order to map every resource. 

### Example of a index resource

{% highlight json %}
{ 
  "name": "index",
  "metadata": { "type":"index", "version": "1" },
  "scripts": [
    { "path":"json/mytables/index.json", "type":"index" },
    { "path":"json/myform/index.json", "type":"index" },
    { "path":"json/globaldashboard.json", "type":"dashboard", "name":"globaldashboard" },
    { "path":"json/info/myinfopanel.json", "type":"info", "name":"myinfopanel" },
    { "path":"json/info/mytabbedpage.json", "type":"tabbedpage", "name":"mytabbedpage" }
  ]
}
{% endhighlight %}
