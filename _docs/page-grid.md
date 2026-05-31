---
layout: page
name: Page (Grid layout)
---

# Description

A **Grid Page** is a controller that assembles one or more <a href="{{site.baseurl}}/docs/component">Components</a> into a Bootstrap grid layout. Each component occupies a position defined by a CSS class string (e.g. `col-md-6`). The page handles authorization, CSRF, GET/POST dispatch, and collects `<head>` / `<foot>` contributions from all its components.

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

Each entry in `$panels` is a **node**. Three node types are supported.

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

Nodes may be nested to any depth.

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

Override `check_authorization_get_request()` and `check_authorization_post_request()` to guard access to the whole page.

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
