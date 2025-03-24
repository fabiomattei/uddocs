---
layout: page
name: DAO
---

Data Access Object are what UD uses in order to query a database.

It is a good idea to create a DAO class for each table we define in the database, 
a DAO class should extend [BasicDao](https://github.com/fabiomattei/uglyduckling/blob/master/src/Framework/DataBase/BasicDao.php)
that define the basic features needed in order to query the data.

### Starting to define a DAO

This is the bare minimum we need to define a DAO class. 

{% highlight php %}
class BookDao extends BasicDao {

    const DB_TABLE = 'books';
    const DB_TABLE_PK = 'bk_id';
    const DB_TABLE_UPDATED_FIELD_NAME = 'bk_updated';
    const DB_TABLE_CREATED_FLIED_NAME = 'bk_created';

    /*
    Field list
    bk_id  	                     Primary key
	bk_title
    bk_author
	bk_updated
	bk_created
    */

    /**
     * it overloads the getEmpty method of the parent class
     */
    public function getEmpty() {
        $empty = new \stdClass;
        $empty->bk_id      = 0;
        $empty->bk_title   = '';
        $empty->bk_author  = '';
        $empty->cr_updated = '';
        $empty->cr_created = '';
        return $empty;
    }
}
{% endhighlight %}

#### Basic setup

We are defining a new BookDao class and we are setting a few constants:

* **DB_TABLE = 'books';** the name of the dable in the database
* **DB_TABLE_PK = 'bk_id';** the name of the primary key
* **DB_TABLE_UPDATED_FIELD_NAME = 'bk_updated';** this fields get automatically updated every time his row is updated
* **DB_TABLE_CREATED_FLIED_NAME = 'bk_created';** this field is initiated with time when a row is created

#### Empty object

Think it as a null object. This is the object returned in case there is no row to return in the database.
This helps to keep the colde much cleaner in the database.

### Creating an instance

In order to be able to use a DAO we need to create an instance. We usullay do this in a **getRequest** or in a 
**postRequest** in a <a href="{{site.baseurl}}/docs/controller">controller</a>.

{% highlight php %}
$bookDao = new BookDao();
$bookDao->setDBH($this->dbconnection->getDBH());
$bookDao->setLogger($this->logger);
{% endhighlight %}


### main methods

#### getAll
Return all rows contained in a table. It return those as list of stdClass following the PDO::FETCH_OBJ directive

{% highlight php %}
$bookDao = new BookDao();
$bookDao->setDBH($this->dbconnection->getDBH());
$bookDao->setLogger($this->logger);
$mybooks = $bookDao->getAll();
{% endhighlight %}

#### getById($id)
It Get the row with the selected id
if no corresponding row is found it gives the empty object
calling the getEmpty method (null object).

{% highlight php %}
$bookDao = new BookDao();
$bookDao->setDBH($this->dbconnection->getDBH());
$bookDao->setLogger($this->logger);
$mybooks = $bookDao->getById(5);
{% endhighlight %}

#### insert($fields, $debug = false)
Insert a row in the database.
Set the updated and created fields to current date and time
It accpts an array containing as key the field name and as value
the field content.

@param $fields :: array of fields to insert

EX. [ 'field1' => 'content field 1', 'field2', 'content field 2' ];

{% highlight php %}
$bookDao = new BookDao();
$bookDao->setDBH($this->dbconnection->getDBH());
$bookDao->setLogger($this->logger);
$myNewId = $bookDao->insert([ 'title' => 'The Tragedy of Macbeth', 'author', 'William Shakespeare' ]);
{% endhighlight %}

#### insertWithUUID($fields, $debug = false)
Insert a row in the database and automaticalli creates a UUID for the table primary key.
Set the updated and created fields to current date and time.
It accepts an array containing as key the field name and as value
the field content.

@param $fields :: array of fields to insert

EX. [ 'field1' => 'content field 1', 'field2', 'content field 2' ];

{% highlight php %}
$bookDao = new BookDao();
$bookDao->setDBH($this->dbconnection->getDBH());
$bookDao->setLogger($this->logger);
$myNewUUID = $bookDao->insertWithUUID([ 'title' => 'The Tragedy of Macbeth', 'author', 'William Shakespeare' ]);
{% endhighlight %}

#### update($id, $fields, $debug = false)

This function updates a single row of the delared table.
It uptades the row haveing id = $id

@param $id :: integer id

@param $fields :: array of fields to update

Ex. array( 'field1' => 'value1', 'field2' => 'value2' )

{% highlight php %}
$bookDao = new BookDao();
$bookDao->setDBH($this->dbconnection->getDBH());
$bookDao->setLogger($this->logger);
$bookDao->update(5, [ 'title' => 'The Tragedy of Macbeth', 'author', 'William Shakespeare' ]);
{% endhighlight %}

#### updateNoDate($id, $fields, $debug = false)

This function updates a single row of the delared table.
It uptades the row haveing id = $id

@param $id :: integer id

@param $fields :: array of fields to update

Ex. array( 'field1' => 'value1', 'field2' => 'value2' )

{% highlight php %}
$bookDao = new BookDao();
$bookDao->setDBH($this->dbconnection->getDBH());
$bookDao->setLogger($this->logger);
$bookDao->updateNoDate(5, [ 'title' => 'The Tragedy of Macbeth', 'author', 'William Shakespeare' ]);
{% endhighlight %}

#### updateByFields($conditionsfields, $fields, $debug = false)

This method allow to update many rows of a single table at the same time

@param $conditionsfields :: array of fields to put in where clause
$tododao->getByFields( [ 'open' => '0' ] );
this will get all the row having the field open = 0

you can set more then a search parameter (evaluated in AND)

$tododao->getByFields( array( 'open' => '0', 'handling' => '1' ) );

@param $fields :: array of fields to update

Ex. array( 'field1' => 'value1', 'field2' => 'value2' )

{% highlight php %}
$bookDao = new BookDao();
$bookDao->setDBH($this->dbconnection->getDBH());
$bookDao->setLogger($this->logger);
$bookDao->updateByFields(['scaffolding' => 5, 'category' => 'literature'], [ 'title' => 'The Tragedy of Macbeth', 'author', 'William Shakespeare' ]);
{% endhighlight %}

#### updateByFieldsNoDate($conditionsfields, $fields, $debug = false)

This method allow to update many rows of a single table at the same time

@param $conditionsfields :: array of fields to put in where clause

$tododao->getByFields( array( 'open' => '0' ) );

this will get all the row having the field open = 0

you can set more then a search parameter (evaluated in AND)

$tododao->getByFields( array( 'open' => '0', 'handling' => '1' ) );

@param $fields :: array of fields to update

Ex.

{% highlight php %}
$bookDao = new BookDao();
$bookDao->setDBH($this->dbconnection->getDBH());
$bookDao->setLogger($this->logger);
$bookDao->updateByFieldsNoDate(['scaffolding' => 5, 'category' => 'literature'], [ 'title' => 'The Tragedy of Macbeth', 'author', 'William Shakespeare' ]);
{% endhighlight %}

#### delete( $id )

This is the basic function for one row from a table specifying the primary key
of the row you want to delete.
Once you created a instance of the DAO object you can do for example:

$tododao->delete( 15 );

this will delete the row having the primary key set to 15.

Remeber that you need to set the primary key in the tabledao.php file in a costant named DB_TABLE_PK

Example:

{% highlight php %}
$bookDao = new BookDao();
$bookDao->setDBH($this->dbconnection->getDBH());
$bookDao->setLogger($this->logger);
$bookDao->delete( 5 );
{% endhighlight %}

#### deleteByFields( $fields )

This function deletes a set of row from a table depending from the
parameters you set when calling it.

$tododao->delete( [ 'open' => '0', 'handling' => '1' ] );

this will delete the row having the field open set to 0 and the field handling set to 1.

Remeber that you need to set the table name in the tabledao.php file in a costant named DB_TABLE

Example:

{% highlight php %}
$bookDao = new BookDao();
$bookDao->setDBH($this->dbconnection->getDBH());
$bookDao->setLogger($this->logger);
$bookDao->deleteByFields( ['open' => '0', 'handling' => '1'] );
{% endhighlight %}

#### getByFields($conditionsfields, $orderby = 'none', $requestedfields = 'none')

This is the basic function for getting a set of elements from a table.
Once you created a instance of the DAO object you can do for example:

$tododao->getByFields( array( 'open' => '0' ) );

this will get all the row having the field open = 0

you can set more then a search parameter (evaluated in AND)

$tododao->getByFields( array( 'open' => '0', 'handling' => '1' ) );

you can even specify how to order the rows you requested

$tododao->getByFields( array( 'id' => '42' ), array('name', 'description') );

you can even request few specific fields and not the whole table fields

$tododao->getByFields( array( 'id' => '42' ), array('name', 'description'), array('id', 'name', 'description') );


{% highlight php %}
$bookDao = new BookDao();
$bookDao->setDBH($this->dbconnection->getDBH());
$bookDao->setLogger($this->logger);
$myBooks = $bookDao->getByFields( ['open' => '0', 'handling' => '1'] );
{% endhighlight %}

#### getBySQLQuery($sqlQuery, $fields, $debug = false)

This is the basic function for running a SQL query.
Once you created a instance of the DAO object you can do for example:

$sqlQuery string containing the query

$tododao->getBySQLQuery( 'SELECT * FROM mytable WHERE myfield = :myfieldcontent;', [ ':myfieldcontent' => '0' ] );

this will get all the row having the field myfield = 0

{% highlight php %}
$bookDao = new BookDao();
$bookDao->setDBH($this->dbconnection->getDBH());
$bookDao->setLogger($this->logger);
$myBooks = $bookDao->getBySQLQuery( "SELECT * FROM books WHERE id=:id;", [':id'=>5] );
{% endhighlight %}

#### getByFieldList($fieldname, $ids, $conditionsfields, $orderby = 'none', $requestedfields = 'none')

This function allows user to get a set of elements from a table.

@param $fieldname                name of field that needs to be confronted with the array of ids

@param $ids                      array of ids

@param $conditionsfields

@param string $orderby

@param string $requestedfields

@return array|PDOStatement

#### getArrayByFieldList($fieldname, $ids, $conditionsfields, $orderby = 'none', $requestedfields = 'none')

This function allows user to get a set of elements from a table.

@param  $fieldname                name of field that needs to be confronted with the array of ids

@param  $ids                      array of ids

@param  $conditionsfields

@param  string $orderby

@param  string $requestedfields

@return array|PDOStatement

@throws\Exception


#### getOneByFields($conditionsfields, $requestedfields = 'none')

This is the basic function for getting one element from a table.
Once you created a instance of the DAO object you can do for example:

$tododao->getOneByFields( array( 'id' => '42' ) );

this will get the field having id = 42

you can set more then a search parameter (evaluated in AND)

$tododao->getOneByFields( array( 'id' => '42', 'open' => '1' ) );

you can even request few specific fields and not the whole table fields

$tododao->getOneByFields( array( 'id' => '42' ), array('id', 'name', 'description') );

{% highlight php %}
$bookDao = new BookDao();
$bookDao->setDBH($this->dbconnection->getDBH());
$bookDao->setLogger($this->logger);
$myBook = $bookDao->getOneByFields( [ 'id' => '42' ], ['id', 'name', 'description' ] );
{% endhighlight %}

#### getArrayByFields($conditionsfields, $orderby = 'none', $requestedfields = 'none')

This is the basic function for getting an array of elements from a table.
The returned array will have the entity id as index
Once you created a instance of the DAO object you can do for example:

$tododao->getArrayByFields( array( 'open' => '0' ) );

this will get all the row having the field open = 0

you can set more then a search parameter (evaluated in AND)

$tododao->getArrayByFields( array( 'open' => '0', 'handling' => '1' ) );

you can even specify how to order the rows you requested

$tododao->getArrayByFields( array( 'id' => '42' ), array('name', 'description') );

you can even request few specific fields and not the whole table fields

$tododao->getArrayByFields( array( 'id' => '42' ), array('name', 'description'), array('id', 'name', 'description') );

{% highlight php %}
$bookDao = new BookDao();
$bookDao->setDBH($this->dbconnection->getDBH());
$bookDao->setLogger($this->logger);
$myCount = $bookDao->getArrayByFields( [ 'id' => '42' ], [ 'name', 'description' ], ['id', 'name', 'description' ] );
{% endhighlight %}
    
#### countByFields( $conditionsfields )

his method allow to count the number of rows, contained in a table, that
espect given conditions.

nce you created a instance of the DAO object you can do for example:

tododao->getByFields( array( 'open' => '0' ) );

his will get all the row having the field open = 0

ou can set more then a search parameter (evaluated in AND)

tododao->getByFields( [ 'open' => '0', 'handling' => '1' ] );

{% highlight php %}
$bookDao = new BookDao();
$bookDao->setDBH($this->dbconnection->getDBH());
$bookDao->setLogger($this->logger);
$myCount = $bookDao->countByFields( [ 'open' => '0', 'handling' => '1' ] );
{% endhighlight %}
    
#### countByFieldList($fieldname, $ids, $conditionsfields, $orderby = 'none', $requestedfields = 'none')

This method allow to count the number of rows, contained in a table, that
respect given conditions.

@param $fieldname                name of field that needs to be confronted with the array of ids

@param $ids                      array of ids

@param $conditionsfields

@param string $orderby

@param string $requestedfields

@return array|PDOStatement

@throws\Exception

{% highlight php %}
$bookDao = new BookDao();
$bookDao->setDBH($this->dbconnection->getDBH());
$bookDao->setLogger($this->logger);
$myCount = $bookDao->countByFieldList( 'id', [4, 6, 7], [ 'open' => '0', 'handling' => '1' ] );
{% endhighlight %}
 
#### getOneField($fieldname, $conditionsfields)

This method get just one filed from a table

@param $fieldname                name of field to get

@param $conditionsfields         conditions evaluated in AND

@return the field content

@throws\Exception

{% highlight php %}
$bookDao = new BookDao();
$bookDao->setDBH($this->dbconnection->getDBH());
$bookDao->setLogger($this->logger);
$mybook = $bookDao->getOneField( 'title', [ 'open' => '0', 'handling' => '1' ] );
{% endhighlight %}

