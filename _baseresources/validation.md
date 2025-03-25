---
layout: page
name: Validation
---

# Validation

The properties *get_validation_rules*, *get_filter_rules*, *post_validation_rules*, and *post_filter_rules* allow the user to set the validation and flter rules for a GET or a POST request. 

UD uses the <a href="https://github.com/Wixel/GUMP">GUMP library</a> in order to implement validation correclty.
Here we are going to give an introducation but please feel free to refer to <a href="https://github.com/Wixel/GUMP">GUMP library documentation</a>
in order to have a complete view of all possibilities.

The validation section of a resource checks the content of a set o paramters that can be send through a GET request or ghruogh a POST requeste.

In case of a get request we need to deal with something like this:

{% highlight json %}
"request": {
  "parameters": [
    { "validation":"required|integer", "name":"id" }
  ]
}
{% endhighlight %}

Here we are saying two thigs:

1. we expect this GET request to have a parameter named id
2. we expect this parameter to be a required interger number. 

This is a list of all many type of validation we can apply to a parameter but feel free to refer to <a href="https://github.com/Wixel/GUMP">GUMP library documentation</a> in order to have more information.

* boolean             can contain [true, false]
* integer             can contain an integer number
* onlynumeric         can contain a number integer or float (with comma or dots)
* onlyalpha           can contain only alphabetical characters (even with accents) and ? , : ; . ! @ € £ $ & + = * _ \r \n \t
* alpha_numeric       it is a combination of onlyalpha and onlynumeric
* exactlen            followed by a number is the exact allowed lenght of the passed parameter
* min_len             followed by a number is the minimum allowed lenght of the passed parameter
* max_len             followed by a number is the maximum allowed lenght of the passed parameter
* minnumeric          followed by a number n the filled content need to be greater then n
* maxnumeric          followed by a number n the filled content need to be less then n
* calendardate        it has to have the format dd/mm/yyyy
* mysqldate           it has to have the format yyyy-mm-dd
* required            the field is mandatory
* checkbox            the field could be missing and that would not give an error, a check box must be followed by a type ex: integer or alphanumerical

Example of usages

* "required\|integer"                       the parameter is a required integer number

* "required\|alphanumerical\|max_len250"    the parameter is required, alphanumerical and has a maximum allowed lenght of 250



