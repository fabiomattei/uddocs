---
layout: page
name: Content Block
title: Content Block
---

## Description

A **content block** is a resource that renders static authored content — headings, paragraphs, lists, tables, and images — without requiring a SQL query. The content is written once in the visual editor and stored in the `ud_content_blocks` database table. At render time the framework reads the stored content and outputs HTML.

Content blocks are useful whenever you need to place editorial content alongside data-driven blocks on a page: introductory text above a table, an image next to a chart, a section heading inside a grid.

---

## Database table

Before using content blocks you must create the following table in your database:

{% highlight sql %}
CREATE TABLE ud_content_blocks (
  cb_id      INT AUTO_INCREMENT PRIMARY KEY,
  cb_name    VARCHAR(255) NOT NULL UNIQUE,
  cb_type    VARCHAR(50)  NOT NULL,
  cb_content JSON         NOT NULL,
  cb_updated DATETIME,
  cb_created DATETIME
);
{% endhighlight %}

---

## Resource file

A content block resource file is minimal. It declares the resource name and type. The actual content lives in the database row whose `cb_name` matches the resource `name`.

{% highlight json %}
{
  "name": "intro-paragraph",
  "metadata": { "type": "contentblock", "version": "1" },
  "allowedgroups": [ "admingroup" ]
}
{% endhighlight %}

Register the file path in your resource index and register the template class:

{% highlight php %}
$jsonResourceTemplates['contentblock'] =
    \Fabiom\UglyDuckling\Framework\ContentBlocks\ContentBlockJsonTemplate::class;
{% endhighlight %}

---

## Block sub-types

### heading

Renders an HTML heading tag (`<h1>` – `<h6>`).

{% highlight json %}
{ "level": 2, "text": "Welcome to the dashboard" }
{% endhighlight %}

| Property | Type | Description |
|---|---|---|
| `level` | integer 1–6 | Heading level |
| `text` | string | Heading text |

---

### paragraph

Renders a `<p>` tag.

{% highlight json %}
{ "text": "This section shows the latest activity across all projects." }
{% endhighlight %}

| Property | Type | Description |
|---|---|---|
| `text` | string | Paragraph text |

---

### list

Renders a `<ul>` or `<ol>` tag with `<li>` items.

{% highlight json %}
{ "ordered": false, "items": ["First item", "Second item", "Third item"] }
{% endhighlight %}

| Property | Type | Description |
|---|---|---|
| `ordered` | boolean | `true` for a numbered list, `false` for a bullet list |
| `items` | array of strings | List items |

---

### statictable

Renders a `<table>` with fixed headers and rows.

{% highlight json %}
{
  "headers": ["Name", "Role", "Location"],
  "rows": [
    ["Alice", "Developer", "Rome"],
    ["Bob",   "Designer",  "Milan"]
  ]
}
{% endhighlight %}

| Property | Type | Description |
|---|---|---|
| `headers` | array of strings | Column header labels |
| `rows` | array of arrays | Each inner array is one row of cell values |

---

### image

Renders an `<img>` tag.

{% highlight json %}
{ "src": "uploads/images/banner.jpg", "alt": "Project banner", "cssclass": "img-fluid" }
{% endhighlight %}

| Property | Type | Description |
|---|---|---|
| `src` | string | Path to the image file (relative to the application public root) |
| `alt` | string | Alternative text for accessibility |
| `cssclass` | string | CSS class(es) applied to the `<img>` tag |

---

## Using a content block inside a grid

Place the content block resource name in a grid panel exactly like any other resource:

{% highlight json %}
{
  "name": "homepage-grid",
  "metadata": { "type": "grid", "version": "1" },
  "allowedgroups": [ "admingroup" ],
  "cssclass": "row",
  "panels": [
    { "id": "panel1", "cssclass": "col-12",   "resource": "intro-paragraph" },
    { "id": "panel2", "cssclass": "col-md-8", "resource": "articlestable" },
    { "id": "panel3", "cssclass": "col-md-4", "resource": "sidebar-image" }
  ]
}
{% endhighlight %}

---

## Editing content

Content blocks are authored through the <a href="{{site.baseurl}}/docs/visual-editor">Visual Editor</a>. You can also insert or update a row directly in the database:

{% highlight sql %}
INSERT INTO ud_content_blocks (cb_name, cb_type, cb_content, cb_updated, cb_created)
VALUES (
  'intro-paragraph',
  'paragraph',
  '{"text": "Welcome to the system."}',
  NOW(), NOW()
)
ON DUPLICATE KEY UPDATE
  cb_type    = VALUES(cb_type),
  cb_content = VALUES(cb_content),
  cb_updated = NOW();
{% endhighlight %}
