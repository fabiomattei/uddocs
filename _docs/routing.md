---
layout: page
name: Linking & Visibility
---

# Linking & Visibility

UglyDuckling has no built-in URL router. A <a href="{{site.baseurl}}/docs/controller">Controller</a>, standalone <a href="{{site.baseurl}}/docs/component">Component</a>, or <a href="{{site.baseurl}}/docs/page-grid">Grid</a>/<a href="{{site.baseurl}}/docs/page-tabs">Tabs</a> page becomes reachable over HTTP however your application wires it up — there is no attribute, generator command, or route table involved.

---

## Wiring a class to a URL

The simplest approach is one PHP file per page, requested directly (e.g. `articles-edit.html` served by `articles-edit.php`), which instantiates the class and drives the same lifecycle:

{% highlight php %}
// articles-edit.php
$controller = new ArticleEditController();
$controller->setPageStatus($pageStatus);
$controller->setGroupsIndex($groupsIndex);
$controller->makeAllPresets($dbconnection, $logger, $securityChecker, $mailer);
$controller->showPage();
{% endhighlight %}

This works identically for a Grid or Tabs page, since both extend `BaseController` too. A standalone Component uses `renderAsPanel()`/`handlePost()` instead of `showPage()`:

{% highlight php %}
// articles-list.php
$component = new ArticlesList();
$component->pageStatus = $pageStatus;
if (ServerWrapper::isGetRequest()) {
    $component->renderAsPanel();
} else {
    $component->handlePost();
}
{% endhighlight %}

If you'd rather not write one file per page, `index_components.php` gives Components a slug-based registry without that boilerplate — see <a href="{{site.baseurl}}/docs/component#registering-a-component-as-a-standalone-page">Registering a component as a standalone page</a>. There is no equivalent registry for Controllers or Grid/Tabs pages; wire those up individually, or route to them from your own front controller if you have one.

An unmatched slug is likewise your application's own concern — point your web server or front controller at a `StaticPageController` for the 404 case, the same pattern used for a 403 in <a href="{{site.baseurl}}/docs/controller#unauthorized-access">Controller</a>.

---

## Building links

There's no link-building helper for Controllers, Components, or pages — write the URL yourself, following whatever slug convention you wired up above:

{% highlight php %}
<a href="articles-edit.html?id=<?= $article->id ?>">Edit</a>
{% endhighlight %}

{% highlight php %}
$this->redirectToPage('articles.html');
$this->postSuccessUrl = 'articles.html';
{% endhighlight %}

JSON-driven links — menu items, Grid/Tabs JSON panels, buttons (see <a href="{{site.baseurl}}/docs/json-template">JSON Template</a>) — are the exception: they go through `UrlServices::make_resource_url()`, which builds the same `slug.html` URLs from a JSON action's `resource`/`controller`/`url` fields. You never call this directly; the JSON template layer does it for you.

---

## Filtering links by group

`ResourceVisibility::isVisible(?string $slug)` answers one narrow question: should the current session's group see a link to this slug at all? It's a visibility check for building menus and nav — not a substitute for `check_authorization_get_request()` / `check_authorization_resource_request()`, which still run at request time regardless of what any menu shows.

{% highlight php %}
use Fabiom\UglyDuckling\Framework\Json\Visibility\ResourceVisibility;

if (ResourceVisibility::isVisible('articles')) {
    echo '<a href="articles.html">Articles</a>';
}
{% endhighlight %}

Load it once at bootstrap from the same slug => JSON-file registry used for JSON resources elsewhere:

{% highlight php %}
ResourceVisibility::load($resourceIndex); // e.g. the array from index_json_resources.php
{% endhighlight %}

`ResourceVisibility` only knows about JSON resources — Controllers and Components have no central slug registry for it to consult, so it can't evaluate them at all:

| Index state | Result |
|---|---|
| Slug not a JSON resource (a Controller/Component slug, or unregistered) | Visible — fails open; those enforce their own authorization at request time |
| `allowedgroups` absent on the resource | Hidden — fail-closed, mirrors `JsonResourceController::check_authorization_resource_request()` |
| `allowedgroups` is `[]` | Visible to every group |
| `allowedgroups` is non-empty | Visible only if the session's group is in it |

Each group's own menu JSON already calls this automatically when rendering `menu`/`submenu`/`rightmenu` — an item pointing at a JSON resource the current session's group can't open is dropped silently, rather than rendering a link that would just 403 when clicked. An item pointing at a Controller or Component slug is always kept, since `ResourceVisibility` has no way to evaluate it; group-restrict those from the menu by hand (build the menu JSON per-group, or check `$allowedGroups` yourself before including the item) if you need the same effect. A dropdown whose every child gets dropped this way is itself dropped.

The same check reaches every other place a JSON structure points at a route:

- **Grid and Tabs JSON resources** (`GridJsonTemplate`, `TabsJsonTemplate`) drop a panel entirely — no wrapper `<div>`, no empty tab — if its `resource` isn't visible.
- **Grid Page / Tabs Page components** (`BaseGridComponent`/`BaseTabsComponent`, via `BasePageComponent`) apply the equivalent check to `component`/`embed` nodes through each component's own `isAuthorized()` (see <a href="{{site.baseurl}}/docs/component#authorization">Component</a>) rather than a `ResourceVisibility` lookup, since those nodes reference a PHP class directly instead of a slug — but the effect is identical: an unauthorized node, or a tab left with nothing authorized inside it, disappears rather than leaving an empty shell behind.
- **Buttons and links built from JSON** — anything registered in `index_json_tag_templates.php` (button bars, per-row table actions, plain links) — go through `JsonDefaultTemplateFactory::getHTMLTag()`, which checks the tag's own `resource`/`controller` (or, for AJAX-style buttons, the nested `dataudurl` object) the same way before ever instantiating the tag class. Since `ResourceVisibility` fails open on anything that isn't a JSON resource, this only actually filters `resource`-targeted tags; a `controller`-targeted tag is always kept.

So a JSON resource a group can't open is never part of the rendered interface — not shown disabled, not shown and left to 403 on click, just absent. Controllers and Components don't get this for free from any of the above; only their own request-time authorization protects them, and any menu link to one is shown regardless of group.
