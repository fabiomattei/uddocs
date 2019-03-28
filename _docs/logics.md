---
layout: page
name: Actions
---

# Logics

Logics are script that allows the user to update the state of the system that include:

* update database
* update session variables
* send upadtnig messages to other users

At the end of the script the user is redirected to another page or to a page previously open.

The script is divided in two sections: GET and POST. The part of the script that refers to the request made to the server is executed.

The contents of the two sections are the same

### Request

In this section we describe the parameters expected by this script and the validation bounds they need to respect.
Check out the <a href="{{site.baseurl}}/docs/validation">Validation</a> page in order to have a better description.

### Transactions

This array contains a set of queries we can send to the database in order to update it.
Check out the <a href="{{site.baseurl}}/docs/transaction">Transactions</a> page in order to have a better description.

### Session updates 

This array contains instructions in order to update the session variables.
Check out the <a href="{{site.baseurl}}/docs/sessionupdates">Session Updates</a> page in order to have a better description.

### Complete example

{% highlight json %}
{
  "name": "updatesiteid",
  "metadata": { "type":"transaction", "version": "1" },
  "allowedgroups": [ "administrationgroup", "teachergroup", "managergroup" ],
  "get": {
    "request": {
      "parameters": [
        { "type":"long", "validation":"required|numeric", "name":"riskcenterid" }
      ]
    },
    "transactions": [
      {
        "sql":"DELETE from requestv1 WHERE id=:id;",
        "parameters":[
          { "placeholder": ":id", "getparameter": "id" }
        ]
      }
    ],
    "sessionupdates": [
      { "type":"long", "sessionparameter":"siteid", "getparameter":"riskcenterid" }
    ],
    "redirect": {
      "internal": { "type": "onepageback" }
    }
  },
  "post": {
    "request": {
      "postparameters": [
        { "type":"long", "validation":"required|numeric", "name":"riskcenterid" }
      ]
    },
    "transactions": [
      {
        "sql":"DELETE from requestv1 WHERE id=:id;",
        "parameters":[
          { "placeholder": ":id", "getparameter": "id" }
        ]
      }
    ],
    "sessionupdates": [
      { "type":"long", "sessionparameter":"siteid", "postparameter":"riskcenterid" }
    ],
    "redirect": {
      "internal": { "type": "onepageback" }
    }
  }
}
{% endhighlight %}
