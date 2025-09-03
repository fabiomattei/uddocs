---
layout: page
name: Ajax buttons
title: Ajax buttons
---

Ajax buttons are part of a trik that allows the user to implement *one page applications using UD*.

The main idea is based on a tag named *TagAjaxButton* that extends BaseHTMLTag. The Ajax button allows UD to load asyncronously resources  in different part of the applcation without the need ot reloading the whole page. In order to work properly the ajax buttons need the file udajaxactivator.js properly linked in the HTML template.

## Delete button

This button, if clicked, makes a request (GET or POST) to a resource and removes a part of the DOM from the interface.

{% highlight json %}
{
  "type": "ajaxbutton", 
  "label": "Del Threat", 
  "cssclass": "udbuttonremove", 
  "method": "GET",
  "dataudurl": {
    "controller": "partial", 
    "resource": "bowtie-delete-model-threat-barrier-ajax", 
    "parameters": [
      {"name": "field", "constantparameter": "proactivebarriers"},
      {"name": "mid", "getparameter": "resid"}
    ]
  },
  "dataudiddestination": "#mycontainer"
}
{% endhighlight %}

As we can see this button needs a type (ajax button) and a css class. The class is the things that gives life to this button and is important that is **udbuttonremove** because this class is needed in the javascript side in order to activate the button.

We can specify the method: GET or POST.

The resource we need to make the request to is specified in the **dataurl** side of the object. This is an <a href="{{site.baseurl}}/baseresources/actions">action</a> and has the  usual action properties. You can specify the controller (usually is partial) the resource that contains the things to implement and so on.

The **dataudiddestination** specifies the DOM element that we want to delete from the interface. The element is specified using a css selector, in this example, once the request return with success the DOM element having the HTML id property  set as mycontainer will be deleted

## Empty button

This button, if clicked, makes a request (GET or POST) to a resource and removes all contents of a part of the DOM from the interface.

{% highlight json %}
{
  "type": "ajaxbutton", 
  "label": "Empty Threat", 
  "cssclass": "udbuttonempty", 
  "method": "GET",
  "dataudurl": {
    "controller": "partial", 
    "resource": "empty-model-ajax", 
    "parameters": [
      { "name": "field", "constant": "barriers"},
      { "name": "mid", "getparameter": "resid", "description": "model id"}
    ]
  },
  "dataudiddestination": "#mycontainer"
}
{% endhighlight %}

As we can see this button needs a type (ajax button) and a css class. The class is the things that gives life to this button and is important that is **udbuttonremove** because this class is needed in the javascript side in order to activate the button.

We can specify the method: GET or POST.

The resource we need to make the request to is specified in the **dataurl** side of the object. This is an <a href="{{site.baseurl}}/baseresources/actions">action</a> and has the usual action properties. You can specify the controller (usually is partial) the resource that contains the things to implement and so on.

The **dataudiddestination** specifies the DOM element that we want to empty. The element is specified using a css selector.

## Load and overwrite button

This button, if clicked, makes a request (GET or POST) to a resource and **replace the respource loaded in the section of the DOM** pointed from **dataudiddestination**.

{% highlight json %}
{
  "type": "ajaxbutton",
  "label": "Create new barrier",
  "cssclass": "udbuttonoverwrite",
  "method": "GET",
  "dataudurl": {
    "controller": "partial",
    "resource":"bowtie-partial-add-proactive-barrier-form",
    "parameters": [
      {"name": "field", "constantparameter": "proactivebarriers"},
      {"name": "resid", "getparameter": "resid"}
    ]
  },
  "dataudiddestination": "#addProactiveBarrierFormContainer"
}
{% endhighlight %}

As we can see the type of the button is always ajaxbutton but now we are linking a different class to this specific button: udbuttonoverwrite.
This class allows UD to activate the javascript section of the button.

We can specify the method: GET or POST.

The resource we need to make the request to is specified in the **dataurl** object property. This is an <a href="{{site.baseurl}}/baseresources/action">action</a> and has the usual action properties. It is important the controller is **partial**. Partial stays for *partial interface* it means that the reousource linked will be loaded without the template of the interface. We do not need a whole HTML page here, we need only a section to load in the DOM.

