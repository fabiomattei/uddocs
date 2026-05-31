---
layout: page
title: CRUD with Components
orderfield: 3
---

This tutorial builds a complete Create / Read / Update / Delete feature using the component system.
Each operation is a self-contained <a href="{{site.baseurl}}/docs/component">Component</a> class.

A component can be registered directly as a route, or embedded inside a page built with <a href="{{site.baseurl}}/docs/page-grid">BaseGridComponent</a> or <a href="{{site.baseurl}}/docs/page-tabs">BaseTabsComponent</a>.
For single-component screens the direct registration is simpler — no page class is needed.

The tutorial uses a single `articles` table throughout:

{% highlight sql %}
CREATE TABLE articles (
    art_id      VARCHAR(36)  NOT NULL PRIMARY KEY,
    art_title   VARCHAR(200) NOT NULL,
    art_body    TEXT         NOT NULL,
    art_created DATETIME     NOT NULL
);
{% endhighlight %}

---

## READ — list all records

The list component runs a SELECT query and renders the results as a table.
Each row carries links to the edit and delete pages.

{% highlight php %}
use Fabiom\UDDemo\Components\BaseComponent;

class ArticlesList extends BaseComponent {

    protected function get_request(): array {
        return [
            'articles' => $this->executeSelectQuery(
                "SELECT art_id, art_title, art_created
                 FROM articles
                 ORDER BY art_created DESC",
                []
            ),
        ];
    }

    public function render(array $data): void { ?>
        <div class="card">
            <div class="card-header d-flex justify-content-between align-items-center">
                <h5 class="mb-0">Articles</h5>
            </div>
            <div class="card-body p-0">
                <table class="table table-striped table-sm mb-0">
                    <thead>
                        <tr>
                            <th>Title</th>
                            <th>Date</th>
                            <th></th>
                        </tr>
                    </thead>
                    <tbody>
                        <?php foreach ($data['articles'] as $row): ?>
                            <tr>
                                <td><?= htmlspecialchars($row['art_title']) ?></td>
                                <td><?= htmlspecialchars($row['art_created']) ?></td>
                                <td class="text-end">
                                    <a href="<?= url_for('article-edit', ['art_id' => $row['art_id']]) ?>"
                                       class="btn btn-sm btn-primary me-1">Edit</a>
                                    <a href="<?= url_for('article-delete', ['art_id' => $row['art_id']]) ?>"
                                       class="btn btn-sm btn-danger">Delete</a>
                                </td>
                            </tr>
                        <?php endforeach; ?>
                    </tbody>
                </table>
            </div>
        </div>
    <?php }

}
{% endhighlight %}

`url_for('article-edit', ...)` and `url_for('article-delete', ...)` refer to the route keys used when registering the edit and delete components in `index_components.php`, as shown in the registration section below.

---

## CREATE — insert a new record

The create component has no GET data to load. It only needs the POST logic.

{% highlight php %}
use Fabiom\UDDemo\Components\BaseComponent;

class ArticleNew extends BaseComponent {

    public string $postSuccessMessage = 'Article created';

    public function __construct() {
        $this->postSuccessUrl = url_for('articles-page');
    }

    protected array $post_validation_rules = [
        'art_title' => 'required|max_len,200',
        'art_body'  => 'required',
    ];
    protected array $post_filter_rules = [
        'art_title' => 'trim|sanitize_string',
        'art_body'  => 'trim',
    ];

    protected function post_request(): void {
        $this->executeWriteQuery(
            "INSERT INTO articles (art_id, art_title, art_body, art_created)
             VALUES (:art_id, :art_title, :art_body, NOW())",
            array_merge($this->postParameters, ['art_id' => uniqid('', true)])
        );
    }

    public function render(array $data): void { ?>
        <div class="card">
            <div class="card-header"><h5 class="mb-0">New Article</h5></div>
            <div class="card-body">
                <form method="post">
                    <input type="hidden" name="_component" value="<?= self::class ?>">
                    <div class="mb-3">
                        <label class="form-label">Title</label>
                        <input type="text" class="form-control" name="art_title">
                    </div>
                    <div class="mb-3">
                        <label class="form-label">Body</label>
                        <textarea class="form-control" name="art_body" rows="5"></textarea>
                    </div>
                    <button type="submit" class="btn btn-success">Save</button>
                </form>
            </div>
        </div>
    <?php }

}
{% endhighlight %}

