---
layout: page
name: Migrations
---

# Migrations

UglyDuckling ships a small migration system to manage database schema changes over time, similar in spirit to Laravel's migrations. Each migration is a PHP file that returns a class with an `up()` and a `down()` method; a CLI tool (`ud-migrate`) tracks which migrations have run and applies the ones that haven't.

Migration files live in **your application**, not in the library — the library only provides the base class, the runner, and the CLI.

---

## Migration skeleton

A migration file returns an anonymous class extending `Migration`. Both methods receive the raw PDO connection — there is no query builder, only SQL, consistent with the rest of UglyDuckling.

{% highlight php %}
use Fabiom\UglyDuckling\Framework\DataBase\Migrations\Migration;

return new class extends Migration {

    public function up( PDO $pdo ): void {
        $pdo->exec('CREATE TABLE books (
            bk_id INT(11) UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
            bk_title VARCHAR(255) NOT NULL,
            bk_author VARCHAR(255) NOT NULL,
            bk_updated DATETIME NOT NULL,
            bk_created DATETIME NOT NULL
        )');
    }

    public function down( PDO $pdo ): void {
        $pdo->exec('DROP TABLE books');
    }

};
{% endhighlight %}

---

## Creating a migration

Use `ud-migrate make` to scaffold a new, timestamp-prefixed migration file:

{% highlight bash %}
ud-migrate make create_books_table
# Created migration: database/migrations/2026_07_19_180602_create_books_table.php
{% endhighlight %}

The timestamp prefix is what determines run order, so migrations always apply in the order they were created.

---

## Connecting to the database

`ud-migrate` needs a PDO connection to run against. It looks for a `migrations-config.php` file in the current directory — a plain PHP file that must `return` a `PDO` instance:

{% highlight php %}
<?php
// migrations-config.php

$connection = new Fabiom\UglyDuckling\Framework\DataBase\DBConnection(
    'mysql:host=localhost;dbname=',
    'my_database',
    'my_user',
    'my_password'
);

return $connection->getDBH();
{% endhighlight %}

Both the config file path and the migrations directory can be overridden as extra arguments (see [Command reference](#command-reference)). By default:

| What | Default |
|---|---|
| Config file | `./migrations-config.php` |
| Migrations directory | `./database/migrations` |

---

## Command reference

### `ud-migrate migrate [config-file] [migrations-dir]`

Runs every migration that hasn't run yet, in filename order. Each migration runs inside its own transaction; if one fails, its transaction is rolled back and the command stops.

{% highlight bash %}
ud-migrate migrate
# Migrated:  2026_07_19_180602_create_books_table
{% endhighlight %}

Running it again when nothing is pending is a no-op:

{% highlight bash %}
ud-migrate migrate
# Nothing to migrate.
{% endhighlight %}

### `ud-migrate rollback [config-file] [migrations-dir]`

Reverts every migration from the **most recent batch** (all the migrations applied together in the last `migrate` run), in reverse order.

{% highlight bash %}
ud-migrate rollback
# Rolled back: 2026_07_19_180602_create_books_table
{% endhighlight %}

### `ud-migrate status [config-file] [migrations-dir]`

Lists every migration found on disk and whether it has run:

{% highlight bash %}
ud-migrate status
# [X] 2026_07_19_180602_create_books_table
# [ ] 2026_07_19_183000_add_isbn_to_books
{% endhighlight %}

### `ud-migrate make <name> [migrations-dir]`

Scaffolds a new migration file from a stub, prefixed with the current timestamp.

---

## How it tracks progress

A `migrations` table is created automatically (on first `migrate`, `rollback`, or `status` call) to record which migrations have run and in which batch. You never need to create or touch this table yourself.

---

## Running migrations without the CLI

The `Migrator` class can be used directly from PHP — useful for running migrations as part of a deploy script:

{% highlight php %}
use Fabiom\UglyDuckling\Framework\DataBase\Migrations\Migrator;
use Fabiom\UglyDuckling\Framework\DataBase\Migrations\MigrationRepository;

$pdo = $dbConnection->getDBH();

$migrator = new Migrator($pdo, new MigrationRepository($pdo), __DIR__ . '/database/migrations');

$ran = $migrator->migrate(); // string[] of migration names that were run
{% endhighlight %}
