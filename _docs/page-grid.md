---
layout: page
name: Page (Grid layout)
---

# Description

A **Grid Page** is a controller that assembles one or more <a href="{{site.baseurl}}/docs/component">Components</a> into a Bootstrap grid layout. Each component occupies a position defined by a CSS class string (e.g. `col-md-6`). The page handles authorization, CSRF, GET/POST dispatch, and collects `<head>` / `<foot>` contributions from all its components.

`BaseGridComponent` and <a href="{{site.baseurl}}/docs/page-tabs">`BaseTabsComponent`</a> are both thin subclasses of `BasePageComponent`, which actually implements all of this shared behaviour: the `showPage()` GET/POST lifecycle, CSRF token creation, `<head>`/`<foot>` collection, the `_component` POST dispatcher, and the embed node type. `BaseGridComponent` adds only the `$panels` array and a `renderPanels()` that walks it as a grid; `BaseTabsComponent` adds only the `$tabs` array and a `renderPanels()` that renders it as a Bootstrap tab widget instead. So everything documented below under Lifecycle hooks, POST dispatch, and Authorization is defined once on `BasePageComponent` and inherited by both page types unchanged.

For simple cases — a single component, or a fixed layout with no custom authorization or redirect logic — you do not need an explicit page class at all. Register the component or an inline panels array directly in `index_components.php` and the bootstrap creates the page for you. See <a href="{{site.baseurl}}/docs/component#registering-a-component-as-a-standalone-page">Registering a component as a standalone page</a>.

Create an explicit Grid Page by extending `BaseGridComponent` when you need custom authorization, a custom `onPostSuccess()`, or page-level POST handling.

---

## Minimal example

{% highlight php %}
use Fabiom\UDDemo\Components\BaseGridComponent;

class ArticlesDashboardPage extends BaseGridComponent {

    const CONTROLLER_NAME = 'articles-dashboard';

    protected array $panels = [
        ['cssclass' => 'col-12',   'component' => ArticlesList::class],
        ['cssclass' => 'col-md-6', 'component' => ArticleNew::class],
    ];

}
{% endhighlight %}

The `CONTROLLER_NAME` constant registers the page with the router. A request to `/articles-dashboard.html` will instantiate this class and call `showPage()`.

---

## The `$panels` array

Each entry in `$panels` is a **node**. Four node types are supported.

### 1. Component node

Renders a single component inside a `<div>` with the given CSS class.

{% highlight php %}
['cssclass' => 'col-md-8', 'component' => ArticlesList::class]
{% endhighlight %}

### 2. Row / container node

Groups child nodes inside a `<div>`. Use this to nest Bootstrap rows or to apply a wrapper class around several components.

{% highlight php %}
['cssclass' => 'row', 'panels' => [
    ['cssclass' => 'col-md-6', 'component' => ArticlesList::class],
    ['cssclass' => 'col-md-6', 'component' => ArticleNew::class],
]]
{% endhighlight %}

### 3. Tabs node

Renders a Bootstrap tabs widget inside a `<div>`. See <a href="{{site.baseurl}}/docs/page-tabs">Page (Tabs layout)</a> for full tab documentation.

{% highlight php %}
['cssclass' => 'col-12', 'tabs' => [
    ['id' => 'tab-list', 'label' => 'Articles', 'panels' => [
        ['cssclass' => 'col-12', 'component' => ArticlesList::class],
    ]],
    ['id' => 'tab-new', 'label' => 'New Article', 'panels' => [
        ['cssclass' => 'col-md-8 offset-md-2', 'component' => ArticleNew::class],
    ]],
]]
{% endhighlight %}

### 4. Embed node

Nests an entire other routable page — a `BaseGridComponent` or `BaseTabsComponent` subclass — inside a `<div>`, rather than a single component. This is how grid-in-grid or grid-in-tabs layouts are built without JSON.

{% highlight php %}
['cssclass' => 'col-md-6', 'embed' => ArticleStatsPage::class]
{% endhighlight %}

