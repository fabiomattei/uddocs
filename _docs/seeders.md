---
layout: page
name: Seeders
---

# Seeders

UglyDuckling ships a small seeder system to load data into a migrated database, similar in spirit to Laravel's seeders. Each seeder is a PHP file that declares a named class with a `run()` method — the same shape as a real Laravel seeder, not the anonymous-class convention migration files use; the same `ud-migrate` CLI tool used for [migrations](migrations) can run every seeder in a directory, or just one.

Seeder files live in **your application**, not in the library — the library only provides the base class, the runner, and the CLI.

Unlike migrations, seeders are **not tracked** as "already run". Running `ud-migrate seed` twice runs every seeder twice. A seeder is expected to be re-runnable by design — e.g. by truncating a table first, or using `INSERT ... ON DUPLICATE KEY` / `INSERT OR IGNORE` — the same way Laravel seeders are typically written to be safe to call repeatedly.

---

## Seeder skeleton

A seeder file declares a named class, under a fixed `Database\Seeders` namespace, extending `Seeder`. `run()` receives the raw PDO connection and is the first method in the class — exactly like a Laravel seeder:

{% highlight php %}
namespace Database\Seeders;

use Fabiom\UglyDuckling\Framework\DataBase\Seeders\Seeder;
use PDO;

class SeedAuthors extends Seeder {

    public function run( PDO $pdo ): void {
        foreach ( $this->authors() as $name ) {
            $pdo->exec( "INSERT INTO authors (name) VALUES (" . $pdo->quote( $name ) . ")" );
        }
    }

    private function authors(): array {
        return [
            'Italo Calvino',
            'Primo Levi',
        ];
    }

}
{% endhighlight %}

Keeping `run()` first and pushing any reference data or helpers into private methods below it (as above) is the recommended shape once a seeder needs more than a line or two of data — it keeps the actual logic at the top of the file instead of buried under a wall of constants.

The class name **must** match the filename with the timestamp prefix stripped and the remainder converted to StudlyCase — `2026_07_20_180602_seed_authors.php` must declare `SeedAuthors`. `ud-migrate make-seeder` (below) gets this right automatically; if you rename a seeder file by hand, rename the class to match. Two seeder files that produce the same class name once their timestamps are stripped (e.g. two "seed_authors" made on different days) will fatal on the second one, since both declare `Database\Seeders\SeedAuthors` — keep the descriptive part of the name unique.

The `Database\Seeders` namespace is a fixed convention, not a PSR-4 mapping — the runner `require`s the file directly, so nothing needs to be added to your application's `composer.json` autoload config.

There is no schema-building helper for seeders — `Schema`/`Blueprint` are migration-only concerns. Use `$pdo` directly, or a `BasicDao` subclass if one already exists for the table you're seeding.

---

## Creating a seeder

Use `ud-migrate make-seeder` to scaffold a new, timestamp-prefixed seeder file, pre-filled with the correctly-derived class name:

{% highlight bash %}
ud-migrate make-seeder seed_authors
# Created seeder: database/seeders/2026_07_20_180602_seed_authors.php
# -> class Database\Seeders\SeedAuthors
{% endhighlight %}

The timestamp prefix determines run order when seeding "all", so seeders that depend on rows inserted by another seeder (e.g. a foreign key) should be named so their timestamp sorts after it.

---

## Connecting to the database

`ud-migrate seed` uses the same `migrations-config.php` connection file as migrations (see [Migrations — Connecting to the database](migrations#connecting-to-the-database)). Both the config file path and the seeders directory can be overridden as extra arguments (see [Command reference](#command-reference)). By default:

| What | Default |
|---|---|
| Config file | `./migrations-config.php` |
| Seeders directory | `./database/seeders` |

---

## Command reference

### `ud-migrate seed [--class=Name] [config-file] [seeders-dir]`

With no `--class`, runs every seeder found in the directory, in filename order. Each seeder runs inside its own transaction; if one fails, its transaction is rolled back and the command stops.

{% highlight bash %}
ud-migrate seed
# Seeded: 2026_07_20_180602_seed_authors
# Seeded: 2026_07_20_180700_seed_books
{% endhighlight %}

With `--class=Name`, runs only the named seeder (the filename without `.php`):

{% highlight bash %}
ud-migrate seed --class=2026_07_20_180602_seed_authors
# Seeded: 2026_07_20_180602_seed_authors
{% endhighlight %}

### `ud-migrate make-seeder <name> [seeders-dir]`

Scaffolds a new seeder file from a stub, prefixed with the current timestamp (see [Creating a seeder](#creating-a-seeder)).

---

## Running seeders without the CLI

The `SeederRunner` class can be used directly from PHP — useful for seeding as part of a deploy script or a test bootstrap:

{% highlight php %}
use Fabiom\UglyDuckling\Framework\DataBase\Seeders\SeederRunner;

$pdo = $dbConnection->getDBH();

$seederRunner = new SeederRunner( $pdo, __DIR__ . '/database/seeders' );

$ran = $seederRunner->run();                              // string[] of seeder names that were run, in filename order
$seederRunner->runOne( '2026_07_20_180602_seed_authors' ); // run just one, by name
{% endhighlight %}
