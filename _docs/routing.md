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

Both kinds can also declare `$allowedGroups` — see <a href="{{site.baseurl}}/docs/controller">Controller</a> and <a href="{{site.baseurl}}/docs/component">Component</a> for how. The generator reads that property straight off the class (it never runs your code to do so) and carries it into the route table as `allowedgroups`, which is what powers the group-visibility filtering covered later on this page.

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

## Merging in JSON resources

A JSON resource (see <a href="{{site.baseurl}}/docs/json-template">JSON Template</a>) isn't a class and can't carry a `#[Route]` attribute, but the generator can still merge one in — so the same table covers everything a link might point at, not just attributed classes:

{% highlight php %}
// routes-config.php
return [
    'scan' => [
        ['namespace' => 'App\\Controllers\\', 'directory' => __DIR__ . '/app/Controllers'],
    ],
    'jsonResources' => [
        'registry' => __DIR__ . '/index_json_resources.php',   // slug => JSON file path
        'variable' => 'index_resources',                        // the array variable that file defines
    ],
    'output' => __DIR__ . '/config/routes.php',
];
{% endhighlight %}

Each entry is keyed by the resource's own slug — a JSON resource has no separate `name`, unlike a `#[Route]` class — and its `allowedgroups` is read straight from the resource's own JSON:

{% highlight json %}
{
  "allowedgroups": ["readergroup", "writergroup"]
}
{% endhighlight %}

If a slug is claimed by both a scanned class and a JSON resource — a full-page controller that loads its own JSON resource internally over AJAX under the same slug is a common case — the class wins and the JSON resource entry is skipped, rather than the generator failing on a "duplicate" it didn't cause.

A resource with no `"allowedgroups"` key at all is recorded as `null`, not `[]` — deliberately different from "empty means every group", so the group-visibility check described below can tell "explicitly open to everyone" apart from "no policy declared, deny by default."

---

## Wiring it into `index.php`

{% highlight php %}
use Fabiom\UglyDuckling\Framework\Routing\RouteTable;
use Fabiom\UglyDuckling\Framework\Routing\RouteDispatcher;

RouteTable::load(__DIR__ . '/config/routes.php');

(new RouteDispatcher(
    $dbconnection, $logger, $securityChecker, $mailer, $pageStatus, $groupsIndex,
    notFoundTemplateFile: 'staticpage',
    notFoundViewFile: 'errors/notfound',
))->dispatch();
{% endhighlight %}

`RouteTable::load()` only reads a plain array — it does not touch any of your controller or component classes. `RouteDispatcher::dispatch()` resolves the current request's slug and instantiates exactly the one matching controller or component; every other class named in the table is left untouched, so registering hundreds of routes costs nothing per request beyond the array lookup.

An unmatched slug renders a shared 404 page (`StaticPageController`, `http_response_code(404)`) using the `notFoundTemplateFile`/`notFoundViewFile` you configure above. `'staticpage'` is a minimal built-in template (`src/Templates/staticpage.php`) that just requires your view file — write `errors/notfound.php` (or wherever `notFoundViewFile` points) as a normal, self-contained HTML page. Point `notFoundTemplateFile` at your own template instead if you want the 404 page wrapped in your app's full chrome (nav, sidebar, etc.).

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

---

## Filtering links by group

`RouteTable::isVisible(string $slug)` answers one narrow question: should the current session's group see a link to this slug at all? It's a visibility check for building menus and nav — not a substitute for `check_authorization_get_request()` / `check_authorization_resource_request()`, which still run at request time regardless of what any menu shows.

{% highlight php %}
use Fabiom\UglyDuckling\Framework\Routing\RouteTable;

if (RouteTable::isVisible('articles')) {
    echo '<a href="' . url_for('articles') . '">Articles</a>';
}
{% endhighlight %}

| Table state | Result |
|---|---|
| Slug not in the table at all | Visible — fails open, so nothing you haven't generated a route for is ever hidden |
| `allowedgroups` is `null` | Hidden — a JSON resource with no `"allowedgroups"` key at all |
| `allowedgroups` is `[]` | Visible to every group |
| `allowedgroups` is non-empty | Visible only if the session's group is in the list |

Each group's own menu JSON already calls this automatically when rendering `menu`/`submenu`/`rightmenu` — an item pointing at a `resource` or `controller` the current session's group can't open is dropped silently, rather than rendering a link that would just 403 when clicked. A dropdown whose every child gets dropped this way is itself dropped. This means the menu JSON only has to describe the app's navigation *shape*; which items actually show up for a given group now follows from each item's own `$allowedGroups` / `"allowedgroups"`, instead of being kept in sync by hand across two separate lists.

The same check reaches every other place a JSON structure points at a route:

- **Grid and Tabs JSON resources** (`GridJsonTemplate`, `TabsJsonTemplate`) drop a panel entirely — no wrapper `<div>`, no empty tab — if its `resource` isn't visible.
- **Grid Page / Tabs Page components** (`BaseGridComponent`/`BaseTabsComponent`, via `BasePageComponent`) apply the equivalent check to `component`/`embed` nodes through each component's own `isAuthorized()` (see <a href="{{site.baseurl}}/docs/component#authorization">Component</a>) rather than a route-table lookup, since those nodes reference a PHP class directly instead of a slug — but the effect is identical: an unauthorized node, or a tab left with nothing authorized inside it, disappears rather than leaving an empty shell behind.
- **Buttons and links built from JSON** — anything registered in `index_json_tag_templates.php` (button bars, per-row table actions, plain links) — go through `JsonDefaultTemplateFactory::getHTMLTag()`, which checks the tag's own `resource`/`controller` (or, for AJAX-style buttons, the nested `dataudurl` object) the same way before ever instantiating the tag class. The check mirrors `UrlServices::make_resource_url()`'s own precedence for which of `resource`/`controller` is the real destination — `controller` normally wins when both are set, except when `controller` is the special `"partial"` marker, in which case `resource` is the actual target.

So a link, panel, tab, or component a group can't open is never part of the rendered interface — not shown disabled, not shown and left to 403 on click, just absent.
