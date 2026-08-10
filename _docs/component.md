---
layout: page
name: Component
---

# Description

A **Component** is a PHP class that encapsulates a single, self-contained unit of a page: it fetches data, validates input, handles POST submissions, and renders its own HTML.

Every component extends `BaseComponent` and must implement the abstract method `render(array $data)`.

For the common cases of a form, a read-only info panel, or a table, you don't need to hand-write `render()` at all — see <a href="{{site.baseurl}}/docs/declarative-components">Declarative leaf components</a>.

A component can be used in two ways:

* **Embedded in a page** — a <a href="{{site.baseurl}}/docs/page-grid">Grid Page</a> or <a href="{{site.baseurl}}/docs/page-tabs">Tabs Page</a> declares it in its `$panels` or `$tabs` array alongside other components.
* **Standalone** — registered by slug in `index_components.php`, or wired up directly by calling `renderAsPanel()`/`handlePost()` from your own entry-point file. See <a href="{{site.baseurl}}/docs/routing">Linking &amp; Visibility</a> for both approaches, and <a href="#registering-a-component-as-a-standalone-page">Registering a component as a standalone page</a> below for the `index_components.php` shortcut.

For a step-by-step walkthrough of all four CRUD operations using components, see the <a href="{{site.baseurl}}/tutorials/crud-components">CRUD with Components</a> tutorial.

When the logic inside `get_request()`/`post_request()` grows complex enough to need isolated testing, extract it into a <a href="{{site.baseurl}}/docs/service">Service</a> and keep the component itself limited to validation, delegating to the Service, and rendering the outcome.

---

## Minimal example

{% highlight php %}
use Fabiom\UDDemo\Components\BaseComponent;

class ArticlesList extends BaseComponent {

    protected function get_request(): array {
        return [
            'articles' => $this->executeSelectQuery(
                "SELECT art_id, art_title, art_created FROM articles ORDER BY art_created DESC",
                []
            ),
        ];
    }

    public function render(array $data): void { ?>
        <div class="card">
            <div class="card-header"><h5 class="mb-0">Articles</h5></div>
            <div class="card-body p-0">
                <table class="table table-sm mb-0">
                    <thead>
                        <tr><th>Title</th><th>Date</th></tr>
                    </thead>
                    <tbody>
                        <?php foreach ($data['articles'] as $row): ?>
                            <tr>
                                <td><?= htmlspecialchars($row['art_title']) ?></td>
                                <td><?= htmlspecialchars($row['art_created']) ?></td>
                            </tr>
                        <?php endforeach; ?>
                    </tbody>
                </table>
            </div>
        </div>
    <?php }

}
{% endhighlight %}

---

## GET: fetching data

Override `get_request()` to load data from the database. The returned array is passed as `$data` to `render()`.

{% highlight php %}
protected function get_request(): array {
    return [
        'articles' => $this->executeSelectQuery(
            "SELECT art_id, art_title FROM articles WHERE art_authorid = :authorid",
            ['authorid' => $_SESSION['user_id']]
        ),
    ];
}
{% endhighlight %}

### Validating GET parameters

Declare `$get_validation_rules` and `$get_filter_rules` to validate URL parameters before using them. Validated values land in `$this->getParameters`.

{% highlight php %}
protected array $get_validation_rules = [
    'art_id' => 'required|max_len,38',
];
protected array $get_filter_rules = [
    'art_id' => 'trim',
];

protected function get_request(): array {
    if (empty($this->getParameters)) {
        return [];
    }
    $rows = $this->executeSelectQuery(
        "SELECT art_id, art_title, art_body FROM articles WHERE art_id = :art_id",
        $this->getParameters
    );
    return $rows[0] ?? [];
}
{% endhighlight %}

