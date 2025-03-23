---
layout: page
name: Code
---

Sometimes it is needed to include in our resource structure a new HTML component. It this new component contains just html
without the need of making a query to the database, if it is basically just an include of HTML code it is possible
to define a **code resource**.

### Example

{% highlight json %}
{
  "name": "my-new-code-block",
  "metadata": { "type":"code", "version": "1" },
  "allowedgroups":  [ "reader", "writer" ],
  "get": {
    "request": {
      "parameters": []
    },
    "code": {
      "bodyfile" : "Code/MyCodeBlock/body.html",
      "headfile" : "Code/MyCodeBlock/addToHead.html",
      "footfile" : "Code/MyCodeBlock/addToFoot.html",
      "headoncefile" : "",
      "footoncefile" : ""
    }
  }
}
{% endhighlight %}

As you can see in this example we are defining a strcture that allows the include of different files.

All this file are contained in a folder starting from the constant **TEMPLATES_DIRECTORY** that usually points to **src/Templates/**.

As we can see we are including many different files let's see what is the logic behind this include.

* **bodyfile**: it contains the main part of the HTML we need to include, the part that need to be visualized in the page.
* **headfile**: it contains a section that will be added in the HEAD section of the document. It is going to be added once 
for each code block we are going to include in a page. This can be useful if we have CSS to include that is going to change
from block to block
* **footfile**: it contains a section that will be added before closing the BODY of the document. It is going to be added 
once for each code block we are going to include in a page. This can be useful if we have Javascript to include that is going 
to change from block to block.
* **headoncefile**: it contains a section that will be added in the HEAD section of the document. It is going to be added 
once for all code blocks we are going to include in a page. This can be useful if we have CSS to include that is common between
all block.
* **footoncefile**: it contains a section that will be added closing the BODY of the document. It is going to be added 
once for all code blocks we are going to include in a page. This can be useful if we have javascript to include that is common 
between all blocks.

Not all the files need to be specified.