## Load and append  button

Same as above, the only difference it that this button append the loaded resource to the DOM specifyed element.

{% highlight json %}
{ 
  "type": "ajaxbutton", 
  "label": "Link barrier", 
  "cssclass": "udbuttonappend",
  "method": "GET",
  "dataudurl": { 
    "controller": "bowtieudbuttonsendpoint",
    "parameters": [
      {"name": "field", "constantparameter": "proactivebarriers"},
      {"name": "mid", "getparameter": "resid"}
    ]
  },
  "dataudiddestination": "#practivebarriers"
}
{% endhighlight %}

## Button for ajax request

All buttons seen above allow the user to perform a single specific action. Sometimes we need to perform many different changes in the interface when a button is clicked. Then we need the button ajax request and the ajax reponse.

A button for an ajax request is not much different from the buttons defined previously.

It need a css class named **udbuttonajaxrequest** in oder to be activated by the framework and need a **dataurl** field to call. The resource linked in the **dataurl** field will contain the **Ajax response** object that is going to give a list of actions to perform.

{% highlight json %}
{
  "type": "ajaxbutton",
  "label": "Add this proactive barrier to this threat and to database",
  "cssclass": "udbuttonajaxrequest",
  "method": "POST",
  "dataudurl": {
    "controller": "partial",
    "resource":"bowtie-partial-add-proactive-barrier-form",
    "parameters": [
      {"name": "field", "constantparameter": "proactivebarriers"},
      {"name": "resid", "getparameter": "resid"}
    ]
  }
}
{% endhighlight %}

The button can **link a form**, using the property *dataudformid*, in this case all form fields are sent with the request.

{% highlight json %}
{
  "type": "ajaxbutton",
  "label": "Add this proactive barrier to this threat and to database",
  "cssclass": "udbuttonajaxrequest",
  "method": "POST",
  "dataudformid": "bowtie-partial-add-proactive-barrier-form",
  "dataudurl": {
    "controller": "partial",
    "resource":"bowtie-partial-add-proactive-barrier-form",
    "parameters": [
      {"name": "field", "constantparameter": "proactivebarriers"},
      {"name": "resid", "getparameter": "resid"}
    ]
  }
}
{% endhighlight %}

## Ajax reponses

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

# HTML

Ajax buttons are connected to an HTML represantation. 

## Button Remove

This class adds an event to a DOM element (it could be a button or a link or whaterver you need) this button will empty a certain target in the DOM defined by his **id** and if a url is designed it is going to call a fetch function to the designed URL.

{% highlight html %}
<button class="udbuttonremove" data-udiddestination="#mydestination" data-udurl="myapp.com?var1=A&var2=B" data-udmethod="GET">Remove</button>
{% endhighlight %}



* data-udiddestination: destination of the remove in the DOM. It is going to be used in a document.querySelector(args) call.
* data-udurl: URL called when the button is pushed
* data-udmethod: method used in order to call URL: GET or POST

## Button Empty

{% highlight html %}
<button class="udbuttonempty" data-udiddestination="#mydestination" data-udurl="myapp.com?var1=A&var2=B" data-udmethod="GET">Remove</button>
{% endhighlight %}

## Button Append

{% highlight html %}
<button class="udbuttonappend" data-udiddestination="#mydestination" data-udurl="myapp.com?var1=A&var2=B" data-udmethod="GET">Remove</button>
{% endhighlight %}

## Button Overwrite

{% highlight html %}
<button class="udbuttonoverwrite" data-udiddestination="#mydestination" data-udurl="myapp.com?var1=A&var2=B" data-udmethod="GET">Remove</button>
{% endhighlight %}


## Button Ajax Request

{% highlight html %}
<button class="udbuttonajaxrequest" data-udiddestination="#mydestination" data-udurl="myapp.com?var1=A&var2=B" data-udmethod="GET">Remove</button>
{% endhighlight %}


