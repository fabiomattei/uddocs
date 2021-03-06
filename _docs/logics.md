---
layout: page
name: Logics
---

# Logics

Logics are script that allow the user to update the state of the system that include:

* update database
* update session variables
* send updating messages to other users

At the end of the script the user is redirected to another page or to a page previously open.

The script is divided in two sections: GET and POST. The part of the script that refers to the request made to the server is executed.

The contents of the two sections have the same structure.

### Request

We use this section in order to list all parameters expected by this script and to express the validation bounds they need to respect.
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
        { "type":"long", "validation":"required|numeric", "name":"bookid" }
      ]
    },
    "transactions": [
      {
        "sql":"DELETE FROM mylibrary WHERE id=:id;",
        "parameters":[
          { "placeholder": ":id", "getparameter": "bookid" }
        ]
      }
    ],
    "sessionupdates": [
      { "type":"long", "sessionparameter":"lastbookdeletedid", "getparameter":"bookid" }
    ],
    "redirect": {
      "internal": { "type": "onepageback" }
    }
  },
  "post": {
    "request": {
      "postparameters": [
        { "type":"long", "validation":"required|numeric", "name":"bookid" }
      ]
    },
    "transactions": [
      {
        "sql":"DELETE FROM mylibrary WHERE id=:id;",
        "parameters":[
          { "placeholder": ":id", "postparameter": "bookid" }
        ]
      }
    ],
    "sessionupdates": [
      { "type":"long", "sessionparameter":"lastbookdeletedid", "postparameter":"bookid" }
    ],
    "redirect": {
      "internal": { "type": "onepageback" }
    }
  }
}
{% endhighlight %}
