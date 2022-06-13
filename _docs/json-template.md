---
layout: page
name: Json Templates
---

Json Templates are the back bone of UD. Their job is to read a json resource, and set ad HTMLBlock to give back to the page builder.

UD defines few default templates you can start from but it is possible to create your personal templates.

In order to do that you need to extend the <a href="https://github.com/fabiomattei/uglyduckling/blob/master/src/Common/Json/JsonTemplates/JsonTemplate.php">JsonTemplate class</a> and define at least two things:

* **const blocktype** is the block type json resource will be referring to
* **function createHTMLBlock** that returns the block well setted from the JsonTemplate

This is an essential script.

{% highlight php %}
<?php
use Fabiom\UglyDuckling\Common\Json\JsonTemplates\JsonTemplate;
use Fabiom\UglyDuckling\Custom\HTMLBlocks\HTMLBlockExample;

class JsonTemplateExample extends JsonTemplate {

    const blocktype = 'templatebuilderexample';

    /**
     * @return \Fabiom\UglyDuckling\Common\Blocks\EmptyHTMLBlock|HTMLBlockExample
     */
    public function createHTMLBlock() {
        return new HTMLBlockExample;
    }

}
{% endhighlight %}

The constructor of the base <a href="https://github.com/fabiomattei/uglyduckling/blob/master/src/Common/Json/JsonTemplates/JsonTemplate.php">JsonTemplate class</a> requires two parameters: 

* ApplicationBuilder $applicationBuilder;
* PageStatus $pageStatus;

{% highlight php %}
public function __construct( $applicationBuilder, $pageStatus ) {
    $this->applicationBuilder = $applicationBuilder;
    $this->pageStatus = $pageStatus;
}
{% endhighlight %}

These are the bridge between the JsonTemplate class and the rest of the application.

