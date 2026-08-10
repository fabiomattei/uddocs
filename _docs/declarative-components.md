---
layout: page
name: Declarative leaf components
---

# Description

`BaseFormComponent`, `BaseInfoComponent`, and `BaseTableComponent` are `BaseComponent` subclasses that render a common panel shape from a declarative `$fields` array instead of a hand-written `render()`. Use them for the majority of panels — forms, read-only detail views, and lists — and reserve a raw `BaseComponent` for layouts these three don't cover.

All the usual `BaseComponent` mechanics — `get_request()`/`post_request()`, GET/POST validation rules, `$postSuccessUrl`/`$postSuccessMessage`, `$allowedGroups`, `executeSelectQuery()`/`executeWriteQuery()` — work exactly the same on these subclasses. Only `render()` is already implemented for you.

---

## BaseFormComponent

Renders a `<form>` from a list of fields, grouped into rows via the optional `row` key (default `1`).

{% highlight php %}
use Fabiom\UDDemo\Components\BaseFormComponent;

class ArticleEdit extends BaseFormComponent {

    protected string $title  = 'Edit Article';
    protected string $action = '';
    protected string $formId = 'article-edit-form';

    protected array $get_validation_rules = ['art_id' => 'required|max_len,36'];
    protected array $get_filter_rules     = ['art_id' => 'trim'];

    protected array $post_validation_rules = [
        'art_id'    => 'required|max_len,36',
        'art_title' => 'required|max_len,200',
        'art_body'  => 'required',
    ];
    protected array $post_filter_rules = [
        'art_title' => 'trim|sanitize_string',
        'art_body'  => 'trim',
    ];

    protected array $fields = [
        ['type' => 'hidden', 'name' => 'art_id'],
        ['type' => 'text',     'name' => 'art_title', 'label' => 'Title', 'width' => 12,
            'attributes' => ['required' => 'required']],
        ['type' => 'textarea', 'name' => 'art_body',  'label' => 'Body',  'width' => 12, 'row' => 2],
        ['type' => 'submit',   'name' => 'save',      'label' => 'Save',  'row' => 3],
    ];

    public string $postSuccessMessage = 'Article updated';

    public function __construct() {
        $this->postSuccessUrl = 'articles-list.html';
    }

    protected function entity(): array {
        if (empty($this->getParameters)) {
            return [];
        }
        $rows = $this->executeSelectQuery(
            "SELECT art_id, art_title, art_body FROM articles WHERE art_id = :art_id",
            $this->getParameters
        );
        return $rows[0] ?? [];
    }

    protected function post_request(): void {
        $this->executeWriteQuery(
            "UPDATE articles SET art_title = :art_title, art_body = :art_body WHERE art_id = :art_id",
            $this->postParameters
        );
    }

}
{% endhighlight %}

Override `entity()` (not `get_request()`) to supply the values the form pre-fills; `get_request()` already calls it for you.

### Field reference

Every entry describes one field.

| Key | Type | Description |
|---|---|---|
| `type` | string | `text` · `number` · `password` · `textarea` · `select` · `checkbox` · `radio` · `hidden` · `file` · `date` · `time` · `info` · `raw` · `submit`. Defaults to `text`. |
| `name` | string | Input `name`/`id` attribute and the key looked up in the entity array. |
| `label` | string | Label text (not used by `hidden`, `raw`, `submit`). |
| `value` | callable | `fn(array $entity): string`. Takes priority over the entity lookup. |
| `default` | string | Fallback used when the entity has no value for `name` and no `value` callable is given. |
| `width` | int | Bootstrap column width, emitted as `col-md-{width}`. |
| `row` | int | Groups fields onto the same visual row. Defaults to `1`. |
| `attributes` | array | Extra HTML attributes rendered verbatim, e.g. `['placeholder' => '…', 'required' => 'required']`. |
| `options` | array | `['value' => 'label', …]` — only for `select`. |
| `optionvalue` | string | The submitted value for a `checkbox`/`radio`. Defaults to `'1'`. |
| `checked` | callable | `fn(array $entity): bool` — only for `checkbox`/`radio`. |
| `raw` | bool | For `info`/`raw` fields, skip `htmlspecialchars()` escaping. |
| `html` | string\|callable | For `raw` fields: literal HTML, or `fn(array $entity): string`. |

The form automatically adds `enctype="multipart/form-data"` when any field is `type => 'file'`, and (for `POST` forms) the hidden `_component` and `csrftoken` fields needed by the page's <a href="{{site.baseurl}}/docs/component#routing-post-submissions-to-the-right-component">POST dispatcher</a> and CSRF check.

---

## BaseInfoComponent

Renders a read-only field list — the detail-view counterpart to `BaseFormComponent`.

