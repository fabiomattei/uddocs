---
layout: page
name: Chartjs
---

# Description

Charts are often requested in a project management application. This brought us to integrate <a href="https://www.chartjs.org/" alt="ChartJS library">Chart.js</a> natively in our framework.
Please feel free to check out the <a href="https://www.chartjs.org/" alt="ChartJS library">Chart.js documentation</a> in order to understand what that is for. In the next few lines we are just going to describe how UD leverages it.

## Information needed in order to set up a table

### Get parameters

Sometimes we need to pass to the query behind the chart some parameter using a get request. 

Example: www.example.com/mytable?parentid=2302

In order to catch that get parameter you need to add an object in the parameters section. That object has tree properties:

* type: the type (long, string, etc..) expected the parameter to take after validation
* validation: the rules the parameter has to follow
* name: the parameter name in the URL

{% highlight json %}
"parameters": [
  { "type":"integer", "validation":"required|integer", "name":"parentid" }
]
{% endhighlight %}
If you want to know about the <a href="{{site.baseurl}}/docs/validation">Validation</a> check out the related page.

### Query

We need to make a query to the datase in order to populate our table.
The simplest thing to do is just to write the query in plain SQL and eventually connect the parameters needed from the GET section.

{% highlight json %}
"query": {
  "sql": "select id, typeid, name, description FROM mytable WHERE parentid = :parentid;",
  "parameters":[
    { "type":"long", "placeholder": ":parentid", "getparameter": "parentid" }
  ]
}
{% endhighlight %}

As you can see the SQL parameter is inserted in the query using a placeholder: *:parametername*
The SQL parameter is connected to the GET parameter using: "getparameter": "parentid"
We can insert as many paremeters as we need.

If you want to know more about SQL paramenters check out the <a href="{{site.baseurl}}/docs/query">Query</a> page.

### Chart Structure

The chart structure is basically taken from the structure the <a href="https://www.chartjs.org/" alt="ChartJS library">Chart.js library</a> requires.
Nothing is added by ugly duckling.

It is important to note that sometimes there are placeholder in the stucture. You can recognise them because they are strings starting with a #.

Here there are two placeholders but they could be more as the structure is recursively checked from UD and you are free to set all the placeholders you need.

What are the placeholder for? We need them in order to **glue** the data resulting from the query to the chart.

Here you can see the two placehoders:

* #labels
* #amounts

We will understand better how they relate to the database query in the next paragraph. For now have a look to the **chart structure**.

{% highlight json %}
"chart": {
  "type":"bar",
  "data": {
    "labels": "#labels",
    "datasets": [{
      "label": "# of Articles",
      "data": "#amounts",
      "backgroundColor:":[
        "rgba(255, 99, 132, 0.2)",
        "rgba(54, 162, 235, 0.2)",
        "rgba(255, 206, 86, 0.2)",
        "rgba(75, 192, 192, 0.2)",
        "rgba(153, 102, 255, 0.2)",
        "rgba(255, 159, 64, 0.2)"
      ],
      "borderColor": [
        "rgba(255,99,132,1)",
        "rgba(54, 162, 235, 1)",
        "rgba(255, 206, 86, 1)",
        "rgba(75, 192, 192, 1)",
        "rgba(153, 102, 255, 1)",
        "rgba(255, 159, 64, 1)"
      ],
      "borderWidth": "1"
    }]
  },
  "options": {
    "scales": {
      "yAxes": [{
        "ticks": {
          "beginAtZero":true
        }
      }]
    }
  }
}
{% endhighlight %}

# Chart Data Glue

We previously said that the **chart structure** allows the presence of placeholders in order to kwno where to put results from the query:

* #labels
* #amounts

The SQL query is the following:
{% highlight sql %}
select count(id) counted, created FROM articles GROUP BY created;
{% endhighlight %}

{% highlight json %}
"chartdataglue":[
  { "type":"string", "placeholder":"#labels", "sqlfield":"created" },
  { "type":"long", "placeholder":"#amounts", "sqlfield":"counted" }
]
{% endhighlight %}

# Complete example

{% highlight json %}
{
  "name": "articleschartv1",
  "metadata": { "type":"chartjs", "version": "1" },
  "allowedgroups": [ "author" ],
  "get": {
    "request": {
      "parameters": []
    },
    "query": {
      "sql":"select count(id) counted, created FROM articles GROUP BY created;",
      "parameters":[]
    },
    "chartdataglue":[
      { "type":"string", "placeholder":"#labels", "sqlfield":"created" },
      { "type":"long", "placeholder":"#amounts", "sqlfield":"counted" }
    ],
    "chart": {
      "type":"bar",
      "data": {
        "labels": "#labels",
        "datasets": [{
          "label": "# of Articles",
          "data": "#amounts",
          "backgroundColor:":[
            "rgba(255, 99, 132, 0.2)",
            "rgba(54, 162, 235, 0.2)",
            "rgba(255, 206, 86, 0.2)",
            "rgba(75, 192, 192, 0.2)",
            "rgba(153, 102, 255, 0.2)",
            "rgba(255, 159, 64, 0.2)"
          ],
          "borderColor": [
            "rgba(255,99,132,1)",
            "rgba(54, 162, 235, 1)",
            "rgba(255, 206, 86, 1)",
            "rgba(75, 192, 192, 1)",
            "rgba(153, 102, 255, 1)",
            "rgba(255, 159, 64, 1)"
          ],
          "borderWidth": "1"
        }]
      },
      "options": {
        "scales": {
          "yAxes": [{
            "ticks": {
              "beginAtZero":true
            }
          }]
        }
      }
    }
  }
}
{% endhighlight %}
