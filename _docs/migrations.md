---
layout: page
name: Migrations
---

# Migrations

UglyDuckling ships a small migration system to manage database schema changes over time, similar in spirit to Laravel's migrations. Each migration is a PHP file that returns a class with an `up()` and a `down()` method; a CLI tool (`ud-migrate`) tracks which migrations have run and applies the ones that haven't.

Migration files live in **your application**, not in the library — the library only provides the base class, the `Schema`/`Blueprint` builder, the runner, and the CLI.

---

## Migration skeleton

A migration file returns an anonymous class extending `Migration`. Both methods receive the raw PDO connection. Table changes are usually written through `Schema`/`Blueprint`, a fluent builder that compiles to the correct SQL for MySQL or SQLite depending on which one the connection is talking to:

{% highlight php %}
use Fabiom\UglyDuckling\Framework\DataBase\Migrations\Blueprint;
use Fabiom\UglyDuckling\Framework\DataBase\Migrations\Migration;
use Fabiom\UglyDuckling\Framework\DataBase\Migrations\Schema;

return new class extends Migration {

    public function up( PDO $pdo ): void {
        Schema::create( 'books', function ( Blueprint $table ) {
            $table->id();
            $table->string( 'title' );
            $table->string( 'author' );
            $table->timestamps();
        } );
    }

    public function down( PDO $pdo ): void {
        Schema::dropIfExists( 'books' );
    }

};
{% endhighlight %}

`$pdo` is still there for anything `Schema` doesn't cover — raw SQL, seeding data, `ALTER`s outside the Blueprint vocabulary — and is used exactly as before:

{% highlight php %}
public function up( PDO $pdo ): void {
    $pdo->exec("UPDATE books SET status = 'draft' WHERE status IS NULL");
}
{% endhighlight %}

---

## Schema & Blueprint

`Schema` is a static facade for building/altering/dropping tables. `ud-migrate` wires it to the right connection automatically before running a migration's `up()`/`down()` — you never call `Schema::setConnection()` yourself in a migration file.

### Creating a table

{% highlight php %}
Schema::create( 'books', function ( Blueprint $table ) {
    $table->id();                              // unsigned bigint, auto-increment, primary key
    $table->string( 'title' );                 // VARCHAR(255)
    $table->string( 'isbn', 20 )->nullable();   // VARCHAR(20), nullable
    $table->text( 'description' )->nullable();
    $table->boolean( 'published' )->default( false );
    $table->decimal( 'price', 8, 2 )->nullable();
    $table->foreignId( 'author_id' )->constrained( 'authors' );
    $table->timestamps();                       // nullable created_at / updated_at datetime columns
} );
{% endhighlight %}

Available column types:

| Method | SQL |
|---|---|
| `id($name = 'id')` | unsigned `BIGINT` auto-increment primary key |
| `increments($name)` | `INTEGER` auto-increment primary key |
| `string($name, $length = 255)` | `VARCHAR($length)` |
| `text($name)` | `TEXT` |
| `integer($name)` | `INTEGER` |
| `bigInteger($name)` | `BIGINT` |
| `unsignedBigInteger($name)` | unsigned `BIGINT` |
| `foreignId($name)` | unsigned `BIGINT` (alias for `unsignedBigInteger`) |
| `boolean($name)` | `BOOLEAN` |
| `decimal($name, $precision = 8, $scale = 2)` | `DECIMAL($precision, $scale)` |
| `date($name)` | `DATE` |
| `dateTime($name)` | `DATETIME` |
| `timestamp($name)` | `TIMESTAMP` |
| `uuid($name)` | `CHAR(36)` |
| `foreignUuid($name)` | `CHAR(36)` (alias for `uuid`, for referencing a `uuid()` primary key) |
| `timestamps()` | adds nullable `created_at` / `updated_at` `dateTime` columns |

Every column method except `timestamps()` returns a `ColumnDefinition` you can chain modifiers onto:

| Modifier | Effect |
|---|---|
| `->nullable()` | allows `NULL` (columns are `NOT NULL` by default) |
| `->default($value)` | sets a default value |
| `->unsigned()` | marks a numeric column unsigned |
| `->unique()` | adds a single-column unique constraint |
| `->primary()` | marks this the primary key, without auto-increment |
| `->constrained($table, $column = 'id')` | adds a `FOREIGN KEY` referencing `$table.$column` |
| `->references($column)->on($table)` | same as `constrained()`, spelled out in two steps |
| `->onDelete($action)` / `->onUpdate($action)` | `cascade` \| `restrict` \| `set_null` \| `no_action` |

### UUID primary/foreign keys

`id()`/`increments()` are always auto-increment. For a UUID primary key, use `uuid()` with the plain `->primary()` modifier instead, and `foreignUuid()` (not `foreignId()`) on the referencing side so the column types match:

{% highlight php %}
Schema::create( 'authors', function ( Blueprint $table ) {
    $table->uuid( 'id' )->primary();
    $table->string( 'name' );
} );

Schema::create( 'books', function ( Blueprint $table ) {
    $table->id();
    $table->foreignUuid( 'author_id' )->constrained( 'authors' );
} );
{% endhighlight %}

`Schema`/`Blueprint` only declare the column - they don't generate UUID values. Neither MySQL nor SQLite has a portable UUID function, so generate the value in PHP before inserting (the same way Laravel's `HasUuids` trait does it), e.g. with `random_bytes()`-based UUIDv4 generation in your own code.

`constrained()`/`references()`/`on()` always take an explicit table name — there's no pluralization guesswork (`author_id` does not automatically imply an `authors` table), to avoid a constraint silently pointing at the wrong table.

{% highlight php %}
$table->foreignId( 'author_id' )
    ->constrained( 'authors' )
    ->onDelete( 'cascade' );
{% endhighlight %}

