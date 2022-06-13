---
layout: page
name: Session updates
---

# Session updates

This array contains instructions in order to update the session variables.

### Sintax of a single object contained in the array

{% highlight json %}
{ 
  "type":"long", 
  "sessionparameter":"siteid", 
  "getparameter":"riskcenterid"
}
{% endhighlight %}

* type: it is the type of variable is expected to update the session variable
* sessionparameter: the name of the session variable to update
* getparameter: the name of the get parameter we can get the value from in order to update to the session parameter
* postparameter: the name of the post parameter we can get the value from in order to update to the session parameter

### Example in a GET request

{% highlight json %}
"sessionupdates": [
  { "type":"long", "sessionparameter":"siteid", "getparameter":"riskcenterid" }
]
{% endhighlight %}



### Example in a POST request

{% highlight json %}
"sessionupdates": [
  { "type":"long", "sessionparameter":"siteid", "postparameter":"riskcenterid" }
]
{% endhighlight %}


