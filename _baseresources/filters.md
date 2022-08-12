---
layout: page
name: Filters
---

## Basic idea

A filter is a structure that allows the developer to filter and modify an output of a database query, or of a get parameter or of a value o any type in order to to some change before to show it to the user.

Let's say that I have a table and I need to write a list of purchases having product name, quantity, price in euro and date and I want to show the euro simbol close to the price.

{% highlight json %}
{
  "name": "purchasestable",
  "metadata": { "type":"table", "version": "1" },
  "allowedgroups": [ "buyer" ],
  "get": {
    "request": {
      "parameters": []
    },
    "query": {
      "sql": "SELECT name, quantity, price, mydate FROM purchases;",
      "parameters":[]
    },
    "table": {
      "title": "My purchases",
      "fields": [
        {"headline": "Name", "sqlfield": "name"},
        {"headline": "Quantity", "sqlfield": "quantity"},
		{"headline": "Price", "sqlfield": "price", "filter":"euro"},
        {"headline": "Date", "sqlfield": "mydate", "filter":"mysqltohumandate"}
      ]
    }
  }
}
{% endhighlight %}

As you can see the database field are filtered using filters:

* euro
* mysqltohumandate


## What's happening behind the scenes

In order to implement a filter it is needed to define a function having the same name. That function is going to be called when the filter is activated.

{% highlight php %}
class CustomPageStatus extends PageStatus {

    /**
     * This method can be overridden for each project implementation.
     *
     * @param array $filtercall: contains the function call and the parameters to call
     * @param string $value: contains the value we need to apply the filter to
     *
     * Ex: substr: $filtercall = [ 'substr', 2, 5 ]    =>    $value = substr($value, 2, 5)
     *     substr: $filtercall = [ 'substr', 7 ]       =>    $value = substr($value, 7)
     *
     * This function can be overridden and customized
     */
    function applyFilter(array $filtercall, string $value): string {
        if ( $filtercall[0] == 'substr' AND count( $filtercall ) == 3 ) return substr( $value, $filtercall[1], $filtercall[2] );
        if ( $filtercall[0] == 'substr' AND count( $filtercall ) == 2 ) return substr( $value, $filtercall[1] );
        if ( $filtercall[0] == 'mysqltohumandate' ) return date ('d/m/Y', strtotime( $value ) );
        if ( $filtercall[0] == 'euro' ) return number_format($value, 0, ".", ",");

        return $value;
    }
}
{% endhighlight %}

## We can pipe filters

It is possible 

## We can pass parameters to filters

