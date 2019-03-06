---
layout: page
name: Actions
---

# Logics

{% highlight json %}
{ 
  "name": "deletereportv1",
  "metadata": { "type":"logic", "version": "1" },
  "allowedgroups": [ "administrationgroup", "teachergroup", "managergroup" ],
  "get": {
    "request": {
      "parameters": [
        { "type":"integer", "validation":"required|integer", "name":"id" }
      ]
    },
    "transactions": [
      {
        "sql":"DELETE from requestv1 WHERE id=:id;",
        "parameters":[
          { "placeholder": ":id", "getparameter": "id" }
        ]
      }
    ]
  }
}
{% endhighlight %}
