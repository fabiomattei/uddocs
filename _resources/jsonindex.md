---
layout: page
name: Index
---

# Index

The index system is the map that tells UD where every resource file lives and what type it is.
At boot time the framework reads the root `index.json`, walks every linked index recursively, and builds an in-memory table that maps each resource **name** to its file path and type.
After this walk, the framework can resolve any `"resource": "my-resource-name"` reference from a menu, an action, or a redirect without knowing the physical path.

---

## Directory layout

Resources are organised in folders. Each folder has its own `index.json` that lists only the resources in that folder. The root `index.json` links to every folder index. This keeps each file short and gives a clean one-folder-per-feature structure.

```
Json/
├── index.json                ← root index: links to every folder index
├── session.json              ← session bootstrap config (see below)
├── groups/
│   ├── index.json            ← lists all group files
│   ├── authorgroup.json
│   └── admingroup.json
├── articles/
│   ├── index.json            ← lists all article resources
│   ├── articles-table.json
│   ├── article-new-form.json
│   └── article-edit-form.json
└── dashboard/
    ├── index.json
    └── main-dashboard.json
```

---

## Root `index.json`

The root index links to every folder index using `"type":"index"`. It may also list a small number of global resources directly.

{% highlight json %}
{
  "name": "index",
  "metadata": { "type":"index", "version": "1" },
  "scripts": [
    { "path":"./Json/articles/index.json",  "type":"index" },
    { "path":"./Json/dashboard/index.json", "type":"index" },
    { "path":"./Json/groups/index.json",    "type":"index" },
    { "path":"./Json/empty.json",           "name":"empty" }
  ]
}
{% endhighlight %}

Index entries do not need a `name` field because they are not addressable resources — they are just pointers to more index files.

---

## Folder `index.json`

Each folder index lists the resources inside that folder. Every resource entry needs a **path**, a **type**, and a **name**.

{% highlight json %}
{
  "name": "index",
  "metadata": { "type":"index", "version": "1" },
  "scripts": [
    { "path":"./Json/articles/articles-table.json",    "type":"table",  "name":"articles-table" },
    { "path":"./Json/articles/article-new-form.json",  "type":"form",   "name":"article-new-form" },
    { "path":"./Json/articles/article-edit-form.json", "type":"form",   "name":"article-edit-form" },
    { "path":"./Json/articles/article-delete.json",    "type":"transaction", "name":"article-delete" }
  ]
}
{% endhighlight %}

### Entry fields

| Field | Required | Description |
|---|---|---|
| `path` | yes | Path to the resource file, relative to the application root. |
| `type` | yes | Tells the framework which parser/renderer to use. See the full list below. |
| `name` | yes (except for `index` type) | Globally unique identifier for this resource. All menus, actions, and redirects reference resources by this name. |

---

## Resource types

The `type` field in an index entry tells the framework how to interpret the resource file.
All resource types correspond to a JSON-driven renderer except `index`, `group`, and `page`.

| Type | Description |
|---|---|
| `index` | Links to another index file. The framework follows it recursively. No `name` needed. |
| `group` | A user group definition: default action, navigation menu, access rights. See <a href="{{site.baseurl}}/resources/group">Group</a>. |
| `table` | A table rendered with the standard UD table renderer. |
| `datatable` | A table rendered with the DataTables JS library. |
| `form` | A form with GET pre-fill and POST transaction. See <a href="{{site.baseurl}}/resources/form">Form</a>. |
| `info` | A read-only panel that displays data from a query. See <a href="{{site.baseurl}}/resources/info">Info</a>. |
| `dashboard` | A page composed of several panels, each backed by another resource. See <a href="{{site.baseurl}}/resources/dashboard">Dashboard</a>. |
| `grid` | A Bootstrap-grid page that assembles resource panels. |
| `transaction` | A resource that only runs database queries; renders no page. See <a href="{{site.baseurl}}/resources/transaction">Transaction</a>. |
| `export` | A resource that queries the database and streams a CSV or Excel file to the browser. |
| `calendar` | A small calendar widget backed by a query. |
| `bigcalendar` | A full-page calendar (e.g. FullCalendar). |
| `chartjs` | A Chart.js chart backed by a query. See <a href="{{site.baseurl}}/resources/chartjs">Chart js</a>. |
| `page` | A custom PHP controller page, registered here so its name is addressable. |
| `logic` | A set of server-side logic rules applied to query results. See <a href="{{site.baseurl}}/resources/logics">Logics</a>. |
| `titlebar` | A title/header bar panel. |
| `tabbedpage` | A page with Bootstrap tabs, each containing a resource. |

---

## The global name namespace

