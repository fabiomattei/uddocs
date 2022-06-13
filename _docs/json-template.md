---
layout: page
name: Json Templates
---

UD defines few default templates you can start from but it is possible to create your personal templates.

{% highlight php %}
<?php

namespace Fabiom\UDDemo\JsonTemplates;

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
