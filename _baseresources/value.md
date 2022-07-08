---
layout: page
name: Value
---

# Value

Every time we need a value taken from the status of the page we need to use a **value object**.

This object is goint to look up for what we need and give it back where we need. Things we are lookgin for could be session parameters, get parameters, costant and more. 

The possible properties of the **value** object are:

* sqlfield: used in case we need a value stored in a field that comes from a query
* sessionparameter: used in case we need a value stored in a session parameter
* getparameter: used in case we need a value stored in a getparameter parameter
* postparameter: used in case we need a value stored in a postparameter parameter
* constant: used in case we need define a costant
* composite: if we need to build a value as concatenation of different values
* filter: if we need to apply a filter of some sort to a value before to give it back

### Examples:

{% highlight json %}
{
    "sqlfield":"username"
}
{% endhighlight %}

This value object get the field **username** from the results of a query.

{% highlight json %}
{
    "getparameter":"userid"
}
{% endhighlight %}

This value object get the content of the get parameter **userid**.

{% highlight json %}
{
    "composite":"${placeholder1} ${placeholder2}", "parameters": [
      { "name":"${placeholder1}", "sqlfield": "name"  },
      { "name":"${placeholder2}", "sqlfield": "surname"  }
    ] 
}
{% endhighlight %}

This value object get the fields **name** and **surname** from the results of a query and creates a concatenation overwriting ${placeholder1} with the value stored in the field **name** and ${placeholder2} with the value stored in the field **surname**.

