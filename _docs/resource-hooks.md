---
layout: page
name: Resource Hooks
---

# Description

A **Resource Hook** is a PHP class that attaches business logic to a single JSON resource by naming convention alone — no `usecases` entry, no registration array. Drop a PHP file next to the resource's JSON file, name it the same way, and UD picks it up automatically.

It exists for the two moments a JSON resource can't express on its own:

* **`beforeSave($data)`** — runs right before a POST resource's `transactions` (and `inplaceeditor` updates) execute. Mutate `$data` to change what gets saved, or return `false` to block the save entirely.
* **`afterFetch($entity)`** — runs right after a table row or form entity is fetched from the database, before it's mapped into the rendered HTML. Mutate `$entity`'s properties to add or reshape fields for display.

This is a different tool from <a href="{{site.baseurl}}/baseresources/usecase">Use Cases</a>: a use case is declared explicitly in the JSON (`"usecases": [...]`), takes named parameters, and runs at a fixed point (before a GET's HTML block, after a POST's transactions). A Resource Hook needs no JSON at all and sits specifically around the save/fetch boundary — reach for it when the logic is intrinsic to *this* resource (a cross-field validation, a computed display column) rather than a reusable operation you'd want to name and parameterize from JSON.

---

## Enabling it

Resource Hooks are off by default. Define `RESOURCE_HOOKS_NAMESPACE` in your application's bootstrap to turn them on — it's the namespace UD looks in for hook classes:

{% highlight php %}
define('RESOURCE_HOOKS_NAMESPACE', 'MyApp\\ResourceHooks');
{% endhighlight %}

Without this constant defined, resource hook resolution is skipped entirely — existing applications are unaffected.

---

## Naming convention

For a resource `order_form.json`, create `order_form.php` in the **same directory**, containing a class named `OrderFormResourceHook` in the configured namespace:

{% highlight php %}
namespace MyApp\ResourceHooks;

use Fabiom\UglyDuckling\Framework\ResourceHooks\BaseResourceHook;

class OrderFormResourceHook extends BaseResourceHook {

    public function beforeSave(array &$data): bool {
        if ($data['country'] === 'IT' && empty($data['vat_number'])) {
            $this->pageStatus->addError('VAT number is required for Italian customers.');
            return false;
        }

        if ($data['order_type'] === 'premium') {
            $data['discount_rate'] = 0.15;
        }

        return true;
    }

    public function afterFetch(\stdClass $entity): void {
        $entity->formatted_total = number_format($entity->total_amount, 2, ',', '.') . ' €';
    }

}
{% endhighlight %}

The class name is derived from the resource name: dashes and underscores become spaces, each word is capitalized, then `ResourceHook` is appended. `order_form` → `OrderFormResourceHook`, `bowtie-add-author-form` → `BowtieAddAuthorFormResourceHook`.

`BaseResourceHook`'s constructor already gives you everything you'd otherwise have to pull off `$pageStatus`:

{% highlight php %}
abstract class BaseResourceHook {
    public function __construct(
        protected \PDO $dbh,
        protected Logger $logger,
        protected PageStatus $pageStatus
    ) {}
}
{% endhighlight %}

Both methods are optional to override — the base class defaults to "allow the save" / "do nothing to the fetched entity".

---

## `beforeSave`

`$data` is the resource's resolved POST parameters — the same array `transactions` and `inplaceeditor` pull their bound SQL values from, so mutating it changes what actually gets written to the database:

{% highlight php %}
public function beforeSave(array &$data): bool {
    $data['slug'] = strtolower(str_replace(' ', '-', $data['title']));
    return true;
}
{% endhighlight %}

Returning `false` skips the resource's `transactions` and `inplaceeditor` blocks — no SQL runs. Call `$this->pageStatus->addError(...)` before returning `false` so the rejection has a message; the resource's existing ajax error handling picks it up automatically, the same way it already does for a failed `transactions` query. Everything after that point in the request (`usecases`, `sessionupdates`, `redirect`, `ajaxreponses`) still runs unchanged.

---

## `afterFetch`

Called once per row for a table resource, and once for a form resource's single entity — always after the row is fetched, before any field in the JSON's `fields` array is read for rendering:

{% highlight php %}
public function afterFetch(\stdClass $entity): void {
    $entity->days_open = (new \DateTime($entity->created_at))->diff(new \DateTime())->days;
}
{% endhighlight %}

Add `days_open` as a plain `sqlfield`-less field in the resource's `fields` array (pointing at a constant or the same name) to display it — `afterFetch` only produces the value, the JSON still declares where it's shown.

---

## Where it's wired in

`afterFetch` fires for every table and form resource, regardless of which controller served the request, because it's resolved inside the shared `TableJsonTemplate`/`FormJsonTemplate` classes.

`beforeSave` currently only fires for POST requests handled by the standard `JsonResourceController` — the controller behind the ordinary table/form/transaction flow. It is not yet wired into the partial or small-partial controller variants.
