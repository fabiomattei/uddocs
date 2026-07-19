---
layout: page
name: DAO
---

# DAO

A Data Access Object (DAO) is the layer UglyDuckling uses to query the database. Create one DAO class per database table. Each DAO extends `BasicDao`, which provides all standard CRUD methods and query helpers through PDO prepared statements.

---

## DAO skeleton

{% highlight php %}
use Fabiom\UglyDuckling\Framework\DataBase\BasicDao;

class BookDao extends BasicDao {

    const DB_TABLE                  = 'books';
    const DB_TABLE_PK               = 'bk_id';
    const DB_TABLE_UPDATED_FIELD_NAME = 'bk_updated';
    const DB_TABLE_CREATED_FLIED_NAME = 'bk_created';

    public function getEmpty() {
        $empty            = new \stdClass;
        $empty->bk_id     = 0;
        $empty->bk_title  = '';
        $empty->bk_author = '';
        $empty->bk_updated = '';
        $empty->bk_created = '';
        return $empty;
    }
}
{% endhighlight %}

---

## Constants

| Constant | Description |
|---|---|
| `DB_TABLE` | Name of the database table. |
| `DB_TABLE_PK` | Name of the primary key column. Used by `getById`, `delete`, `update`, and count queries. |
| `DB_TABLE_UPDATED_FIELD_NAME` | Column automatically set to the current timestamp on every `insert` and `update`. |
| `DB_TABLE_CREATED_FLIED_NAME` | Column set to the current timestamp only on `insert`. |

---

## Empty object

Override `getEmpty()` to return a blank `stdClass` with all fields initialised to safe defaults. `getById()` and `getOneByFields()` return this object instead of `null` when no row is found, so the rest of your code can always dereference properties without null checks.

{% highlight php %}
public function getEmpty() {
    $empty            = new \stdClass;
    $empty->bk_id     = 0;
    $empty->bk_title  = '';
    $empty->bk_author = '';
    return $empty;
}
{% endhighlight %}

---

## Creating an instance

Instantiate a DAO inside a controller's `getRequest()` or `postRequest()`, inject the PDO connection, and optionally a logger:

{% highlight php %}
$bookDao = new BookDao();
$bookDao->setDBH($this->dbconnection->getDBH());
$bookDao->setLogger($this->logger);
{% endhighlight %}

---

## Read methods

### getAll()

Returns all rows in the table as a PDO statement (iterable with `PDO::FETCH_OBJ`).

{% highlight php %}
$books = $bookDao->getAll();
foreach ($books as $book) {
    echo $book->bk_title;
}
{% endhighlight %}

### getById($id)

Returns the row with the given primary key. Returns `getEmpty()` when no row is found.

{% highlight php %}
$book = $bookDao->getById(5);
echo $book->bk_title;
{% endhighlight %}

### getByFields($conditionsfields, $orderby = 'none', $requestedfields = 'none')

Returns a PDO statement for all rows matching the given conditions. All conditions are combined with `AND`.

{% highlight php %}
// All published books
$books = $bookDao->getByFields(['bk_published' => 1]);

// Published books ordered by title
$books = $bookDao->getByFields(
    ['bk_published' => 1],
    ['bk_title']
);

// Specific columns only
$books = $bookDao->getByFields(
    ['bk_published' => 1],
    ['bk_title'],
    ['bk_id', 'bk_title', 'bk_author']
);
{% endhighlight %}

### getOneByFields($conditionsfields, $requestedfields = 'none')

Returns a single row as a `stdClass`. Returns `getEmpty()` when no row is found.

{% highlight php %}
$book = $bookDao->getOneByFields(['bk_id' => 42]);

// Specific columns only
$book = $bookDao->getOneByFields(
    ['bk_id' => 42],
    ['bk_id', 'bk_title', 'bk_author']
);
{% endhighlight %}

### getArrayByFields($conditionsfields, $orderby = 'none', $requestedfields = 'none')

Same as `getByFields()` but returns an associative array keyed by primary key instead of a PDO statement. Convenient when you need random access by ID.

{% highlight php %}
$booksById = $bookDao->getArrayByFields(['bk_published' => 1]);
$book = $booksById[42];
{% endhighlight %}

