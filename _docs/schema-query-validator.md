---
layout: page
name: Schema query validator
---

# Schema query validator

JSON resources embed raw SQL in `get.query.sql` (and, for forms, `post.query.sql`). Nothing checks those strings against the database until the resource is actually requested — a renamed or dropped column only surfaces as a runtime error, on whichever page happens to hit it first.

`SchemaQueryValidator` closes that gap: point it at a directory of JSON resource files and a real database connection, and it asks the database to `PREPARE` every query it finds, without executing any of them. A `PREPARE` fails on an unknown table or column exactly the way `execute()` would, so this catches schema drift ahead of time, offline, with no data required.

---

## Usage

{% highlight php %}
use Fabiom\UglyDuckling\Framework\DataBase\SchemaQueryValidator;

$validator = new SchemaQueryValidator($pdo);
$errors = $validator->validateDirectory(__DIR__ . '/Json');

foreach ($errors as $error) {
    echo $error, "\n";
}
{% endhighlight %}

`validateDirectory()` scans the given directory recursively for `*.json` files, and for each one checks `get.query.sql` and `post.query.sql` (whichever are present). It returns an array of `SchemaQueryValidationError` — empty means every query prepared cleanly.

| Property | Meaning |
|---|---|
| `file` | Path to the JSON resource file |
| `verb` | `'get'` or `'post'` (or `'json'`, if the file itself failed to parse) |
| `sql` | The query that failed to prepare |
| `message` | The underlying `PDOException` message |

`SchemaQueryValidationError` also implements `__toString()`, so printing one directly gives a one-line summary suitable for a test failure message or CLI output.

---

## Why the connection matters

The check is only as good as what's on the other end of `$pdo`:

* It must be a **real, current schema** — a stale copy will produce both false positives (columns you removed weeks ago) and false negatives (columns you haven't added yet). Building that connection — from your own migrations, or a fresh dump of a live database — is up to the caller; `SchemaQueryValidator` only runs the checks.
* On MySQL, the constructor forces `PDO::ATTR_EMULATE_PREPARES` off. PDO's emulated prepares only parse placeholder positions client-side and never ask the server anything — every query would silently "pass" regardless of whether the columns exist. Real (server-side) prepares are what actually validate against the schema. SQLite always prepares server-side, so this only matters for the `mysql` driver.

A convenient way to get a real, current schema without touching a shared database: run your own migrations (see <a href="{{site.baseurl}}/docs/migrations">Migrations</a>) against a throwaway SQLite in-memory connection, then validate against that:

{% highlight php %}
use Fabiom\UglyDuckling\Framework\DataBase\Migrations\MigrationRepository;
use Fabiom\UglyDuckling\Framework\DataBase\Migrations\Migrator;
use Fabiom\UglyDuckling\Framework\DataBase\SchemaQueryValidator;

$pdo = new PDO('sqlite::memory:');
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

$migrator = new Migrator($pdo, new MigrationRepository($pdo), __DIR__ . '/database/migrations');
$migrator->migrate();

$errors = (new SchemaQueryValidator($pdo))->validateDirectory(__DIR__ . '/Json');
{% endhighlight %}

If your migrations use MySQL-specific SQL that `Schema`/`Blueprint` doesn't abstract away, or your schema isn't tracked through this library's migrations at all, point `SchemaQueryValidator` at a real MySQL connection instead — a disposable database restored from a structure-only dump of your actual schema is the most reliable source of truth.

---

## Scope

* Only `get.query.sql` and `post.query.sql` inside JSON resource files are checked. Controllers and components that build SQL themselves are not scanned — keep that SQL inside a DAO (see <a href="{{site.baseurl}}/docs/dao">DAO</a>) where it's covered by your own tests instead.
* A JSON file that fails to `json_decode` is reported as one error with `verb = 'json'`, rather than aborting the whole scan — one broken resource file doesn't hide problems in the rest.
* A file whose `get`/`post` block has no `query.sql` (static pages, info panels without a query, etc.) is skipped, not reported.
