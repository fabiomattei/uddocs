---
layout: page
name: Visibility
title: Visibility
---

# Description

**allowedgroups**, **visibleIf** and **visibleIfHook** let you attach declarative visibility rules to an individual element in a resource — an <a href="{{site.baseurl}}/baseresources/actions">action</a> (link/button), a <a href="{{site.baseurl}}/resources/table-page">table</a> field, or a <a href="{{site.baseurl}}/resources/form">form</a> field — without writing PHP in the common cases, and without leaving the JSON at all in the complex ones.

This is a different, more granular check than the resource-level `allowedgroups` (the one at the top of every resource file, which decides whether the whole resource is reachable). These decide whether one link, one column, or one field is shown, once the surrounding resource is already accessible.

| Property | Applies to | Checked against |
|---|---|---|
| `allowedgroups` | actions, table fields, form fields | the current session's group |
| `visibleIf` | actions, table fields, form fields | the current row/entity |
| `visibleIfHook` | actions, table fields, form fields | whatever a <a href="{{site.baseurl}}/docs/resource-hooks">Resource Hook</a> method decides |

---

## allowedgroups

Absent or `[]` means visible to everyone; otherwise the element is shown only if the current session's group is in the list — the same comparison the resource-level `allowedgroups` already uses.

{% highlight json %}
{ "type": "link", "label": "Elimina Progetto", "url": "/delete-project", "allowedgroups": ["admin", "supermanager"] }
{% endhighlight %}

On a table field, `allowedgroups` masks the **whole column** — header included — the same way for every row, since it's a session check, not a row check:

{% highlight json %}
{ "headline": "Margine Guadagno", "sqlfield": "margin_amount", "allowedgroups": ["director", "admin"] }
{% endhighlight %}

A masked table column still renders an empty `<th>`/`<td>` rather than being omitted — dropping the cell would misalign every column after it against the header row. A masked form field is simply skipped; a form isn't a fixed-column grid, so there's no alignment to protect.

---

## visibleIf

A condition evaluated against the current row (`table`) or entity (`form`). Leaf conditions:

{% highlight json %}
{
  "type": "link",
  "label": "Invia Sollecito",
  "url": "/send-reminder",
  "visibleIf": { "field": "status", "equals": "pending_payment" }
}
{% endhighlight %}

`equals`, `not_equals`, `less_than`, `greater_than` are the available comparisons. Conditions nest with `AND`/`OR`:

{% highlight json %}
{
  "type": "link",
  "label": "Approva Preventivo",
  "url": "/approve",
  "visibleIf": {
    "OR": [
      { "AND": [
        { "field": "status", "equals": "pending" },
        { "field": "total_amount", "less_than": 5000 }
      ]},
      { "field": "user_role", "equals": "admin" }
    ]
  }
}
{% endhighlight %}

On a table field, the same "never drop the cell" rule as `allowedgroups` applies — but here it's per row, so the header always renders normally and only the individual `<td>`s that fail the condition go empty. `table->topactions`/`table->bottomactions` aren't tied to a row, so `visibleIf` there has no meaningful row to check against — use `allowedgroups` for those instead.

---

## visibleIfHook

Once a condition needs real `AND`/`OR` nesting plus business rules that don't reduce to a handful of comparisons, deep JSON trees stop being readable. `visibleIfHook` delegates the decision to a named method on the resource's co-located <a href="{{site.baseurl}}/docs/resource-hooks">Resource Hook</a> class instead — write it as plain PHP with the IDE's full support, no expression tree to keep in sync.

{% highlight json %}
{ "type": "link", "label": "Approva Preventivo", "url": "/approve", "visibleIfHook": "canApproveInvoice" }
{% endhighlight %}

{% highlight php %}
class OrderFormResourceHook extends BaseResourceHook {

    public function canApproveInvoice(\stdClass $rowData): bool {
        return ($rowData->status === 'pending' && $rowData->total_amount < 5000)
            || ($_SESSION['group'] ?? null) === 'admin';
    }

}
{% endhighlight %}

The method receives the current row/entity as a `\stdClass`, same as `afterFetch()` on a Resource Hook. If `visibleIfHook` names a method that doesn't exist on the resolved hook — or no hook file exists for the resource at all — UD throws rather than silently showing or hiding the element, since that's a resource/hook mismatch, not a runtime visibility decision.

## Choosing between visibleIf and visibleIfHook

Use `visibleIf` while the condition is a couple of comparisons, optionally combined with `AND`/`OR` — it keeps the whole resource declarative and inspectable from the JSON alone. Move to `visibleIfHook` as soon as the tree gets hard to read at a glance; the 90/10 split holds here the same way it does for <a href="{{site.baseurl}}/baseresources/usecase">Use Cases</a> — most resources cover the linear 90% in JSON, PHP resolves the algorithmic 10%.
