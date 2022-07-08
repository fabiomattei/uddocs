---
layout: page
name: Ajax buttons
---

# Ajax buttons

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
      {"name": "mid", "getparameter": "btmid"}
    ]
  },
  "dataudiddestination": "threatstablecontainer"
}
{% endhighlight %}

As we can see this button needs a type (ajax button) and a css class. The class is the things that gives life to this button and is important that is **udbuttonremove** because this class is needed in the javascript side in order to activate the button.

We can specify the method: GET or POST.

The resource we need to make the request to is specified in the **dataurl** side of the object. This is an <a href="{{site.baseurl}}/baseresources/actions">action</a> and has the  usual action properties. You can specify the controller (usually is partial) the resource that contains the things to implement and so on.

The **dataudiddestination** specifies the DOM element that we want to delete from the interface. 


## Load and  delete button

This button, if clicked, makes a request (GET or POST) to a resource and **replace the respource loaded in the section of the DOM** pointed from **dataudiddestination**.

{% highlight json %}
{
  "type": "ajaxbutton",
  "label": "Create new barrier",
  "cssclass": "udbuttonload",
  "dataudurl": {
    "controller": "partial",
    "resource":"bowtie-partial-add-proactive-barrier-form",
    "parameters": [
      {"name": "field", "constantparameter": "proactivebarriers"},
      {"name": "btmid", "getparameter": "btmid"}
    ]
  },
  "dataudiddestination": "addProactiveBarrierFormContainer"
}
{% endhighlight %}

As we can see the type of the button is always ajaxbutton but now we are linking a different class to this specific button: udbuttonload.
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
  "dataudurl": { 
    "controller": "bowtieudbuttonsendpoint",
    "parameters": [
      {"name": "field", "constantparameter": "proactivebarriers"},
      {"name": "mid", "getparameter": "btmid"}
    ]
  },
  "dataudiddestination": "practivebarriers"
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
      {"name": "btmid", "getparameter": "btmid"}
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
      {"name": "btmid", "getparameter": "btmid"}
    ]
  }
}
{% endhighlight %}

## Ajax  reponses

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