The `_component` hidden field is required so the page's POST dispatcher knows which component owns this form. See <a href="{{site.baseurl}}/docs/component#routing-post-submissions-to-the-right-component">POST routing</a>.

---

## UPDATE — edit an existing record

The edit component validates the `art_id` URL parameter, loads the current values in `get_request()` to pre-fill the form, then updates the row on POST.

{% highlight php %}
use Fabiom\UDDemo\Components\BaseComponent;

class ArticleEdit extends BaseComponent {

    public string $postSuccessMessage = 'Article updated';

    public function __construct() {
        $this->postSuccessUrl = url_for('articles-page');
    }

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
            "UPDATE articles
             SET art_title = :art_title, art_body = :art_body
             WHERE art_id = :art_id",
            $this->postParameters
        );
    }

    public function render(array $data): void {
        if (empty($data)) { ?>
            <div class="alert alert-warning">Article not found.</div>
        <?php return; } ?>
        <div class="card">
            <div class="card-header"><h5 class="mb-0">Edit Article</h5></div>
            <div class="card-body">
                <form method="post">
                    <input type="hidden" name="_component" value="<?= self::class ?>">
                    <input type="hidden" name="art_id"
                           value="<?= htmlspecialchars($data['art_id']) ?>">
                    <div class="mb-3">
                        <label class="form-label">Title</label>
                        <input type="text" class="form-control" name="art_title"
                               value="<?= htmlspecialchars($data['art_title']) ?>">
                    </div>
                    <div class="mb-3">
                        <label class="form-label">Body</label>
                        <textarea class="form-control" name="art_body" rows="6"><?=
                            htmlspecialchars($data['art_body'])
                        ?></textarea>
                    </div>
                    <a href="<?= url_for('articles-page') ?>" class="btn btn-secondary me-2">Cancel</a>
                    <button type="submit" class="btn btn-primary">Save</button>
                </form>
            </div>
        </div>
    <?php }

}
{% endhighlight %}

---

## DELETE — confirm and remove a record

The delete component shows a confirmation screen on GET and performs the deletion on POST.
Showing the record details before deletion avoids accidental deletes.

{% highlight php %}
use Fabiom\UDDemo\Components\BaseComponent;

class ArticleDelete extends BaseComponent {

    public string $postSuccessMessage = 'Article deleted';

    public function __construct() {
        $this->postSuccessUrl = url_for('articles-page');
    }

    protected array $get_validation_rules = ['art_id' => 'required|max_len,36'];
    protected array $get_filter_rules     = ['art_id' => 'trim'];

    protected array $post_validation_rules = ['art_id' => 'required|max_len,36'];
    protected array $post_filter_rules     = ['art_id' => 'trim'];

    protected function get_request(): array {
        if (empty($this->getParameters)) {
            return [];
        }
        $rows = $this->executeSelectQuery(
            "SELECT art_id, art_title FROM articles WHERE art_id = :art_id",
            $this->getParameters
        );
        return $rows[0] ?? [];
    }

    protected function post_request(): void {
        $this->executeWriteQuery(
            "DELETE FROM articles WHERE art_id = :art_id",
            $this->postParameters
        );
    }

    public function render(array $data): void {
        if (empty($data)) { ?>
            <div class="alert alert-warning">Article not found.</div>
        <?php return; } ?>
        <div class="card border-danger">
            <div class="card-header text-bg-danger">
                <h5 class="mb-0">Delete Article</h5>
            </div>
            <div class="card-body">
                <p>Are you sure you want to delete
                   <strong><?= htmlspecialchars($data['art_title']) ?></strong>?
                   This action cannot be undone.
                </p>
                <form method="post">
                    <input type="hidden" name="_component" value="<?= self::class ?>">
                    <input type="hidden" name="art_id"
                           value="<?= htmlspecialchars($data['art_id']) ?>">
                    <a href="<?= url_for('articles-page') ?>" class="btn btn-secondary me-2">Cancel</a>
                    <button type="submit" class="btn btn-danger">Delete</button>
                </form>
            </div>
        </div>
    <?php }

}
{% endhighlight %}

---

## Assembling the pages

The list and create components share one page because they appear together on the same screen. Edit and delete each own a dedicated screen because they receive a row identifier via the URL.

### Main page: list + new form