Every `name` field across all index files shares a single namespace. The framework builds a flat map:
```
name → { path, type }
```

This means two resources in different folders cannot have the same name. It also means any part of the application can reference any resource by name without knowing its folder:

{% highlight json %}
{ "label":"Edit", "resource":"article-edit-form", "parameters":[{"name":"id", "sqlfield":"id"}] }
{% endhighlight %}

Choosing names carefully avoids collisions. A common convention is to prefix with the module name: `articles-table`, `articles-new-form`, `articles-edit-form`.

---

## Groups folder

The `Json/groups/` folder holds one file per user group and an `index.json` that lists them all.

{% highlight json %}
{
  "name": "index",
  "metadata": { "type":"index", "version": "1" },
  "scripts": [
    { "path":"./Json/groups/authorgroup.json",  "type":"group", "name":"authorgroup" },
    { "path":"./Json/groups/admingroup.json",   "type":"group", "name":"admingroup" }
  ]
}
{% endhighlight %}

Each group file defines the navigation menu and the landing page for users belonging to that group.
Full group documentation: <a href="{{site.baseurl}}/resources/group">Group</a>.

---

## `session.json`

`session.json` is a special configuration file (not listed in any index) that the framework reads at login time to initialise the session. It runs queries and stores the results as session variables.

{% highlight json %}
{
  "name": "setup",
  "metadata": { "type":"setup", "version": "1" },
  "queryset": [
    {
      "label": "userquery",
      "sql": "SELECT usr_defaultgroup FROM ud_users WHERE usr_id = :usrid",
      "parameters": [
        { "type":"long", "placeholder": ":usrid", "sessionparameter": "user_id" }
      ]
    }
  ],
  "sessionvars": [
    { "name":"user_id",    "system":"ud" },
    { "name":"username",   "system":"ud" },
    { "name":"group",      "system":"ud" },
    { "name":"logged_in",  "system":"ud" },
    { "name":"ip",         "system":"ud" },
    { "name":"last_login", "system":"ud" },
    { "name":"usr_type",   "sqlfield":"usr_defaultgroup", "querylabel":"userquery" }
  ]
}
{% endhighlight %}

### `queryset`

A list of named SQL queries run immediately after the user authenticates.
Each query can use session parameters (values already set by the UD framework itself) as placeholders.

| Field | Description |
|---|---|
| `label` | Identifies this query result for use in `sessionvars`. |
| `sql` | Plain SQL with `:placeholder` substitution. |
| `parameters` | Parameter bindings. Use `"sessionparameter"` to bind a value from an existing session variable. |

### `sessionvars`

A list of session variables to populate.

| Property | Description |
|---|---|
| `name` | The session key (accessible as `$_SESSION['name']`). |
| `system: "ud"` | Populated automatically by the UD authentication system (user_id, group, username, logged_in, ip, last_login). |
| `sqlfield` + `querylabel` | Populated from a specific column of a named queryset result. |

---

## Complete example

A real application with three modules typically looks like this:

Root `Json/index.json`:

{% highlight json %}
{
  "name": "index",
  "metadata": { "type":"index", "version": "1" },
  "scripts": [
    { "path":"./Json/articles/index.json",   "type":"index" },
    { "path":"./Json/comments/index.json",   "type":"index" },
    { "path":"./Json/groups/index.json",     "type":"index" }
  ]
}
{% endhighlight %}

`Json/articles/index.json`:

{% highlight json %}
{
  "name": "index",
  "metadata": { "type":"index", "version": "1" },
  "scripts": [
    { "path":"./Json/articles/articles-table.json",     "type":"table",       "name":"articles-table" },
    { "path":"./Json/articles/article-new-form.json",   "type":"form",        "name":"article-new-form" },
    { "path":"./Json/articles/article-edit-form.json",  "type":"form",        "name":"article-edit-form" },
    { "path":"./Json/articles/article-delete.json",     "type":"transaction", "name":"article-delete" },
    { "path":"./Json/articles/articles-dashboard.json", "type":"dashboard",   "name":"articles-dashboard" }
  ]
}
{% endhighlight %}

`Json/groups/index.json`:

{% highlight json %}
{
  "name": "index",
  "metadata": { "type":"index", "version": "1" },
  "scripts": [
    { "path":"./Json/groups/authorgroup.json", "type":"group", "name":"authorgroup" },
    { "path":"./Json/groups/admingroup.json",  "type":"group", "name":"admingroup" }
  ]
}
{% endhighlight %}

After boot, the framework's name map contains: `articles-table`, `article-new-form`, `article-edit-form`, `article-delete`, `articles-dashboard`, `authorgroup`, `admingroup` — all globally addressable by any action, menu item, or redirect in the application.
