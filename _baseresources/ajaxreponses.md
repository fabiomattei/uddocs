---
layout: page
name: Ajax reponses
title: Ajax reponses
---

The ajax response is usually in the post section of a resource.

We can have 5 types of responses:

* delete
* append
* overwrite
* appendurl
* overwriteurl

Let's see a quick example:

{% highlight json %}
"ajaxreponses" : [
  { 
    "type": "delete", 
    "destination": "#addProactiveBarrierFormContainer"
  },
  { 
    "type": "append",
    "destination": "#practivebarriers",
    "body": { 
      "composite": "<tr><td>${name}</td><td>${surname}</td></tr>",
      "parameters": [ 
        { "name":"${surname}", "postparameter": "surname" },
        { "name":"${name}", "postparameter": "name" }  
      ] 
    }
  }
]
{% endhighlight %}

In this example we have two ajax actions to implement.

* the first one is a **delete action** and it means that we need to delete whatever is the DOM element having id addProactiveBarrierFormContainer.
* the second one is an **append action** and it means that we need to append the content, defined in the body section, in the destination: practivebarriers. The content of the body could be a simple string but, as we need to create a pattern containing two post parameters, we are using a <a href="{{site.baseurl}}/baseresources/value">value</a> object.

Let's see another example:

{% highlight json %}
"ajaxreponses" : [
  { 
    "type": "appendurl",
    "destination": "#myexperts",
	"position": "beforeend",
    "url": { 
      "controller": "partial", 
      "resource": "user-tables", 
      "method": "GET"
      "parameters": [
        { "name": "usergroup", "constantparameter": "ajaxexperts" },
        { "name": "cityid", "postparameter": "cityid" }
      ] 
    }
  }
]
{% endhighlight %}

In this exaample the **ajaxreponses** contains an **appendurl action**. What is going to happen is that the framework is goning to make a GET call to the controller **partial** asking for the resource **user-tables** passing as parameters the usergroup named *ajaxexperts* and the city id that comes from the post parameter *cityid*. Once the HTML will be returned from the framework this will be added in append in the DOM element having id *myexperts*.

The position attribute allows the programmer to specify where in the dom element put the new content: 

* beforebegin: Before the element;
* afterbegin: Inside the element, before its first child;
* beforeend: Inside the element, after its last child;
* afterend: After the element.

The **overwriteurl** ajax response is used in the same way.

It is possible to use <a href="{{site.baseurl}}/baseresources/value">value</a> objects also for the **destination** section as you can see in the following example.

{% highlight json %}
"ajaxreponses" : [
  { 
    "type": "delete", 
    "destination": {
      "composite":  "#addProactiveBarrierFormContainer${tid}", 
      "parameters": [ 
        { "name":"${tid}", "postparameter": "tid"  } 
      ] 
    }
  },
  { 
    "type": "append",
    "destination": { 
      "composite": "#practivebarriers${tid}", 
      "parameters": [ { "name":"${tid}", "postparameter": "tid"  } ] 
    },
    "body": { 
      "composite": "<tr><td>${name}</td><td>${surname}</td></tr>",
      "parameters": [ 
        { "name":"${surname}", "postparameter": "surname" },
        { "name":"${name}", "postparameter": "name" }  
      ]
    }
  }
]
{% endhighlight %}
