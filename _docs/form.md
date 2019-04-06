---
layout: page
name: Form
---

# Structure

A form resource allows the programmer to insert a form in an application. 
A form is a complex structure. A form often need to be filled in advance with data coming from the database.
Then it send POST data through a POST request and after that some logic is executed in order to update the 
system.

## GET section 

### Request

As we said before, often we need to load data from the database in order to fill the form.
In case we need to send some data to the SQL query we can do that using GET data.


### Query


### Form

## POST section

### Request

### Transactions

### Notifications

## Complete example

{% highlight json %}
{
  "name": "sampleform",
  "metadata": { "type":"form", "version": "1" },
  "allowedgroups": [ "administrationgroup", "teachergroup", "managergroup" ],
  "get": {
    "request": {
      "parameters": [
        { "type":"long", "validation":"required|numeric", "name":"assetid" }
      ]
    },
    "query": {
      "sql": "SELECT ua_id, ua_competentperson, ua_conditionsummary, ua_identifier, ua_description, ua_comment, ta_id, ta_parentid, ta_name FROM unitasset U JOIN asset a on U.ua_assetid = a.ta_id WHERE ua_id = :id;",
      "parameters":[
        { "type":"long", "placeholder": ":id", "getparameter": "assetid" }
      ]
    },
    "form": {
      "title": "",
      "submitTitle": "Save",
      "fields": [
        { "type":"textfield", "name":"name", "label":"Name", "placeholder":"Name", "sqlfield":"ta_name", "width":"6", "row":"1" },
        { "type":"textfield", "name":"plantcode", "label":"Plant Code", "placeholder":"Plant Code", "sqlfield":"ua_identifier", "width":"6", "row":"1" },
        { "type":"textarea", "name":"description", "label":"Description", "placeholder":"Description", "sqlfield":"ua_description", "width":"12", "row":"2" },
        { "type":"textfield", "name":"condition", "label":"Condition", "placeholder":"Condition", "sqlfield":"ua_conditionsummary", "width":"6", "row":"3" },
        { "type":"textfield", "name":"competentperson", "label":"Competent person", "placeholder":"Competent person", "sqlfield":"ua_competentperson", "width":"6", "row":"3" },
        { "type":"textarea", "name":"comment", "label":"Comment", "placeholder":"Comment", "sqlfield":"ua_comment", "width":"12", "row":"4" },
        { "type":"hidden", "name":"unitassetid", "sqlfield":"ua_id", "row":"4" },
        { "type":"hidden", "name":"technicalassetid", "sqlfield":"ta_id", "row":"4" }
      ]
    }
  },
  "post": {
    "request": {
      "postparameters": [
        { "name":"name", "validation":"required|max_len,250" },
        { "name":"plantcode", "validation":"max_len,250" },
        { "name":"description", "validation":"alpha_numeric" },
        { "name":"condition", "validation":"alpha_numeric" },
        { "name":"competentperson", "validation":"alpha_numeric" },
        { "name":"comment", "validation":"alpha_numeric" },
        { "name":"unitassetid", "validation":"required|integer" },
        { "name":"technicalassetid", "validation":"required|integer" }
      ]
    },
    "transactions": [
      {
        "sql":"UPDATE unitasset SET ua_competentperson = :competentperson, ua_conditionsummary = :condition, ua_identifier = :plantcode, ua_description= :description, ua_comment = :comment WHERE ua_id=:unitassetid;",
        "parameters":[
          { "type":"string", "placeholder": ":competentperson", "postparameter": "competentperson" },
          { "type":"string", "placeholder": ":condition", "postparameter": "condition" },
          { "type":"string", "placeholder": ":plantcode", "postparameter": "plantcode" },
          { "type":"string", "placeholder": ":description", "postparameter": "description" },
          { "type":"string", "placeholder": ":comment", "postparameter": "comment" },
          { "type":"long", "placeholder": ":unitassetid", "postparameter": "unitassetid" }
        ]
      },
      {
        "sql":"UPDATE asset SET ta_name = :name WHERE ta_id=:technicalassetid;",
        "parameters":[
          { "type":"string", "placeholder": ":name", "postparameter": "name" },
          { "type":"long", "placeholder": ":technicalassetid", "postparameter": "technicalassetid" }
        ]
      }
    ],
    "notifications": [
      {
        "body":[ "name", "amount" ],
        "type":"new Subscription to ski class",
        "destinationgroups": [ "administrationgroup", "teachergroup", "managergroup" ],
        "action": { "action":"documentinfo", "resource":"documentsubscriptionv1", "parameters":[{"name":"id", "value":"id"}] }
      }
    ]
  }
}
{% endhighlight %}

If you want to know about the [Validation](https://bitbucket.org/fabiomattei/esb/wiki/Validation) check the related page.