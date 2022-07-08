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

The resource we need to make the request to is specified in the **dataurl** side of the object. This is an <a href="{{site.baseurl}}/baseresources/action">action</a> and has the  usual action properties. You can specify the controller (usually is partial) the resource that contains the things to implement and so on.

The **dataudiddestination** specifies the DOM element that we want to delete from the interface. 


## Load and  delete button

{% highlight json %}
{
  "type": "button",
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

## Load and append  button

{% highlight json %}
{ 
  "type": "button", 
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

## Ajax  reponses

{% highlight json %}
"ajax" : [
  { "type": "delete", "destination": {
    "composite":  "#addProactiveBarrierFormContainer${tid}", "parameters": [ { "name":"${tid}", "postparameter": "tid"  } ] }
  },
  { "type": "add",
    "destination": { 
      "composite": "#practivebarriers${tid}", 
	  "parameters": [ { "name":"${tid}", "postparameter": "tid"  } ] 
    },
    "body": { 
	  "composite": "<tr id=\"btmbrow${btmb_bid}\"><td class=\"editable ecl-table__cell\" data-type=\"select\" data-url=\"bowtiejinplaceendpoint.html?field=proactivebarrier&amp;id=${btmb_bid}\" data-loadurl=\"bowtiejinplacefillingendpoint.html?field=proactivebarriers\">${name}</td><td><button data-udiddestination=\"btmbrow${btmb_bid}\" data-udurl=\"bowtieudbuttonsendpoint.html?field=proactivebarrierremove&amp;mid=${btmb_bid}&amp;tid=0\" class=\"udbuttonremove\" data-udmethod=\"GET\" data-activated=\"activated\">Del</button></td></tr>",
      "parameters": [ 
        { "name":"${btmb_bid}", "returnedid": "firstinsert" },
        { "name":"${name}", "postparameter": "name" }  
      ] 
	}
  }
]
{% endhighlight %}
