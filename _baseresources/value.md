---
layout: page
name: Value
---

# Value

The possible properties of the **value** object are:

* sqlfield: used in case we need to load data in a field that comes from a query
* sessionparameter: used in case we need to load data in a field that comes from a session parameter
* getparameter: used in case we need to load data in a field that comes from a getparameter parameter
* postparameter: used in case we need to load data in a field that comes from a postparameter parameter
* constant: used in case we need to load data in a field that comes from a constantparameter parameter
* composite: used in in a column we want to put more than one field
* filter: the filter functions to apply to the field.

{% highlight json %}
{
    "composite":"${placeholder1} ${placeholder2}", "parameters": [
      { "name":"${placeholder1}", "sqlfield": "name"  },
      { "name":"${placeholder2}", "sqlfield": "surname"  }
    ] 
}
{% endhighlight %}
