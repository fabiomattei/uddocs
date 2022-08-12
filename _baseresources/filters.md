---
layout: page
name: Filters
---

## Basic idea

A filter is a structure that allows the developer to filter and modify an output of a database query, or of a get parameter or of a value of any type in order to to some change before to show it to the user. This basically means that for each filter we call for a specific value, a function is called and appied to the value itself. The value returned by the filter will be used in the process later on.

Let's say that I have a table and I need to write a list of purchases having product name, quantity, price in euro and date and I want to show the euro simbol close to the price. All this information comes from a mysql table and I need to work a litle with it before to show it to the user. The date has to change from MySql format YYYY-MM-DD to human format d/m/Y and, as I am based in Europe, in the price filed we need exchange the dot for the comma. 

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

When UD finds a filter call it calls the function **applyFilter** in **PageStatus**. applyFilter checks what to do with the filter call and in this case it calls the function number_format when it finds the filter euro and it calls date('d/m/Y', strtotime( $value ) ) when it finds the filter mysqltohumandate:

* euro: number_format($value, 0, ".", ",")
* mysqltohumandate: date('d/m/Y', strtotime( $value ) )

You can see the complete code here.

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

It is possible to apply to a value a sequence of filters where the output of a filter becomes the input of the following one. In order to achieve that we need to use the pipe symbol.

Let's say that we want to color the prices in a way that each price over 100 euros is bold.

I need to extend the custom page status like this:

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
        if ( $filtercall[0] == 'eurocolor' ) return ( $value > 100 ? '<b>'.$value.'</b>' : $value );

        return $value;
    }
}
{% endhighlight %}

Then I can use both the euro filter and the eurocolor filter using this notation:

{% highlight json %}
{"headline": "Price", "sqlfield": "price", "filter":"euro|eurocolor"}
{% endhighlight %}

## We can pass parameters to filters

In the previous example we hardcoded the number we need in order to hilight the prices in the code. But we know that magic numbers are an ugly ugly solution to a problem. 

UD allows us to use parameter separated by commas like this:

{% highlight json %}
{"headline": "Price", "sqlfield": "price", "filter":"euro|eurocolor,100"}
{% endhighlight %}

The first filter called is **euro** and then the output of this filter is sent to the filter **eurocolor** followed by the parameter that allows us to determine what is the number treshold.

In order to implement this we need to do a little change in the **applyFilter** function:

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
        if ( $filtercall[0] == 'eurocolor' ) return ( $value > $filtercall[1] ? '<b>'.$value.'</b>' : $value );

        return $value;
    }
}
{% endhighlight %}

Basically each call to a filter translates in an array named $filtercall where the first item is the filter name and the following items are the filter parameter.
If I use the filter "substr,2,5" this will be translated in $filtercall = [ 'substr', 2, 5 ] and then applyed as substr( $value, 3, 5 ).

This allows maximum flexibility.

