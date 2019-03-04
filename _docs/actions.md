---
layout: page
name: Actions
---

# Actions 

Actions are link in un UD application.
It is usually structured has un array of objects, each object is an action.

The properties of the action object are:

* label: the descriptive text associated to the link the user can read
* resource: the linked resource
* tooltip: the descriptive text associated to the link the user can read when the mouse pointer is over the link
* onclick: once the user clicks the link a window appear that contiains the onclick text, if user clicks ok the action go on otherwise the action stops and the user goes back to prevous page
* buttoncolor: the color associated to the button
* outline: if true the button is an outline button
* parameters: parameters linked to the action

# Action parameters

Each action can have one or many parameters.
A parameter is a json object that contains two properties

The name property is mandatory:

* name: the parameter name

One of the following parameters is mandatory:

* sqlfield: the name of the field the value is taken from
* constantparameter: a constant parameter
* getparameter: the name of the get parameter the value is taken from

# Complete Example

```
#!json
"actions": [
    { "label": "Info", 
      "resource": "inforequestv1",
      "tooltip": "My tool tip text",
      "onclick": "My on click text",
      "buttoncolor": "btn-success",
      "outline": false,
      "parameters":[
        {"name": "id", "sqlfield": "usr_id"},
        {"name": "secondid", "constantparameter": "3"},
        {"name": "thirdid", "getparameter": "mygetparameter"}
      ] 
    }
    { "label": "Export", "resource": "requestexportv1", "parameters":[] },
    { "icon": "pencil", "resource": "requestexportv1", "parameters":[] }
]
```

