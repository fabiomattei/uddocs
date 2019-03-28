---
layout: page
name: Info
---

This json file allows the user to create an info panel, a panel that basically queryes information from the database and write that information in a panel.
Possible fields are:

* textfield
* textarea
* currency
* date
* dropdown

{% highlight json %}
{
  "name": "technicalassetinfo",
  "metadata": { "type":"info", "version": "1" },
  "allowedgroups": [ "administrationgroup", "teachergroup", "managergroup" ],
  "get": {
    "request": {
      "parameters": [
        { "type":"integer", "validation":"required|integer", "name":"assetid" }
      ]
    },
    "query": {
      "sql": "select * FROM asset WHERE ta_id = :id;",
      "parameters":[
        { "type":"long", "placeholder": ":id", "getparameter": "assetid" }
      ]
    },
    "info": {
      "title": "My new info panel",
      "fields": [
        { "type":"textarea", "validation":"max_len,2500", "name":"name", "label":"Name", "placeholder":"Name", "value":"name", "width":"6", "row":"1" },
        { "type":"currency", "validation":"required|numeric", "name":"amount", "label":"Amount", "placeholder":"10.0", "value":"amount", "width":"6", "row":"1" },
        { "type":"date", "validation":"max_len,10", "name":"duedate", "label":"Due date", "value":"duedate", "width":"12", "row":"2" }
      ]
    }
  }
}
{% endhighlight %}
