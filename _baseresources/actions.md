---
layout: page
name: Actions
---

# Actions 

Actions are links in un UD application.
It is usually structured as un array of objects, each object is an action (or a link).

The properties of the action object are:

* type: "link" or "button"
* label: the descriptive text associated to the link the user can read
* resource: the linked resource
* tooltip: the descriptive text associated to the link the user can read when the mouse pointer is over the link
* onclick: once the user clicks the link a window appear that contiains the onclick text, if user clicks ok the action go on otherwise the action stops and the user goes back to prevous page
* buttoncolor: the color associated to the button. They come out of bootstrap classes, possible values are: [ gray, blue, pink, green, red, yellow ]
* outline: if true the button is an outline button
* parameters: parameters linked to the action
* parameter_assign_symbol: usullay it is **=** but in some case it can be important to be able to personalize it
* parameter_separator: usually it is **&** but in some case it can be important to be able to personalize it

# Action parameters

Each action can have one or many parameters.
A parameter is a json object that contains two properties

The name property is mandatory:

* name: the parameter name

One of the following parameters is mandatory:

* sqlfield: the name of the field the value is taken from
* constantparameter: a constant parameter
* getparameter: the name of the get parameter the value is taken from
* postparemeter: the name of the post parameter the value is taken from

# Complete Example

{% highlight json %}
"actions": [
    { "type": "link",
	  "label": "Info", 
      "resource": "myinfopanel",
      "tooltip": "My tool tip text",
      "onclick": "My on click text",
      "buttoncolor": "greens",
      "outline": false,
      "parameters":[
        {"name": "id", "sqlfield": "id"},
        {"name": "secondid", "constantparameter": "3"},
        {"name": "thirdid", "getparameter": "mygetparameter"}
      ] 
    }
    { "type": "link", "label": "Export", "resource": "myexport", "parameters":[] },
    { "type": "link", "icon": "pencil", "resource": "myeditform", "parameters":[] }
]
{% endhighlight %}

