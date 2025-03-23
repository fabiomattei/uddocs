---
layout: page
name: Controller
---

UD allows to use an entire controller as a json resource to use in a page as a component. This means that a controller
can be inserted in a grid as a usual resource and this gives a lot of flexibility to the framework as the resources approach
and the more usual Controller/View approach can merge together.

In order to achieve that we need to define a <a href="{{site.baseurl}}/docs/controller">controller</a> as usual and then 
we need to wrap this controller as a json resource. This will allow UD to use this as a json resource from now on.

### Example

{% highlight json %}
{
  "name": "my-controller-block",
  "metadata": { "type":"controller", "version": "1" },
  "allowedgroups":  [ "reader", "writer" ],
  "get": {
    "request": {
      "parameters": []
    },
    "controller": {
      "name" : "FabioM\\MyProject\\Chapters\\Users\\Controllers\\UserList",
      "templateFile": "emptyapptemplate"
    }
  }
}
{% endhighlight %}

This controller will behave as usual:
1. the code in the getRequest or the postRequest will be run; usually that implies to get parameters and run some DAO in order
to load data from the database
2. the related view file will be filled with content coming from the controller and shown

### Be careful

In order to activate this new controller we need to have both:

* controller file linked in the **index_controllers** list
* relative json resource linked in the **index_json_resource** list 

The two items needs to have different names.