### getByFieldList($fieldname, $ids, $conditionsfields, $orderby = 'none', $requestedfields = 'none')

Returns a PDO statement for rows where `$fieldname` is in the `$ids` array, filtered further by `$conditionsfields`.

{% highlight php %}
// Books whose author_id is one of [3, 7, 12], and that are published
$books = $bookDao->getByFieldList(
    'bk_author_id',
    [3, 7, 12],
    ['bk_published' => 1]
);
{% endhighlight %}

### getArrayByFieldList($fieldname, $ids, $conditionsfields, $orderby = 'none', $requestedfields = 'none')

Same as `getByFieldList()` but returns an associative array keyed by primary key.

{% highlight php %}
$booksById = $bookDao->getArrayByFieldList(
    'bk_author_id',
    [3, 7, 12],
    ['bk_published' => 1]
);
{% endhighlight %}

### getOneField($fieldname, $conditionsfields)

Returns the value of a single column from the first matching row. Returns an empty string when nothing is found.

{% highlight php %}
$title = $bookDao->getOneField('bk_title', ['bk_id' => 42]);
{% endhighlight %}

### getBySQLQuery($sqlQuery, $fields, $debug = false)

Runs an arbitrary parameterised SQL query and returns a PDO statement. Use this only when the standard methods are not expressive enough.

{% highlight php %}
$books = $bookDao->getBySQLQuery(
    'SELECT * FROM books WHERE bk_author_id = :author_id AND bk_year > :year',
    [':author_id' => 3, ':year' => 2000]
);
{% endhighlight %}

---

## Joins with QueryBuilder

`BasicDao`'s field-based methods only query the DAO's own `DB_TABLE`. For queries that need to join other tables, call `newQuery()` to get a `QueryBuilder`: a fluent builder returning `stdClass` rows (`PDO::FETCH_OBJ`), same as the rest of the DAO.

{% highlight php %}
$rows = $bookDao->newQuery()
    ->select('books.bk_title', 'authors.au_name AS author_name')
    ->join('authors', 'books.bk_author_id = authors.au_id')
    ->where('books.bk_published', '=', 1)
    ->where('authors.au_country', '=', 'UK')
    ->orderBy('books.bk_title')
    ->limit(50)
    ->get();

foreach ($rows as $row) {
    echo $row->author_name;
}
{% endhighlight %}

`where()` values are always parameter-bound, even when the same column name is reused across joined tables. Table names, `select()`/`orderBy()` fields, and join conditions are not parameterisable by PDO, so treat them as trusted SQL you write yourself, not as a place to interpolate user input.

| Method | Description |
|---|---|
| `select(...$fields)` | Columns to return. Defaults to `*`. |
| `join($table, $onCondition, $type = 'INNER')` / `leftJoin($table, $onCondition)` | Adds a join clause. |
| `where($field, $operator, $value)` | Adds a bound `AND` condition. |
| `whereRaw($rawSql, $bindings = [])` | Escape hatch for conditions `where()` can't express, e.g. `BETWEEN`. Caller supplies and binds its own placeholders. |
| `orderBy(...$fields)` | Sets `ORDER BY`. |
| `limit($count)` | Sets `LIMIT`. |
| `get()` | Runs the query, returns a `PDOStatement` (`FETCH_OBJ`). |
| `first()` | Returns the first matching row as `stdClass`, or `null`. |
| `count()` | Returns the number of matching rows, respecting joins and `where()`. |

For one-off queries too irregular even for `QueryBuilder`, fall back to `getBySQLQuery()` above.

---

## Write methods

All write methods accept a `$debug = false` parameter. When `true`, the method echoes the constructed SQL and parameter dump to the page — useful during development.

### insert($fields, $debug = false)

Inserts a row and returns the new auto-increment ID. Sets `DB_TABLE_UPDATED_FIELD_NAME` and `DB_TABLE_CREATED_FLIED_NAME` to the current timestamp automatically.

{% highlight php %}
$newId = $bookDao->insert([
    'bk_title'  => 'The Tragedy of Macbeth',
    'bk_author' => 'William Shakespeare',
]);
{% endhighlight %}

### insertWithUUID($fields, $debug = false)

