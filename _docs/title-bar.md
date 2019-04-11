---
layout: page
name: Title bar
---

It is possible to insert a Title bar in a page in order to show a big title to the user.
This bar is usually used in a dashboard.

The only thing wi need to set is the string we want to show and, eventually, the buttons we want to put close to the title.

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