MySQL and SQLite compile this differently under the hood (MySQL needs a table-level `FOREIGN KEY` clause; SQLite enforces an inline `REFERENCES` clause and requires `PRAGMA foreign_keys = ON`, which `Schema` enables automatically on SQLite connections) — from a migration's point of view the API is identical either way.

### Altering an existing table

{% highlight php %}
Schema::table( 'books', function ( Blueprint $table ) {
    $table->string( 'isbn', 20 )->nullable();
} );
{% endhighlight %}

Dropping columns uses the same closure:

{% highlight php %}
Schema::table( 'books', function ( Blueprint $table ) {
    $table->dropColumn( 'isbn' );
} );
{% endhighlight %}

Foreign keys work the same way when added later, not just at `create()` time:

{% highlight php %}
Schema::table( 'books', function ( Blueprint $table ) {
    $table->foreignId( 'author_id' )->nullable()->constrained( 'authors' );
} );
{% endhighlight %}

### Indexes

{% highlight php %}
Schema::create( 'tags', function ( Blueprint $table ) {
    $table->id();
    $table->string( 'slug' );
    $table->unique( 'slug' );        // CREATE UNIQUE INDEX
    $table->index( [ 'slug' ] );     // CREATE INDEX
} );
{% endhighlight %}

### Dropping tables

{% highlight php %}
Schema::drop( 'books' );            // errors if the table doesn't exist
Schema::dropIfExists( 'books' );    // no-op if it doesn't
{% endhighlight %}

### Checking table existence

{% highlight php %}
if ( ! Schema::hasTable( 'books' ) ) {
    // ...
}
{% endhighlight %}

---

## Creating a migration

Use `ud-migrate make` to scaffold a new, timestamp-prefixed migration file. The name determines both the filename and, when it follows a recognized pattern, the stub content:

{% highlight bash %}
ud-migrate make create_books_table
# Created migration: database/migrations/2026_07_19_180602_create_books_table.php
# -> pre-filled with Schema::create('books', ...) / Schema::dropIfExists('books')

ud-migrate make add_isbn_to_books_table
# -> pre-filled with Schema::table('books', ...) in both up() and down()

ud-migrate make remove_isbn_from_books_table
# -> same as add_..._to_..._table: Schema::table('books', ...)

ud-migrate make backfill_book_slugs
# -> name doesn't match a known pattern, falls back to the generic stub
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

### `ud-migrate rollback [--step=N] [config-file] [migrations-dir]`

With no `--step`, reverts every migration from the **most recent batch** (all the migrations applied together in the last `migrate` run), in reverse order:

{% highlight bash %}
ud-migrate rollback
# Rolled back: 2026_07_19_180602_create_books_table
{% endhighlight %}

With `--step=N`, reverts the last `N` migrations instead, regardless of which batch they ran in:

{% highlight bash %}
ud-migrate rollback --step=2
# Rolled back: 2026_07_19_183000_add_isbn_to_books_table
# Rolled back: 2026_07_19_180602_create_books_table
{% endhighlight %}

### `ud-migrate refresh [config-file] [migrations-dir]`

Reverts every migration that has ever run (via `down()`, most recent first), then migrates again from scratch. Useful when a migration's `down()` also needs to run — e.g. to drop indexes or columns cleanly — before rebuilding.

{% highlight bash %}
ud-migrate refresh
# Database refreshed.
# Migrated:  2026_07_19_180602_create_books_table
{% endhighlight %}

### `ud-migrate fresh [config-file] [migrations-dir]`

Drops every table in the database directly (including the `migrations` tracking table itself), without running any `down()`, then migrates from scratch. Faster than `refresh`, but skips `down()` entirely — use it for local/CI databases you're happy to wipe, not as a substitute for correct `down()` methods.

{% highlight bash %}
ud-migrate fresh
# Database dropped and migrated fresh.
# Migrated:  2026_07_19_180602_create_books_table
{% endhighlight %}

### `ud-migrate status [config-file] [migrations-dir]`

Lists every migration found on disk and whether it has run:

{% highlight bash %}
ud-migrate status
# [X] 2026_07_19_180602_create_books_table
# [ ] 2026_07_19_183000_add_isbn_to_books_table
{% endhighlight %}

### `ud-migrate make <name> [migrations-dir]`

Scaffolds a new migration file from a stub, prefixed with the current timestamp (see [Creating a migration](#creating-a-migration) for how the name affects the stub).

---

## How it tracks progress

A `migrations` table is created automatically (on first `migrate`, `rollback`, `refresh`, `fresh`, or `status` call) to record which migrations have run and in which batch. You never need to create or touch this table yourself.

---

## Running migrations without the CLI

The `Migrator` class can be used directly from PHP — useful for running migrations as part of a deploy script:

{% highlight php %}
use Fabiom\UglyDuckling\Framework\DataBase\Migrations\Migrator;
use Fabiom\UglyDuckling\Framework\DataBase\Migrations\MigrationRepository;

$pdo = $dbConnection->getDBH();

$migrator = new Migrator($pdo, new MigrationRepository($pdo), __DIR__ . '/database/migrations');

$ran = $migrator->migrate();          // string[] of migration names that were run
$rolledBack = $migrator->rollback();  // last batch; pass an int for --step behavior
$migrator->refresh();
$migrator->fresh();
{% endhighlight %}

Constructing a `Migrator` also points the `Schema` facade at that connection, so migration files calling `Schema::create()`/`Schema::table()` work the same way whether they're run through the CLI or through `Migrator` directly.

---

## Loading data after migrating

Once a table exists, use [Seeders](seeders) — `ud-migrate seed` — to load rows into it.
