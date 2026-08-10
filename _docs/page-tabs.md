---
layout: page
name: Page (Tabs layout)
---

# Description

A **Tabs Page** is a controller that assembles <a href="{{site.baseurl}}/docs/component">Components</a> into a Bootstrap tab interface. Each tab has an id, a visible label, and its own grid of components. Everything else — authorization, CSRF, GET/POST dispatch, `<head>` / `<foot>` collection — works identically to a <a href="{{site.baseurl}}/docs/page-grid">Grid Page</a>, because both extend the same `BasePageComponent`: it owns the `showPage()` lifecycle, CSRF, dispatch, and head/foot collection, while `BaseTabsComponent` contributes only the `$tabs` array and a tab-rendering `renderPanels()`. See the class hierarchy note in the <a href="{{site.baseurl}}/docs/page-grid">Grid Page docs</a> for the full picture.

Create a Tabs Page by extending `BaseTabsComponent` and declaring a `$tabs` array.

---

## Minimal example

{% highlight php %}
use Fabiom\UDDemo\Components\BaseTabsComponent;

class ArticlesPage extends BaseTabsComponent {

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
                ['cssclass' => 'col-md-8 offset-md-2', 'component' => ArticleNew::class],
            ],
        ],
    ];

}
{% endhighlight %}

There's no built-in router — wire the page to a URL yourself, e.g. from `articles-page.php`:

{% highlight php %}
$page = new ArticlesPage();
$page->setPageStatus($pageStatus);
$page->setGroupsIndex($groupsIndex);
$page->makeAllPresets($dbconnection, $logger, $securityChecker, $mailer);
$page->showPage();
{% endhighlight %}

The tab interface renders with the first tab active by default. See <a href="{{site.baseurl}}/docs/routing">Linking &amp; Visibility</a> for the full picture.

---

## The `$tabs` array

Each entry in `$tabs` is a tab descriptor with three required keys.

| Key | Type | Description |
|---|---|---|
| `id` | string | Unique HTML id for the tab panel. Must be valid as an HTML id attribute. |
| `label` | string | Text shown on the tab button. |
| `panels` | array | A list of layout nodes, identical to the `$panels` array of a <a href="{{site.baseurl}}/docs/page-grid">Grid Page</a>. |

If every panel inside a tab turns out to be unauthorized for the current session, the whole tab is dropped — button and pane both — rather than showing an empty tab with nothing in it. This is resolved recursively, so a tab whose only content is a nested tabs node or an embed is also hidden if nothing inside *that* would render either.

---

## Panel nodes inside a tab

The `panels` array inside each tab supports the same four node types as a Grid Page.

### Component node

Same rule as a Grid Page: an unauthorized component's `<div>` wrapper is skipped, not left empty.

{% highlight php %}
['cssclass' => 'col-12', 'component' => ArticlesList::class]
{% endhighlight %}

### Container node

{% highlight php %}
['cssclass' => 'row', 'panels' => [
    ['cssclass' => 'col-md-6', 'component' => ArticlesList::class],
    ['cssclass' => 'col-md-6', 'component' => ArticleSummary::class],
]]
{% endhighlight %}

### Nested tabs node

Tabs can be nested — a tab panel can itself contain a tabs node.

{% highlight php %}
['cssclass' => 'col-12', 'tabs' => [
    ['id' => 'tab-published', 'label' => 'Published', 'panels' => [
        ['cssclass' => 'col-12', 'component' => PublishedArticlesList::class],
    ]],
    ['id' => 'tab-drafts', 'label' => 'Drafts', 'panels' => [
        ['cssclass' => 'col-12', 'component' => DraftArticlesList::class],
    ]],
]]
{% endhighlight %}

### Embed node

Nests an entire other routable page — a `BaseGridComponent` or `BaseTabsComponent` subclass — inside a tab, rather than a single component. See the "Embed node" section of the <a href="{{site.baseurl}}/docs/page-grid">Grid Page docs</a> for the full explanation of embedding, which applies identically here.

{% highlight php %}
['cssclass' => 'col-12', 'embed' => ArticleStatsPage::class]
{% endhighlight %}

---

## Lifecycle hooks

Override `onGetRequest()`/`onPostRequest()` for page-level setup around the component dispatch, exactly as on a <a href="{{site.baseurl}}/docs/page-grid#lifecycle-hooks">Grid Page</a>.

---

## POST dispatch

POST handling works the same as in a Grid Page. Each component form must include a hidden `_component` field so the page can route the submission to the correct component:

{% highlight php %}
<form method="post">
    <input type="hidden" name="_component" value="<?= self::class ?>">
    <!-- form fields -->
    <button type="submit" class="btn btn-primary">Save</button>
</form>
{% endhighlight %}

After a successful POST the page redirects back to the referring page by default. Override `onPostSuccess()` to change this:

{% highlight php %}
protected function onPostSuccess(): void {
    $this->redirectToPage('articles-page.html');
}
{% endhighlight %}

---

## Authorization

For simple group restriction, set `$allowedGroups`; the page checks it automatically for both GET and POST.

{% highlight php %}
protected array $allowedGroups = ['editor', 'admin'];
{% endhighlight %}

For anything more than a group check, override `check_authorization_get_request()` and `check_authorization_post_request()` directly.

{% highlight php %}
protected function check_authorization_get_request(): bool {
    return isset($_SESSION['group']) && in_array($_SESSION['group'], ['editor', 'admin']);
}

protected function check_authorization_post_request(): bool {
    return isset($_SESSION['group']) && in_array($_SESSION['group'], ['editor', 'admin']);
}
{% endhighlight %}

Individual components inside any tab can still declare their own `check_authorization_resource_request()` to hide themselves for certain users. As described above, a tab left with nothing authorized to show is itself omitted — see <a href="{{site.baseurl}}/docs/page-grid#authorization">Grid Page authorization</a> for how this check (`isAuthorized()`) works.

---

## Complete example

{% highlight php %}
use Fabiom\UDDemo\Components\BaseTabsComponent;
use Fabiom\UDDemo\Chapters\Articles\Components\ArticlesList;
use Fabiom\UDDemo\Chapters\Articles\Components\ArticleNew;
use Fabiom\UDDemo\Chapters\Articles\Components\ArticleSearch;
use Fabiom\UDDemo\Chapters\Articles\Components\ArticleStats;

class ArticlesPage extends BaseTabsComponent {

    protected array $tabs = [
        [
            'id'     => 'tab-list',
            'label'  => 'Articles',
            'panels' => [
                ['cssclass' => 'col-12 mb-3', 'component' => ArticleSearch::class],
                ['cssclass' => 'col-12',       'component' => ArticlesList::class],
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
        [
            'id'     => 'tab-stats',
            'label'  => 'Statistics',
            'panels' => [
                ['cssclass' => 'col-12', 'component' => ArticleStats::class],
            ],
        ],
    ];

    protected function check_authorization_get_request(): bool {
        return isset($_SESSION['group']);
    }

    protected function check_authorization_post_request(): bool {
        return isset($_SESSION['group']);
    }

}
{% endhighlight %}

This page renders three tabs. The first tab shows a search bar above the articles list. The second tab shows a centred new-article form. The third tab shows statistics. POST submissions from `ArticleNew` are routed automatically via the `_component` field.