The main page combines two components so it needs an explicit layout. Register an inline panels array directly in `index_components.php` — no page class required:

{% highlight php %}
// index_components.php
'articles-page' => [
    ['cssclass' => 'col-12 mb-4', 'component' => ArticlesList::class],
    ['cssclass' => 'row', 'panels' => [
        ['cssclass' => 'col-md-8 offset-md-2', 'component' => ArticleNew::class],
    ]],
],
{% endhighlight %}

### Edit and delete pages

`ArticleEdit` and `ArticleDelete` each fill the whole screen on their own. Register them directly — the bootstrap auto-wraps each one in a full-width grid:

{% highlight php %}
// index_components.php
'article-edit'   => ArticleEdit::class,
'article-delete' => ArticleDelete::class,
{% endhighlight %}

No wrapper class is needed. The route keys `'articles-page'`, `'article-edit'`, and `'article-delete'` are exactly what `url_for()` resolves inside the components.

---

## Tabs variant for the main page

If you prefer a tabbed layout instead of a stacked one, swap `BaseGridComponent` for `BaseTabsComponent`:

{% highlight php %}
use Fabiom\UDDemo\Components\BaseTabsComponent;

class ArticlesPage extends BaseTabsComponent {

    const CONTROLLER_NAME = 'articles-page';

    protected array $tabs = [
        [
            'id'     => 'tab-list',
            'label'  => 'All Articles',
            'panels' => [
                ['cssclass' => 'col-12', 'component' => ArticlesList::class],
            ],
        ],
        [
            'id'     => 'tab-new',
            'label'  => 'New Article',
            'panels' => [
                ['cssclass' => 'row', 'panels' => [
                    ['cssclass' => 'col-md-8 offset-md-2', 'component' => ArticleNew::class],
                ]],
            ],
        ],
    ];

}
{% endhighlight %}

The edit and delete registrations remain unchanged — `ArticleEdit::class` and `ArticleDelete::class` are still registered directly, regardless of how the main list page is laid out.

---

## Navigation flow

```
articles-page (GET)
  └── ArticlesList renders table
        ├── "Edit" link → article-edit?art_id=X (GET)
        │     └── ArticleEdit renders pre-filled form
        │           └── POST → ArticleEdit updates row → redirect articles-page
        └── "Delete" link → article-delete?art_id=X (GET)
              └── ArticleDelete renders confirmation
                    └── POST → ArticleDelete deletes row → redirect articles-page

articles-page (POST, _component=ArticleNew)
  └── ArticleNew inserts row → redirect articles-page
```

---

## Registering in index_components.php

All three routes go into `index_components.php`. The complete registration for this tutorial looks like this:

{% highlight php %}
// index_components.php
$index_components = [

    'articles-page' => [
        ['cssclass' => 'col-12 mb-4', 'component' => ArticlesList::class],
        ['cssclass' => 'row', 'panels' => [
            ['cssclass' => 'col-md-8 offset-md-2', 'component' => ArticleNew::class],
        ]],
    ],

    'article-edit'   => ArticleEdit::class,
    'article-delete' => ArticleDelete::class,

];
{% endhighlight %}

The array key is the route name. `url_for()` inside the components resolves these keys — but only after a matching entry exists in `index_links.php`.

---

## Registering in index_links.php

`$index_links` maps every semantic link name to the route it targets and declares the URL parameters it accepts. `url_for()` reads this map to build HTML-safe URLs.

Add one entry per route used in the tutorial:

{% highlight php %}
// index_links.php
$index_links = [

    'articles-page'  => ['page' => 'articles-page'],
    'article-edit'   => ['page' => 'article-edit',   'params' => ['art_id']],
    'article-delete' => ['page' => 'article-delete', 'params' => ['art_id']],

];
{% endhighlight %}

Each entry has:

* **`page`** — the route key from `$index_components`. This becomes the base of the URL (`article-edit.html`).
* **`params`** — the query-string parameters this link accepts. Listing them documents the contract; `url_for()` appends whatever is passed as its second argument.

With these entries in place, the calls inside the components resolve correctly:

{% highlight php %}
url_for('articles-page')                          // → articles-page.html
url_for('article-edit',   ['art_id' => $id])      // → article-edit.html?art_id=…
url_for('article-delete', ['art_id' => $id])      // → article-delete.html?art_id=…
{% endhighlight %}
