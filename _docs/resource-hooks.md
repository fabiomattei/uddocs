---
layout: page
name: Resource Hooks
---

# Description

A **Resource Hook** is a PHP class that attaches business logic to a single JSON resource by naming convention alone — no `usecases` entry, no registration array. Drop a PHP file next to the resource's JSON file, name it the same way, and UD picks it up automatically.

It exists for the moments a JSON resource can't express on its own. A Resource Hook exposes one method per phase of a resource's GET/POST lifecycle — session updates, use cases, transactions, redirects, and so on — plus authorization and per-row rendering. Override only the phases you need; everything else defaults to a no-op ("proceed", "change nothing").

This is a different tool from <a href="{{site.baseurl}}/baseresources/usecase">Use Cases</a>: a use case is declared explicitly in the JSON (`"usecases": [...]`), takes named parameters, and is itself a distinct phase. A Resource Hook needs no JSON at all and sits around every phase of the *same* resource's request handling — reach for it when the logic is intrinsic to *this* resource (a cross-field validation, a computed display column, an extra authorization rule) rather than a reusable operation you'd want to name and parameterize from JSON.

---

## Enabling it

Resource Hooks are off by default. Define `RESOURCE_HOOKS_NAMESPACE` in your application's bootstrap to turn them on — it's the namespace UD looks in for hook classes:

{% highlight php %}
define('RESOURCE_HOOKS_NAMESPACE', 'MyApp\\ResourceHooks');
{% endhighlight %}

Without this constant defined, resource hook resolution is skipped entirely — existing applications are unaffected.

---

## Naming convention

For a resource `order_form.json`, create `order_form.php` in the **same directory**, containing a class named **exactly `order_form`** (same as the file, literally — no case transformation, no suffix) in the configured namespace. This is deliberate: IDEs like PhpStorm/IntelliJ only link a file to the class it declares when the two names match exactly, so the hook file gets the same file↔class navigation as any other PHP class in the project.

{% highlight php %}
namespace MyApp\ResourceHooks;

use Fabiom\UglyDuckling\Framework\ResourceHooks\BaseResourceHook;

class order_form extends BaseResourceHook {

    public function beforeTransactions(array &$data): bool {
        if ($data['country'] === 'IT' && empty($data['vat_number'])) {
            $this->pageStatus->addError('VAT number is required for Italian customers.');
            return false;
        }

        if ($data['order_type'] === 'premium') {
            $data['discount_rate'] = 0.15;
        }

        return true;
    }

    public function beforeRenderGet(\stdClass $entity): void {
        $entity->formatted_total = number_format($entity->total_amount, 2, ',', '.') . ' €';
    }

}
{% endhighlight %}

The only transformation applied is `-` → `_`, since `-` is legal in a resource file name (e.g. `my-users-list.json`) but not in a PHP identifier: `my-users-list.php` must contain a class named `my_users_list`.

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

Every method is optional to override.

---

## The phases

Each method fires right at the start of the matching phase — and only if that phase actually exists in the resource's JSON (there's nothing to gate if, say, the resource has no `transactions` block). Boolean-returning methods can only *restrict*: default is `true` ("proceed"), and returning `false` blocks just that phase. A hook can never grant something the standard checks would otherwise deny.

| Method | Fires before... | Verb |
|---|---|---|
| `beforeAuthorization(): bool` | access is granted, once the resource's `allowedgroups` check already passed | GET & POST |
| `beforeSessionUpdatesGet(): void` | `get.sessionupdates` is applied | GET |
| `beforeUseCasesGet(): void` | `get.usecases` run | GET |
| `beforeRenderGet(\stdClass $entity): void` | each fetched row/entity is mapped into the rendered table/form | GET |
| `beforeFileUploads(): void` | `post.fileuploads` are processed | POST |
| `beforeTransactions(array &$data): bool` | `post.transactions` execute | POST |
| `beforeInplaceEdits(array &$data): bool` | `post.inplaceeditor` updates execute | POST |
| `beforeUseCasesPost(): void` | `post.usecases` run | POST |
| `beforeSessionUpdatesPost(): void` | `post.sessionupdates` is applied | POST |
| `beforeRedirect(): void` | `post.redirect` is followed | POST |
| `beforeAjaxResponse(): void` | `post.ajaxreponses` is echoed | POST |

---

## `beforeAuthorization`

Runs after the resource's `allowedgroups` check already passed, and can still deny access:

{% highlight php %}
public function beforeAuthorization(): bool {
    return $this->pageStatus->getValue((object) ['sessionparameter' => 'impersonating']) === null;
}
{% endhighlight %}

It cannot override a denial from the standard `allowedgroups` check — this method is only ever consulted once that check already allowed the request, consistent with default-deny.

---

## `beforeTransactions` / `beforeInplaceEdits`

`$data` is the resource's resolved POST parameters — the same array `transactions` and `inplaceeditor` pull their bound SQL values from, so mutating it changes what actually gets written to the database:

{% highlight php %}
public function beforeTransactions(array &$data): bool {
    $data['slug'] = strtolower(str_replace(' ', '-', $data['title']));
    return true;
}
{% endhighlight %}

They gate independently: returning `false` from `beforeTransactions` skips only `post.transactions`; returning `false` from `beforeInplaceEdits` skips only `post.inplaceeditor`. Call `$this->pageStatus->addError(...)` before returning `false` so the rejection has a message; the resource's existing ajax error handling picks it up automatically, the same way it already does for a failed `transactions` query.

---

## `beforeRenderGet`

Called once per row for a table resource, and once for a form resource's single entity — always after the row is fetched, before any field in the JSON's `fields` array is read for rendering:

{% highlight php %}
public function beforeRenderGet(\stdClass $entity): void {
    $entity->days_open = (new \DateTime($entity->created_at))->diff(new \DateTime())->days;
}
{% endhighlight %}

Add `days_open` as a plain `sqlfield`-less field in the resource's `fields` array (pointing at a constant or the same name) to display it — `beforeRenderGet` only produces the value, the JSON still declares where it's shown.

---

## Where it's wired in

`beforeRenderGet` fires for every table and form resource, regardless of which controller served the request, because it's resolved inside the shared `TableJsonTemplate`/`FormJsonTemplate` classes.

Every other phase hook — `beforeAuthorization`, the `Get`/`Post` session-updates and use-cases hooks, `beforeFileUploads`, `beforeTransactions`, `beforeInplaceEdits`, `beforeRedirect`, `beforeAjaxResponse` — is wired into both `JsonResourceController` (the ordinary table/form/transaction flow) and `JsonResourcePartialBasicController` (embedded page-section resources). The same hook class serves a resource regardless of which of the two controllers ends up handling the request; each controller resolves and calls the phases it actually has (`JsonResourcePartialBasicController` has no `fileuploads`/`inplaceeditor`/`redirect` blocks, so those methods never fire there).
