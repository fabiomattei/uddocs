---
layout: page
name: Session updates
title: Session updates
---

Session variables are important in a web application because they allow us to save some user data we can use later on.
UD allows us to define a json structure in order to set or update a session variable.

### Sintax

A sessionupdates json structure is a json object having two fields:

* queryset: allows us to define one or more queries;
* sessionvars: allows us to define the session vars.

{% highlight json %}
"sessionupdates": {
  "queryset": [
    {
      "label": "query1",
      "sql": "SELECT usr_siteid FROM user where usr_id = :usrid ;",
      "parameters":[
        { "type":"long", "placeholder": ":usrid", "sessionparameter": "user_id" }
      ]
    }
  ],
  "sessionvars": [
      { "name":"user_id", "system":"ud" },
      { "name":"username", "system":"ud" },
      { "name":"group", "system":"ud" },
      { "name":"logged_in", "system":"ud" },
      { "name":"ip", "system":"ud" },
      { "name":"last_login", "system":"ud" },
      { "name":"siteid", "sqlfield":"usr_siteid", "querylabel":"query1" },
      { "name":"tryaconstantparameter", "constantparamenter":"4" }
  ]
}
{% endhighlight %}

### queryset

* label (optional if there is only one query): allows us to identify the query
* sql: sql statement
* parameters (optional): eventual sql parameter to pass to query

### sessionvars

Each session variable we are going to define has a name. The varible can be set using information returned by a query and in this case we are going to spcify the *querylabel* and the *sqlfield*. The variable can be set using a getparameter, a postparameter (it exists if in the POST section), a constant or a composite field.

A filter can be applied when we define a session.

* name: the name of the session variable to update
* querylabel: refeerence to the label field defined in the query set
* sqlfield: used in case we need a value stored in a field that comes from a query
* getparameter: used in case we need a value stored in a getparameter parameter
* postparameter: used in case we need a value stored in a postparameter parameter
* constant: used in case we need define a costant
* composite: if we need to build a value as concatenation of different values
* filter: if we need to apply a filter of some sort to a value before to give it back