The embedded page's own `check_authorization_get_request()` is honored — if it returns `false`, the node renders nothing rather than raising an error. Embedding is type-agnostic and recursive: a Grid Page can embed a Tabs Page inside one of its panels, that Tabs Page can embed another Grid Page inside one of its tabs, and so on to any depth. Because embedding reaches into `renderPanels()`/`allPanels()` on the embedded page, head/foot asset collection and `_component` POST dispatch automatically flow through nested pages too — an embedded page does not get its own CSRF token or `showPage()` lifecycle; only the outermost routed page does.

Nodes may be nested to any depth.

---

## Lifecycle hooks

Override `onGetRequest()`/`onPostRequest()` to run page-level setup — building navigation, a menu, or other scaffolding — around the component dispatch. `onGetRequest()` runs after authorization but before validation on GET; `onPostRequest()` runs after POST dispatch completes.

{% highlight php %}
protected function onGetRequest(): void {
    $this->menubuilder = new SomeMenuBuilder($this->pageStatus);
}
{% endhighlight %}

---

## POST dispatch

When a form inside any component is submitted, the page inspects the hidden field `_component` to route the request to the correct component. Each component form must include:

{% highlight php %}
<input type="hidden" name="_component" value="<?= self::class ?>">
{% endhighlight %}

After a successful POST the page calls `onPostSuccess()`, which by default redirects back to the referring page. Override it to change this behaviour:

{% highlight php %}
protected function onPostSuccess(): void {
    $this->redirectToPage(url_for('articles-dashboard'));
}
{% endhighlight %}

---

## Authorization

For simple group restriction, set `$allowedGroups`; the page checks it automatically for both GET and POST, in place of overriding `check_authorization_get_request()`/`check_authorization_post_request()` yourself.

{% highlight php %}
protected array $allowedGroups = ['editor', 'admin'];
{% endhighlight %}

For anything more than a group check — combining it with a login check, for instance — override `check_authorization_get_request()` and `check_authorization_post_request()` directly.

{% highlight php %}
protected function check_authorization_get_request(): bool {
    return isset($_SESSION['group']) && $_SESSION['group'] === 'editor';
}

protected function check_authorization_post_request(): bool {
    return isset($_SESSION['group']) && $_SESSION['group'] === 'editor';
}
{% endhighlight %}

Individual components can also declare their own `check_authorization_resource_request()` to hide themselves independently of the page.

---

## Page-level POST handling

If none of the components match the `_component` field (or if no `_component` field is present), the page calls its own `post_request()` method. Override this for page-level form handling.

{% highlight php %}
protected array $post_validation_rules = [
    'search_term' => 'max_len,200',
];
protected array $post_filter_rules = [
    'search_term' => 'trim|sanitize_string',
];

protected function post_request(): void {
    $_SESSION['last_search'] = $this->postParameters['search_term'] ?? '';
}
{% endhighlight %}

---

## Complete example

{% highlight php %}
use Fabiom\UDDemo\Components\BaseGridComponent;
use Fabiom\UDDemo\Chapters\Articles\Components\ArticlesList;
use Fabiom\UDDemo\Chapters\Articles\Components\ArticleNew;
use Fabiom\UDDemo\Chapters\Articles\Components\ArticleSearch;

class ArticlesDashboardPage extends BaseGridComponent {

    const CONTROLLER_NAME = 'articles-dashboard';

    protected array $panels = [
        ['cssclass' => 'col-12 mb-4', 'component' => ArticleSearch::class],
        ['cssclass' => 'row', 'panels' => [
            ['cssclass' => 'col-md-8', 'component' => ArticlesList::class],
            ['cssclass' => 'col-md-4', 'component' => ArticleNew::class],
        ]],
    ];

    protected function check_authorization_get_request(): bool {
        return isset($_SESSION['group']);
    }

    protected function check_authorization_post_request(): bool {
        return isset($_SESSION['group']);
    }

}
{% endhighlight %}

This page renders a search bar at the top, then a two-column layout with the articles list on the left and a new-article form on the right. POST submissions from either component are routed automatically to the correct component via the `_component` field.
