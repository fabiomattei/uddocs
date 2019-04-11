---
layout: page
name: Index
---

# Index

Json files are located in a tree of tirectories the programmaer is free to compose.
It is important for the programmer to show to UD how to locate the files with all resources that compose the system.
The **index.json** files does exactly that.

Files are listed in a **scripts** array.
Each file has three fieds:

* path: where the resource is located
* type: the type of the resource, possible values are: [index, table, datatable, chartjs, form, info, group, dashboard, logic, titlebar, tabbedpage]
* name: this field is unique by resource, it allows the system to identify a resource between all resources that are part of the system. Index reources do not need a name.

It is important to rember that from the **root index.json** file it is possible to link more index files. This way we try to divide indexes by context and to avoid to have a single enourmous index file.

Indexes are followed recoursively.


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
