---
layout: page
name: HTML Block
---

# Description

An HTML Block is a PHP Object that extends the <a href="https://github.com/fabiomattei/uglyduckling/blob/master/src/Common/Blocks/BaseHTMLBlock.php">BaseHTMLBlock php class</a>.

This class gives the structure to create an HTML block. A Block is a subsection of a HTML page. It could be a form, a table or a list. 
It is not ment to be a basic HTML tag but a section of a page big enough to be visible from the user as a unity of content.

Any HTML structure can be formalised as a block.

A <a href="https://github.com/fabiomattei/ud-demo/blob/master/src/HTMLBlocks/HTMLBlockExample.php">basic block</a> example is visible in the <a href="https://github.com/fabiomattei/ud-demo">ud-demo</a> repository.

{% highlight php %}
use Fabiom\UglyDuckling\Common\Blocks\BaseHTMLBlock;

class HTMLBlockExample extends BaseHTMLBlock {
	
	const HTML_BLOCK_NAME = 'basehtmlblockexample';

    function getHTML(): string {
        return '<p>Paragraph example</p>';
    }

}
{% endhighlight %}

In order to define an HTML Block you need tow things:

* define a HTML_BLOCK_NAME constat that is unique for this type of block in the codebase;
* define a getHTML method that returns the HTML code eventually loaded with content coming from a database.


