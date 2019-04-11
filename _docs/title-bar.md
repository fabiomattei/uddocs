---
layout: page
name: Title bar
---

It is possible to insert a Title bar in a page in order to show a big title to the user.
This bar is usually used in a dashboard.

The only thing wi need to set is the string we want to show and, eventually, the buttons we want to put close to the title.

The title bar object contains two items:

* title: the string containing the title we want to add to the page
* actions: list of actions we need to put in the page on the same line the title is (optional)

If you want to know more about the <a href="{{site.baseurl}}/docs/actions">Actions</a> check out the related page.

{% highlight json %}
{
  "name": "unitlisttitlebar",
  "metadata": { "type":"titlebar", "version": "1" },
  "allowedgroups": [ "centralheadofficegroup" ],
  "get": {
    "request": {
    },
    "titlebar": {
      "title": "This is the title I want to insert in this page",
      "actions": [
        { "label": "My button to click", "resource": "theresourceiwanttolink", "buttoncolor":"btn-success", "outline": false, "parameters":[] }
      ]
    }
  }
}
{% endhighlight %}