Validation rules follow the [GUMP library](https://github.com/Wixel/GUMP) syntax. If validation fails, `$this->getParameters` is empty and `render()` receives an empty array.

---

## POST: handling form submissions

Override `post_request()` to write data to the database. Declare `$post_validation_rules` and `$post_filter_rules` to validate POST input. Validated values land in `$this->postParameters`.

{% highlight php %}
protected array $post_validation_rules = [
    'art_id'    => 'required|max_len,38',
    'art_title' => 'required|max_len,200',
];
protected array $post_filter_rules = [
    'art_title' => 'trim|sanitize_string',
];

protected function post_request(): void {
    $this->executeWriteQuery(
        "UPDATE articles SET art_title = :art_title WHERE art_id = :art_id",
        $this->postParameters
    );
}
{% endhighlight %}

### Routing POST submissions to the right component

A page can host several components. When a form is submitted, the framework needs to know which component should handle it. Include a hidden field named `_component` with the fully-qualified class name:

{% highlight php %}
<form method="post">
    <input type="hidden" name="_component" value="<?= self::class ?>">
    <!-- form fields -->
    <button type="submit" class="btn btn-primary">Save</button>
</form>
{% endhighlight %}

The page's POST dispatcher reads `$_POST['_component']` and delegates to the matching component.

### Redirecting after a successful POST

Set `$postSuccessUrl` to redirect after a successful submission. Set `$postSuccessMessage` to store a flash message in the session.

{% highlight php %}
public string $postSuccessMessage = 'Article saved successfully';

public function __construct() {
    $this->postSuccessUrl = 'articles-list.html';
}
{% endhighlight %}

---

## Database helpers

`BaseComponent` provides two protected query helpers that handle PDO preparation and parameter binding automatically.

### executeSelectQuery

Returns an array of associative arrays (one per row).

{% highlight php %}
$rows = $this->executeSelectQuery(
    "SELECT art_id, art_title FROM articles WHERE art_authorid = :authorid",
    ['authorid' => $_SESSION['user_id']]
);
{% endhighlight %}

### executeWriteQuery

Executes an INSERT, UPDATE, or DELETE. Returns void; throws on failure.

{% highlight php %}
$this->executeWriteQuery(
    "UPDATE articles SET art_title = :title WHERE art_id = :id",
    ['title' => $this->postParameters['art_title'], 'id' => $this->postParameters['art_id']]
);
{% endhighlight %}

---

## Registering a component as a standalone page

When a component is the only thing on a screen, there is no need to create an explicit page class. Register the component class directly in `index_components.php` and the bootstrap wraps it automatically in a full-width (`col-md-12`) grid:

{% highlight php %}
// index_components.php
$index_components = [
    'article-edit'   => ArticleEdit::class,
    'article-delete' => ArticleDelete::class,
];
{% endhighlight %}

The bootstrap also accepts an inline panels array or a tabs array as the value, so a layout can be described without writing any class at all:

{% highlight php %}
$index_components = [
    // inline grid — no class needed
    'articles-page' => [
        ['cssclass' => 'col-12 mb-4', 'component' => ArticlesList::class],
        ['cssclass' => 'col-md-8 offset-md-2', 'component' => ArticleNew::class],
    ],
    // inline tabs — no class needed
    'articles-tabbed' => ['tabs' => [
        ['id' => 'tab-list', 'label' => 'Articles', 'panels' => [
            ['cssclass' => 'col-12', 'component' => ArticlesList::class],
        ]],
        ['id' => 'tab-new', 'label' => 'New Article', 'panels' => [
            ['cssclass' => 'col-md-8 offset-md-2', 'component' => ArticleNew::class],
        ]],
    ]],
];
{% endhighlight %}

An explicit page class (extending `BaseGridComponent` or `BaseTabsComponent`) is only necessary when the page needs custom authorization logic, a custom `onPostSuccess()` redirect, or page-level POST handling beyond what the components themselves provide.

---

## Authorization

For simple group restriction, set `$allowedGroups`. `BaseComponent` checks it automatically before both rendering and handling a POST; leaving it empty (the default) means the component is visible to any logged-in group.

{% highlight php %}
protected array $allowedGroups = ['admin', 'editor'];
{% endhighlight %}

For anything more than a group check, override `check_authorization_resource_request()` directly. Return `false` to silently skip rendering.

{% highlight php %}
protected function check_authorization_resource_request(): bool {
    return isset($_SESSION['group']) && $_SESSION['group'] === 'admin';
}
{% endhighlight %}

A component embedded in a <a href="{{site.baseurl}}/docs/page-grid">Grid Page</a> or <a href="{{site.baseurl}}/docs/page-tabs">Tabs Page</a> is authorized independently of the page — it can hide itself even on a page that is otherwise visible to the current user.

Code outside the component's own hierarchy — a page deciding whether to render a panel's wrapper `<div>` at all, for instance — can check this via the public `isAuthorized()` accessor, without needing access to the protected `check_authorization_resource_request()` it wraps:

{% highlight php %}
if ($component->isAuthorized()) {
    // safe to render this component, link to it, load its head/foot assets, etc.
}
{% endhighlight %}

This is exactly how <a href="{{site.baseurl}}/docs/page-grid#authorization">Grid Page</a> and <a href="{{site.baseurl}}/docs/page-tabs">Tabs Page</a> avoid leaving an empty wrapper behind for a component a user can't see.

---

## CSS and JavaScript

Override `addToHead()` to inject into the `<head>` section, and `addToFoot()` to inject just before `</body>`. These are called by the parent page, which deduplicates calls automatically.

{% highlight php %}
public function addToHead(): string {
    return '<link rel="stylesheet" href="vendor/datatables/dataTables.bootstrap5.min.css">';
}

public function addToFoot(): string {
    return '<script src="vendor/datatables/dataTables.min.js"></script>';
}
{% endhighlight %}

---

## Complete example: an edit form component

{% highlight php %}
use Fabiom\UDDemo\Components\BaseComponent;

class ArticleEdit extends BaseComponent {

    public string $postSuccessMessage = 'Article updated';

    public function __construct() {
        $this->postSuccessUrl = 'articles-list.html';
    }

    protected array $get_validation_rules = ['art_id' => 'required|max_len,38'];
    protected array $get_filter_rules     = ['art_id' => 'trim'];

    protected array $post_validation_rules = [
        'art_id'    => 'required|max_len,38',
        'art_title' => 'required|max_len,200',
        'art_body'  => 'required',
    ];
    protected array $post_filter_rules = [
        'art_title' => 'trim|sanitize_string',
        'art_body'  => 'trim',
    ];

    protected function get_request(): array {
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

    public function render(array $data): void { ?>
        <div class="card">
            <div class="card-header"><h5 class="mb-0">Edit Article</h5></div>
            <div class="card-body">
                <form method="post">
                    <input type="hidden" name="_component" value="<?= self::class ?>">
                    <input type="hidden" name="art_id"
                           value="<?= htmlspecialchars($data['art_id'] ?? '') ?>">
                    <div class="mb-3">
                        <label class="form-label">Title</label>
                        <input type="text" class="form-control" name="art_title"
                               value="<?= htmlspecialchars($data['art_title'] ?? '') ?>">
                    </div>
                    <div class="mb-3">
                        <label class="form-label">Body</label>
                        <textarea class="form-control" name="art_body" rows="6"><?=
                            htmlspecialchars($data['art_body'] ?? '')
                        ?></textarea>
                    </div>
                    <button type="submit" class="btn btn-primary">Save</button>
                </form>
            </div>
        </div>
    <?php }

}
{% endhighlight %}
