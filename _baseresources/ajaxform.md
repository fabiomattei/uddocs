---
layout: page
name: "Ajax form"
---

# Ajax form

## Ajax forms

It is possible to create a form the insted of triggering a reload of the whole page, it triggers an ajax response

The **get** section of this resource looks a lot like a normal form in UD.

The main difference between an ajax forma a UD form is that in the section **bottombuttons** we find two ajax buttons:
* the first one to send the form as a post call
* the second one to cancel the action

The **post** section of this resource contains a ajax response that gives life to the whole thing.

{% highlight json %}
{
  "name": "my-form-resource-name",
  "metadata": { "type":"djform", "version": "1" },
  "allowedgroups": [ "my-user-group" ],
  "templatefile": "application",
  "get": {
    "request": {
      "parameters": [
        { "type":"string", "validation":"required|alpha_numeric_dash", "name":"resid" }
      ]
    },
    "form": {
      "formid": "add-cons-form",
      "title": "Add a new article",
      "action": { "resource":"my-form-resource-name" },
      "method": "POST",
      "fields": [
        { "type":"sqldropdown", "name":"secondresid", "label":"", "width":"6", "row":"2",
          "query": {
            "sql": "SELECT btc_id, btc_prjid, btc_code, btc_name FROM bt_consequences WHERE btc_prjid = :prjid;",
            "parameters":[
              { "type":"string", "placeholder": ":prjid", "sessionparameter": "prjid"}
            ]
          },
          "valuesqlfield": "btc_id",
          "labelsqlfields": ["btc_code", "btc_name"]
        },
        { "type":"hidden", "name":"resid", "getparameter":"resid", "row":"2" }
      ],
      "bottombuttons": [
        {
          "type":"ajaxbutton",
          "cssclass":"udbuttonajaxrequest btn btn-outline-secondary btn-sm",
          "label":"Add article",
          "method": "POST",
          "dataudformid":"add-cons-form",
          "dataudiddestination": "conequencebarriersrowblock",
          "dataudurl":{
            "controller": "partial",
            "resource":"my-form-resource-name"
          },
          "width":"2", "row":"3"
        },
        {
          "type": "ajaxbutton",
          "label": "Cancel",
          "cssclass": "udbuttonempty btn btn-outline-secondary btn-sm",
          "dataudiddestination": "#consequencesbuttonbarcontainer"
        }
      ]
    }
  },
  "post" : {
    "request": {
      "postparameters": [
        { "type":"string", "validation":"required|alpha_numeric_dash", "name":"resid" },
        { "type":"string", "validation":"required|alpha_numeric_dash", "name":"secondresid" }
      ]
    },
    "usecases" : [
      { "name": "MyEventualUseCaseName", "parameters": [
        { "type":"string", "name":"resid", "postparameter":"resid" },
        { "type":"string", "name":"secondresid", "postparameter":"secondresid" }
      ] }
    ],
    "ajaxreponses": [
      {
        "type": "appendurl",
        "destination": "#panel5",
        "method": "GET",
        "url": {
          "controller": "partial",
          "resource": "bowtie-consequences-barriers-table",
          "parameters": [
            {"name": "resid", "postparameter": "resid"},
            {"name": "secondresid", "postparameter": "secondresid"}
          ]
        }
      },
      {
        "type": "empty",
        "destination":  "#consequencesbuttonbarcontainer"
      }
    ],
    "error": { "destination": "#panel5", "position": "afterbegin" }
  }
}
{% endhighlight %}