Inserts a row using a MySQL-generated UUID as the primary key. Returns the UUID string. Sets the timestamp fields automatically.

{% highlight php %}
$uuid = $bookDao->insertWithUUID([
    'bk_title'  => 'Hamlet',
    'bk_author' => 'William Shakespeare',
]);
{% endhighlight %}

### update($id, $fields, $debug = false)

Updates the row identified by `$id`. Sets `DB_TABLE_UPDATED_FIELD_NAME` to the current timestamp.

{% highlight php %}
$bookDao->update(5, [
    'bk_title'  => 'The Tragedy of Macbeth',
    'bk_author' => 'William Shakespeare',
]);
{% endhighlight %}

### updateNoDate($id, $fields, $debug = false)

Updates the row identified by `$id` without touching the timestamp column. Use when you need to update a row but do not want to change the modification date.

{% highlight php %}
$bookDao->updateNoDate(5, ['bk_published' => 0]);
{% endhighlight %}

### updateByFields($conditionsfields, $fields, $debug = false)

Updates all rows matching `$conditionsfields`. Sets `DB_TABLE_UPDATED_FIELD_NAME` to the current timestamp.

{% highlight php %}
// Mark all books by a given author as published
$bookDao->updateByFields(
    ['bk_author' => 'William Shakespeare'],
    ['bk_published' => 1]
);
{% endhighlight %}

### updateByFieldsNoDate($conditionsfields, $fields, $debug = false)

Same as `updateByFields()` but does not update the timestamp column.

{% highlight php %}
$bookDao->updateByFieldsNoDate(
    ['bk_author' => 'William Shakespeare'],
    ['bk_featured' => 0]
);
{% endhighlight %}

### delete($id)

Deletes the row with the given primary key.

{% highlight php %}
$bookDao->delete(5);
{% endhighlight %}

### deleteByFields($fields)

Deletes all rows matching the given conditions. All conditions are combined with `AND`.

{% highlight php %}
$bookDao->deleteByFields(['bk_published' => 0, 'bk_author' => 'Anonymous']);
{% endhighlight %}

---

## Count methods

### countByFields($conditionsfields)

Returns the number of rows matching the given conditions.

{% highlight php %}
$count = $bookDao->countByFields(['bk_published' => 1]);
{% endhighlight %}

### countByFieldList($fieldname, $ids, $conditionsfields)

Counts rows where `$fieldname` is in the `$ids` array, filtered by `$conditionsfields`. Returns an associative array keyed by each value in `$ids`, so you can look up the count per ID in O(1).

{% highlight php %}
// Count published books per author for authors 3, 7, and 12
$counts = $bookDao->countByFieldList(
    'bk_author_id',
    [3, 7, 12],
    ['bk_published' => 1]
);
// $counts[3] === 4, $counts[7] === 1, etc.
{% endhighlight %}

---

## Extending BasicDao

Add custom query methods to your DAO class whenever the standard methods are not sufficient. Keep domain-specific SQL inside the DAO so controllers stay free of raw queries.

{% highlight php %}
class BookDao extends BasicDao {

    const DB_TABLE                    = 'books';
    const DB_TABLE_PK                 = 'bk_id';
    const DB_TABLE_UPDATED_FIELD_NAME = 'bk_updated';
    const DB_TABLE_CREATED_FLIED_NAME = 'bk_created';

    public function getEmpty() {
        $empty            = new \stdClass;
        $empty->bk_id     = 0;
        $empty->bk_title  = '';
        $empty->bk_author = '';
        return $empty;
    }

    /**
     * Returns [['id' => ..., 'label' => ...], ...] suitable for a dropdown.
     */
    public function makeListForDropdown(): array {
        $out = [];
        $rows = $this->getByFields(['bk_published' => 1], ['bk_title']);
        foreach ($rows as $row) {
            $out[] = ['id' => $row->bk_id, 'label' => $row->bk_title];
        }
        return $out;
    }

    public function getRecentByAuthor(int $authorId, int $limit): array {
        return $this->getBySQLQuery(
            'SELECT * FROM books WHERE bk_author_id = :author ORDER BY bk_created DESC LIMIT :lim',
            [':author' => $authorId, ':lim' => $limit]
        );
    }
}
{% endhighlight %}