{% highlight php %}
use Fabiom\UDDemo\Components\BaseInfoComponent;

class ArticleInfo extends BaseInfoComponent {

    protected string $title = 'Article details';

    protected array $get_validation_rules = ['art_id' => 'required|max_len,36'];
    protected array $get_filter_rules     = ['art_id' => 'trim'];

    protected array $fields = [
        ['type' => 'text',      'label' => 'Title',   'name' => 'art_title', 'width' => 12],
        ['type' => 'date',      'label' => 'Created',  'name' => 'art_created', 'width' => 6],
        ['type' => 'paragraph', 'label' => 'Body',      'name' => 'art_body',    'width' => 12, 'row' => 2],
    ];

    protected function entity(): array {
        if (empty($this->getParameters)) {
            return [];
        }
        $rows = $this->executeSelectQuery(
            "SELECT art_id, art_title, art_body, art_created FROM articles WHERE art_id = :art_id",
            $this->getParameters
        );
        return $rows[0] ?? [];
    }

}
{% endhighlight %}

### Field reference

| Key | Type | Description |
|---|---|---|
| `type` | string | `date`, `paragraph`, and `raw` get distinct rendering (see below); every other value (`text`, `textarea`, `currency`, or anything else) renders identically — a label above the value, with no type-specific formatting. Defaults to `text`. |
| `label` | string | Label shown above the value (not used by `paragraph`/`raw`). |
| `name` | string | Key looked up in the entity array. |
| `value` | callable | `fn(array $entity): string`. Takes priority over the entity lookup. |
| `default` | string | Fallback when the entity has no value for `name`. |
| `width` | int | Emitted as `col-md-{width}`. |
| `row` | int | Groups fields onto the same visual row. Defaults to `1`. |
| `cssclass` | string | Extra CSS classes appended to the field's wrapper `<div>`. |
| `raw` | bool | Skip `htmlspecialchars()` escaping. |
| `html` | string\|callable | For `raw` fields: literal HTML, or `fn(array $entity): string`. |

`type => 'date'` reformats a non-empty resolved value as `d/m/Y`. `type => 'paragraph'` renders the value inside a `<p>` with no label. `type => 'raw'` ignores `name`/`value`/`default` entirely and renders `html` (or the empty string if `html` isn't set).

---

## BaseTableComponent

Renders rows returned from `rows()` as an HTML table, with optional per-row and page-level action buttons.

{% highlight php %}
use Fabiom\UDDemo\Components\BaseTableComponent;

class ArticlesList extends BaseTableComponent {

    protected string $title = 'Articles';

    protected array $fields = [
        ['headline' => 'Title', 'value' => fn(array $row) => $row['art_title']],
        ['headline' => 'Date',  'value' => fn(array $row) => $row['art_created']],
    ];

    protected array $actions = [
        ['label' => 'Edit',   'icon' => 'bi bi-pencil', 'cssclass' => 'btn btn-sm btn-primary',
            'url' => fn(array $row) => 'article-edit.html?art_id=' . urlencode($row['art_id'])],
        ['label' => 'Delete', 'icon' => 'bi bi-trash',  'cssclass' => 'btn btn-sm btn-danger',
            'confirm' => 'Delete this article?',
            'url' => fn(array $row) => 'article-delete.html?art_id=' . urlencode($row['art_id'])],
    ];

    protected array $topActions = [
        ['label' => 'New article', 'cssclass' => 'btn btn-success', 'url' => fn() => 'article-new.html'],
    ];

    protected function rows(): array {
        return $this->executeSelectQuery(
            "SELECT art_id, art_title, art_created FROM articles ORDER BY art_created DESC",
            []
        );
    }

}
{% endhighlight %}

Override `rows()` (not `get_request()`) to supply the table's data; `get_request()` already calls it for you.

### Field reference

Each entry in `$fields` describes one column:

| Key | Type | Description |
|---|---|---|
| `headline` | string | Column header text. |
| `value` | callable | **Required.** `fn(array $row): string`. |
| `raw` | bool | Skip `htmlspecialchars()` escaping of the resolved value. |
| `style` | string | Inline `style` attribute on the `<th>`. |

`$actions` renders a per-row action cell; `$topActions`/`$bottomActions` render once, above/below the table. Each entry in any of the three is one of:

* An array: `['label' => '…', 'url' => string|fn(array $row): string, 'cssclass' => '…', 'icon' => '…', 'confirm' => '…']`. `confirm` wraps the link in a JS `confirm()` prompt before navigating.
* A `Closure(array $row): string` returning raw HTML.
* A raw HTML string.

`$topActions`/`$bottomActions` closures receive an empty array since they aren't bound to a row.

When `rows()` returns an empty array, the table shows a single "No data" row spanning all columns instead of an empty `<tbody>`.
