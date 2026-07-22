---
layout: page
name: Routing
---

# Routing

A <a href="{{site.baseurl}}/docs/controller">Controller</a> or standalone <a href="{{site.baseurl}}/docs/component">Component</a> becomes reachable over HTTP by carrying a `#[Route]` attribute. A generator command scans your codebase for these attributes and writes out a route table; nothing is hand-registered in a separate array.

---

## Declaring a route

{% highlight php %}
use Fabiom\UglyDuckling\Framework\Routing\Route;

#[Route(name: 'article_edit', slug: 'articles-edit')]
class ArticleEditController extends BaseController {
    // ...
}
{% endhighlight %}

| Argument | Description |
|---|---|
| `name` | Identifier used in code to build links — `url_for('article_edit', [...])`. Never appears in the URL. |
| `slug` | The URL segment the class answers to — `articles-edit.html`. |

The same attribute works on a standalone `BaseComponent` subclass:

{% highlight php %}
use Fabiom\UglyDuckling\Framework\Routing\Route;

#[Route(name: 'dashboard_summary', slug: 'dashboard-summary')]
class SummaryComponent extends BaseComponent {
    // ...
}
{% endhighlight %}

A component embedded inside a <a href="{{site.baseurl}}/docs/page-grid">Grid</a> or <a href="{{site.baseurl}}/docs/page-tabs">Tabs</a> page does **not** need a `#[Route]` — only components meant to be hit directly (typically over AJAX) need one.

---

## Generating the route table

{% highlight php %}
// routes-config.php, at your project root
return [
    'scan' => [
        ['namespace' => 'App\\Controllers\\', 'directory' => __DIR__ . '/app/Controllers'],
        ['namespace' => 'App\\Components\\',  'directory' => __DIR__ . '/app/Components'],
    ],
    'output' => __DIR__ . '/config/routes.php',
];
{% endhighlight %}

{% highlight bash %}
vendor/bin/ud-routes generate
{% endhighlight %}

This scans both directories recursively — nested namespaces (`App\Controllers\Admin\UsersController`) are found automatically, following the same convention Composer's own PSR-4 autoloading already uses. Every class carrying `#[Route]` is written into `config/routes.php` as a plain array:

{% highlight php %}
<?php

return [
    'article_edit' => ['slug' => 'articles-edit', 'controller' => 'App\\Controllers\\ArticleEditController'],
    'dashboard_summary' => ['slug' => 'dashboard-summary', 'component' => 'App\\Components\\SummaryComponent'],
];
{% endhighlight %}

Run `ud-routes generate` again whenever you add, rename, or remove a routed class — it's a build/deploy step, not something that runs on every request. Duplicate names or duplicate slugs across classes cause the command to fail loudly rather than silently overwrite one route with another.

---

## Wiring it into `index.php`

{% highlight php %}
use Fabiom\UglyDuckling\Framework\Routing\RouteTable;
use Fabiom\UglyDuckling\Framework\Routing\RouteDispatcher;

RouteTable::load(__DIR__ . '/config/routes.php');

(new RouteDispatcher(
    $dbconnection, $logger, $securityChecker, $mailer, $pageStatus, $groupsIndex,
    notFoundTemplateFile: 'application',
    notFoundViewFile: 'errors/notfound',
))->dispatch();
{% endhighlight %}

`RouteTable::load()` only reads a plain array — it does not touch any of your controller or component classes. `RouteDispatcher::dispatch()` resolves the current request's slug and instantiates exactly the one matching controller or component; every other class named in the table is left untouched, so registering hundreds of routes costs nothing per request beyond the array lookup.

An unmatched slug renders a shared 404 page (`StaticPageController`, `http_response_code(404)`) using the `notFoundTemplateFile`/`notFoundViewFile` you configure above.

For controllers, `RouteDispatcher` calls `setPageStatus()`, `setGroupsIndex()`, `makeAllPresets()`, and `showPage()` — the same lifecycle the controller already goes through, so `check_authorization_get_request()`/`check_authorization_post_request()` and every validation rule still apply exactly as documented in <a href="{{site.baseurl}}/docs/controller">Controller</a>. Failed authorization is handled by the controller itself (see `show_unauthorized_page()` there), not by the dispatcher.

For components, `RouteDispatcher` calls `renderAsPanel()` on GET and `handlePost()` on POST — the same methods used when a component is embedded in a page.

---

## Building links: `url_for()`

{% highlight php %}
<a href="<?= url_for('article_edit', ['id' => $article->id]) ?>">Edit</a>
{% endhighlight %}

`url_for()` looks the route up by name and builds `articles-edit.html?id=4`. For controller routes it also cross-checks the parameter names you pass against that controller's own `$get_validation_rules`/`$post_validation_rules` — so a typo'd or stale parameter name throws immediately, at the point the link is rendered, instead of producing a link that silently fails validation for a real user:

{% highlight php %}
url_for('article_edit', ['artcle_id' => 4]);
// InvalidArgumentException: url_for('article_edit'): unknown parameter(s) artcle_id
// — not in App\Controllers\ArticleEditController's validation rules
{% endhighlight %}

This check reflects on the target controller class once and caches the result, so linking to the same controller many times in a loop (e.g. one "Edit" link per table row) only pays that cost once.

Redirects and component success URLs should go through `url_for()` too, rather than a hardcoded slug string:

{% highlight php %}
$this->redirectToPage(url_for('articles'));
$this->postSuccessUrl = url_for('articles');
{% endhighlight %}
