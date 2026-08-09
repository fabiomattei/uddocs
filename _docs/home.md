---
layout: page
name: Home
---

# Welcome

Here you can find some documentation for the framework behind the code

## Templates

* <a href="{{site.baseurl}}/docs/chartjs">Chart js</a>
* <a href="{{site.baseurl}}/docs/dashboard">Dashboard</a>
* <a href="{{site.baseurl}}/docs/datatable">Datatable</a>
* <a href="{{site.baseurl}}/docs/group">Group</a>
* <a href="{{site.baseurl}}/docs/info">Info</a>
* <a href="{{site.baseurl}}/docs/form">Form</a>
* <a href="{{site.baseurl}}/docs/tabbed-page">Tabbed page</a>
* <a href="{{site.baseurl}}/docs/table-page">Table</a>
* <a href="{{site.baseurl}}/docs/title-bar">Title</a>
* <a href="{{site.baseurl}}/docs/transaction">Transaction</a>
* <a href="{{site.baseurl}}/docs/validation">Validation</a>
* <a href="{{site.baseurl}}/resources/jsonindex">Index</a>

## Database

* <a href="{{site.baseurl}}/docs/dao">DAO</a> — query and write to the database through PDO prepared statements
* <a href="{{site.baseurl}}/docs/migrations">Migrations</a> — version-controlled schema changes, similar to Laravel's migrations
* <a href="{{site.baseurl}}/docs/seeders">Seeders</a> — load data into a migrated database, similar to Laravel's seeders
* <a href="{{site.baseurl}}/docs/queue">Queue</a> — a database-backed job queue for deferring work outside the request/response cycle
* <a href="{{site.baseurl}}/docs/schema-query-validator">Schema query validator</a> — catch JSON resource queries that no longer match the database schema, before they run

## Component system

The component system lets you build pages in pure PHP by composing small, self-contained classes instead of editing JSON files. It is the right choice when a page requires custom business logic, complex rendering, or interactions that go beyond what a JSON resource can express.

* <a href="{{site.baseurl}}/docs/component">Component</a> — a single reusable unit: fetches data, handles POST, renders HTML
* <a href="{{site.baseurl}}/docs/declarative-components">Declarative leaf components</a> — form, info, and table panels rendered from a `$fields` array instead of a hand-written `render()`
* <a href="{{site.baseurl}}/docs/page-grid">Page (Grid layout)</a> — assembles components in a Bootstrap grid
* <a href="{{site.baseurl}}/docs/page-tabs">Page (Tabs layout)</a> — assembles components in a Bootstrap tabbed interface

## Extending the system

You do not need to limit your UD-programming to the ready to go solutions, you can create your own. 
You can create custom controllers, custom json templates and custom HTML Blocks. 
Each of the the following sections is going to drive you to extend your software using the path you like
the most.

* <a href="{{site.baseurl}}/docs/controller">Custom Controller</a>
* <a href="{{site.baseurl}}/docs/jsontemplate">Custom Json template</a>
* <a href="{{site.baseurl}}/docs/htmlblock">Custom HTML Block</a>
* <a href="{{site.baseurl}}/docs/resource-hooks">Resource Hooks</a> — attach beforeSave/afterFetch logic to a resource by naming convention, no JSON declaration needed
