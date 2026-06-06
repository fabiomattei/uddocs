---
layout: page
name: Visual Editor
title: Visual Editor
---

## Description

The **Visual Editor** is a browser-based tool for composing pages without writing JSON by hand. You build a grid layout by adding panels to a canvas, assign a block to each panel, fill in its properties through a form, and save everything with a button click.

The editor supports two categories of blocks:

* **Content blocks** — heading, paragraph, list, static table, image. Content is stored in the `ud_content_blocks` database table.
* **Data blocks** — table, form, chart, info panel, dashboard, grid. Configuration is stored as a JSON resource file on disk.

---

## Setup

### 1. Create the content blocks table

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

### 2. Register the content block template

Add the content block template class to your `$jsonResourceTemplates` index so the framework can render content blocks:

{% highlight php %}
$jsonResourceTemplates['contentblock'] =
    \Fabiom\UglyDuckling\Framework\ContentBlocks\ContentBlockJsonTemplate::class;
{% endhighlight %}

### 3. Route to the editor controller

In your application bootstrap or router, instantiate `EditorController`, wire it up, and call `showPage()`:

{% highlight php %}
use Fabiom\UglyDuckling\Framework\Editor\EditorController;

$editor = new EditorController();
$editor->setPageStatus($pageStatus);
$editor->setResourceIndex($resourceIndex);
$editor->setResourcesBasePath('/var/www/myapp/public/');
$editor->makeAllPresets($dbconnection, $logger, $securityChecker, $mailer);
$editor->showPage();
{% endhighlight %}

`setResourcesBasePath()` sets the base directory used for image uploads. Uploaded images are placed in `{basePath}/uploads/images/`.

---

## Editor layout

The editor opens as a full-page interface divided into three panels.

| Panel | Purpose |
|---|---|
| **Left — Palette** | Lists all available block types grouped by category. Click a block type to assign it to the selected panel. |
| **Centre — Canvas** | Displays the grid panels. Click a panel to select it. Click **✕** to remove it. Click **+ Add panel** to add a new one. |
| **Right — Properties** | Shows configuration fields for the block assigned to the selected panel. Click **Save block** to persist the block. |

The header bar contains a resource name field, a **Load** button, and a **Save page** button.

---

## Workflow

### Opening an existing grid

Type the resource name into the header field and click **Load** (or press Enter). The canvas populates with the panels defined in that grid resource file.

### Creating a new grid

Type a new resource name into the header field. The resource must already be registered in your resource index (pointing to a file path that may or may not exist yet). Build the layout by adding panels, assign blocks, save each block, then click **Save page** to write the grid JSON file.

### Adding a panel

Click **+ Add panel** on the canvas. You will be prompted for:

* **CSS class** — the Bootstrap column class for this panel (e.g. `col-md-6`).
* **Resource name** — the name of the resource this panel will render. For a content block, this name must be registered in your resource index. For a data block, it must also be registered.

### Assigning a block to a panel

Click a panel on the canvas. The properties panel on the right shows a list of all block types. Click the one you want. The properties form appears immediately.

### Configuring a block

Fill in the fields in the properties panel. Field types vary by block:

* **Text / Textarea** — type directly.
* **Select** — choose from the dropdown list.
* **Checkbox** — tick for true, untick for false.
* **Image** — click the file picker. The image is uploaded immediately and the path is stored automatically.
* **List of items** — click **+ Add item** to add a row, **✕** to remove one.
* **List of objects** (e.g. table columns, form fields) — click **+ Add row** to add a sub-object, fill in each sub-field, **✕** to remove a row.

Click **Save block** when done. For content blocks this writes a row to `ud_content_blocks`. For data blocks this writes the JSON resource file to disk.

### Saving the grid

Click **Save page** in the header. This writes the grid JSON resource file containing the panel layout.

---

## API endpoints

The editor communicates with the server via the same URL that rendered the page. The following actions are available:

| Method | Parameters | Description |
|---|---|---|
| GET | `action=list` | Returns a JSON array of all resource names registered in the index |
| GET | `action=load&res={name}` | Returns the JSON content of a resource file |
| POST | `action=save-resource` | Writes a resource JSON file to disk |
| POST | `action=save-content-block` | Inserts or updates a row in `ud_content_blocks` |
| POST | `action=upload-image` | Uploads an image file, returns the stored path |

---

## Security

The editor uses the same session authentication as the rest of the application. `EditorController` calls `makeAllPresets()` which validates the session via your configured `SecurityChecker`. Only logged-in users with a valid session can access the editor.

Resource writes are validated against the registered resource index: a `save-resource` call for a name not present in the index is rejected with an error response.

Image uploads are validated by file type and MIME type. Accepted formats are JPEG, PNG, and GIF.

---

## Related pages

* <a href="{{site.baseurl}}/resources/content-block">Content Block</a> — reference for all content block sub-types and their JSON structure
* <a href="{{site.baseurl}}/resources/resource">Resource index</a> — how to register resources in your application
* <a href="{{site.baseurl}}/docs/page-grid">Page (Grid layout)</a> — how grids are rendered at request time
