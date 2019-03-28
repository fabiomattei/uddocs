---
layout: page
name: Validation
---

This class check the content of a set o fields to see if they contina what they are ment to, possible filters are:

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
"required|integer"                      the parameter is a required integer number
"required|alphanumerical|max_len250"     the parameter is required, alphanumerical and has a maximum allowed lenght of 250
