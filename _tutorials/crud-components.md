---
layout: page
title: CRUD with Components
orderfield: 3
---

This tutorial builds a complete Create / Read / Update / Delete feature using the component system.
Each operation is a self-contained <a href="{{site.baseurl}}/docs/component">Component</a> class.
Components are assembled into pages using <a href="{{site.baseurl}}/docs/page-grid">BaseGridComponent</a> or <a href="{{site.baseurl}}/docs/page-tabs">BaseTabsComponent</a>.

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

`url_for('article-edit', ...)` and `url_for('article-delete', ...)` refer to the `CONTROLLER_NAME` of the edit and delete pages defined further below.

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

Each component is placed on a page. The list and create components share one page. Edit and delete each have their own dedicated page because they receive a row identifier via the URL.

### Main page: list + new form

{% highlight php %}
use Fabiom\UDDemo\Components\BaseGridComponent;

class ArticlesPage extends BaseGridComponent {

    const CONTROLLER_NAME = 'articles-page';

    protected array $panels = [
        ['cssclass' => 'col-12 mb-4', 'component' => ArticlesList::class],
        ['cssclass' => 'row', 'panels' => [
            ['cssclass' => 'col-md-8 offset-md-2', 'component' => ArticleNew::class],
        ]],
    ];

}
{% endhighlight %}

### Edit page

{% highlight php %}
use Fabiom\UDDemo\Components\BaseGridComponent;

class ArticleEditPage extends BaseGridComponent {

    const CONTROLLER_NAME = 'article-edit';

    protected array $panels = [
        ['cssclass' => 'col-md-8 offset-md-2', 'component' => ArticleEdit::class],
    ];

}
{% endhighlight %}

### Delete page

{% highlight php %}
use Fabiom\UDDemo\Components\BaseGridComponent;

class ArticleDeletePage extends BaseGridComponent {

    const CONTROLLER_NAME = 'article-delete';

    protected array $panels = [
        ['cssclass' => 'col-md-6 offset-md-3', 'component' => ArticleDelete::class],
    ];

}
{% endhighlight %}

The `CONTROLLER_NAME` on each page registers it with the router. `url_for('articles-page')`, `url_for('article-edit', ...)`, and `url_for('article-delete', ...)` used inside the components resolve to those names.

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

The edit and delete pages remain unchanged — they are single-component grid pages regardless of how the main list page is organised.

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

## Adding to the router

Register each page in `src/Controllers/CustomRouter.php`:

{% highlight php %}
'articles-page'  => ArticlesPage::class,
'article-edit'   => ArticleEditPage::class,
'article-delete' => ArticleDeletePage::class,
{% endhighlight %}

The key is the `CONTROLLER_NAME` value; the value is the fully-qualified class name.
