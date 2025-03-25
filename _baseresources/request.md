---
layout: page
name: Request
---

# Request

In a web application it is possible to have two kinds of requests: GET or POST

## GET

{% highlight json %}
"request": {
  "parameters": [
    { "validation":"required|integer", "name":"id" }
  ]
}
{% endhighlight %}

The parameter can be one of the following types:

* integer or long
* string

If you want to know about the <a href="{{site.baseurl}}/docs/validation">Validation</a> check out the related page.

## POST

{% highlight json %}
"request": {
  "postparameters": [
    { "validation":"required|integer", "name":"idform" },
    { "validation":"alphanumerical", "name":"name" }
  ]
}
{% endhighlight %}

The parameter can be one of the following types:

* integer or long
* string

If you want to know about the <a href="{{site.baseurl}}/docs/validation">Validation</a> check out the related page.
