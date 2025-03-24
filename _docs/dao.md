---
layout: page
name: Controller
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

#### getById($id)
It Get the row with the selected id
if no corresponding row is found it gives the empty object
calling the getEmpty method (null object).

#### insert($fields, $debug = false)
Insert a row in the database.
Set the updated and created fields to current date and time
It accpts an array containing as key the field name and as value
the field content.

@param $fields :: array of fields to insert

EX. [ 'field1' => 'content field 1', 'field2', 'content field 2' ];

#### insertWithUUID($fields, $debug = false)
Insert a row in the database and automaticalli calls UUID for the table primary key.
Set the updated and created fields to current date and time
It accepts an array containing as key the field name and as value
the field content.

@param $fields :: array of fields to insert
EX. [ 'field1' => 'content field 1', 'field2', 'content field 2' ];

#### update($id, $fields, $debug = false)

This function updates a single row of the delared table.
It uptades the row haveing id = $id
@param $id :: integer id
@param $fields :: array of fields to update
Ex. array( 'field1' => 'value1', 'field2' => 'value2' )

#### updateNoDate($id, $fields, $debug = false)

This function updates a single row of the delared table.
It uptades the row haveing id = $id
@param $id :: integer id
@param $fields :: array of fields to update
Ex. array( 'field1' => 'value1', 'field2' => 'value2' )

#### updateByFields($conditionsfields, $fields, $debug = false)

This method allow to update many rows of a single table at the same time

@param $conditionsfields :: array of fields to put in where clause
$tododao->getByFields( array( 'open' => '0' ) );
this will get all the row having the field open = 0

you can set more then a search parameter (evaluated in AND)
$tododao->getByFields( array( 'open' => '0', 'handling' => '1' ) );

@param $fields :: array of fields to update
Ex. array( 'field1' => 'value1', 'field2' => 'value2' )


    /**
     * This method allow to update many rows of a single table at the same time
     *
     * @param $conditionsfields :: array of fields to put in where clause
     * $tododao->getByFields( array( 'open' => '0' ) );
     * this will get all the row having the field open = 0
     *
     * you can set more then a search parameter (evaluated in AND)
     * $tododao->getByFields( array( 'open' => '0', 'handling' => '1' ) );
     *
     * @param $fields :: array of fields to update
     * Ex. array( 'field1' => 'value1', 'field2' => 'value2' )
     */
    function updateByFieldsNoDate($conditionsfields, $fields, $debug = false) {
        $conditionslist = $this->organizeConditionsFields($conditionsfields);
        $presentmoment = date('Y-m-d H:i:s', time());
        $filedslist = '';
        foreach ($fields as $key => $value) {
            $filedslist .= $key . ' = :' . $key . ', ';
        }
        $filedslist = substr($filedslist, 0, -2);
        try {
            $query = 'UPDATE ' . $this::DB_TABLE . ' SET ' . $filedslist . ' ';
            if ($conditionslist != '') {
                $query .= 'WHERE ' . $conditionslist . '; ';
            }

            $STH = $this->DBH->prepare($query);
            foreach ($fields as $key => &$value) {
                $STH->bindParam($key, $value);
            }
            foreach ($conditionsfields as $key => &$value) {
                $STH->bindParam($key, $value);
            }

            if ( $debug ) {
                echo "query:<br />";
                echo $query."<br />";
                print_r($conditionsfields);
                print_r($fields);
                echo "<br />";
                echo "debugDumpParams:<br />";
                $STH->debugDumpParams();
            }

            $STH->execute();

        } catch (\PDOException $e) {
            $this->logger->write($e->getMessage(), __FILE__, __LINE__);
            throw new \Exception('General malfuction!!!');
        }
    }

    /**
     * This is the basic function for one row from a table specifying the primary key
     * of the row you want to delete.
     * Once you created a instance of the DAO object you can do for example:
     *
     * $tododao->delete( 15 );
     * this will delete the row having the primary key set to 15.
     *
     * Remeber that you need to set the primary key in the tabledao.php file
     * in a costant named DB_TABLE_PK
     *
     * Example:
     * const DB_TABLE_PK = 'stp_id';
     */
    function delete( $id ) {
        try {
            $STH = $this->DBH->prepare('DELETE FROM ' . $this::DB_TABLE . ' WHERE ' . $this::DB_TABLE_PK . ' = :id');
            $STH->bindParam(':id', $id);
            $STH->execute();
        } catch (\PDOException $e) {
            $this->logger->write($e->getMessage(), __FILE__, __LINE__);
            throw new \Exception('General malfuction!!!');
        }
    }

    /**
     * This function deletes a set of row from a table depending from the
     * parameters you set when calling it.
     *
     * $tododao->delete( array( 'open' => '0', 'handling' => '1' ) );
     * this will delete the row having the field open set to 0 and the field handling set to 1.
     *
     * Remeber that you need to set the table name in the tabledao.php file
     * in a costant named DB_TABLE
     *
     * Example:
     * const DB_TABLE = 'mytablename';
     */
    function deleteByFields( $fields ) {
        $filedslist = '';
        foreach ($fields as $key => $value) {
            $filedslist .= $key . ' = :' . $key . ' AND ';
        }
        $filedslist = substr($filedslist, 0, -4);
        try {
            $STH = $this->DBH->prepare('DELETE FROM ' . $this::DB_TABLE . ' WHERE ' . $filedslist);
            foreach ($fields as $key => &$value) {
                $STH->bindParam($key, $value);
            }
            $STH->execute();
        } catch (\PDOException $e) {
            $this->logger->write($e->getMessage(), __FILE__, __LINE__);
            throw new \Exception('General malfuction!!!');
        }
    }

    /**
     * This is the basic function for getting a set of elements from a table.
     * Once you created a instance of the DAO object you can do for example:
     *
     * $tododao->getByFields( array( 'open' => '0' ) );
     * this will get all the row having the field open = 0
     *
     * you can set more then a search parameter (evaluated in AND)
     * $tododao->getByFields( array( 'open' => '0', 'handling' => '1' ) );
     *
     * you can even specify how to order the rows you requested
     * $tododao->getByFields( array( 'id' => '42' ), array('name', 'description') );
     *
     * you can even request few specific fields and not the whole table fields
     * $tododao->getByFields( array( 'id' => '42' ), array('name', 'description'), array('id', 'name', 'description') );
     */
    public function getByFields($conditionsfields, $orderby = 'none', $requestedfields = 'none') {
        $filedslist = $this->organizeConditionsFields($conditionsfields);

        $requestedfieldlist = $this->organizeRequestedFields($requestedfields);

        $orderbyfieldlist = $this->organizeOrderByFields($orderby);

        try {
            // building the query
            $query = 'SELECT ' . $requestedfieldlist . ' FROM ' . $this::DB_TABLE . ' ';
            if ($filedslist != '') {
                $query .= 'WHERE ' . $filedslist . ' ';
            }
            $query .= $orderbyfieldlist;

            $STH = $this->DBH->prepare($query);
            foreach ($conditionsfields as $key => &$value) {
                $STH->bindParam($key, $value);
            }
            $STH->execute();

            # setting the fetch mode
            $STH->setFetchMode(\PDO::FETCH_OBJ);

            return $STH;
        } catch (\PDOException $e) {
            $this->logger->write($e->getMessage(), __FILE__, __LINE__);
            throw new \Exception('General malfuction!!!');
        }
    }

    /**
     * This is the basic function for running a SQL query.
     * Once you created a instance of the DAO object you can do for example:
     *
     * $sqlQuery string containing the query
     *
     * $tododao->getBySQLQuery( 'SELECT * FROM mytable WHERE myfield = :myfieldcontent;', [ ':myfieldcontent' => '0' ] );
     * this will get all the row having the field myfield = 0
     */
    public function getBySQLQuery($sqlQuery, $fields, $debug = false) {
        try {
            $STH = $this->DBH->prepare($sqlQuery);
            foreach ($fields as $key => &$value) {
                $STH->bindParam($key, $value);
            }

            if ( $debug ) {
                echo "query:<br />";
                echo $sqlQuery."<br />";
                print_r($fields);
                echo "<br />";
                echo "debugDumpParams:<br />";
                $STH->debugDumpParams();
            }

            $STH->execute();

            # setting the fetch mode
            $STH->setFetchMode(\PDO::FETCH_OBJ);

            return $STH;
        } catch (\PDOException $e) {
            $this->logger->write($e->getMessage(), __FILE__, __LINE__);
            throw new \Exception('General malfuction!!!');
        }
    }

    /**
     * This function allows user to get a set of elements from a table.
     *
     * @param $fieldname                name of field that needs to be confronted with the array of ids
     * @param $ids                      array of ids
     * @param $conditionsfields
     * @param string $orderby
     * @param string $requestedfields
     * @return array|PDOStatement
     * @throws\Exception
     */
    public function getByFieldList($fieldname, $ids, $conditionsfields, $orderby = 'none', $requestedfields = 'none') {
        if (count($ids) > 0) {
            $ids_string = join(',', $ids);

            $filedslist = $this->organizeConditionsFields($conditionsfields);

            $requestedfieldlist = $this->organizeRequestedFields($requestedfields);

            $orderbyfieldlist = $this->organizeOrderByFields($orderby);

            try {
                // building the query
                $query = 'SELECT ' . $requestedfieldlist . ' FROM ' . $this::DB_TABLE . ' ';
                $query .= 'WHERE ' . $fieldname . ' IN (' . $ids_string . ') ';
                if ($filedslist != '') {
                    $query .= 'AND ' . $filedslist;
                }
                $query .= $orderbyfieldlist;

                $STH = $this->DBH->prepare($query);

                foreach ($conditionsfields as $key => &$value) {
                    $STH->bindParam($key, $value);
                }

                $STH->execute();

                # setting the fetch mode
                $STH->setFetchMode(\PDO::FETCH_OBJ);

                return $STH;
            } catch (\PDOException $e) {
                $this->logger->write($e->getMessage(), __FILE__, __LINE__);
                throw new \Exception('General malfuction!!!');
            }
        } else {
            return array();
        }
    }

    /**
     * This function allows user to get a set of elements from a table.
     *
     * @param  $fieldname                name of field that needs to be confronted with the array of ids
     * @param  $ids                      array of ids
     * @param  $conditionsfields
     * @param  string $orderby
     * @param  string $requestedfields
     * @return array|PDOStatement
     * @throws\Exception
     */
    public function getArrayByFieldList($fieldname, $ids, $conditionsfields, $orderby = 'none', $requestedfields = 'none') {
        if (count($ids) > 0) {
            $ids_string = join(',', $ids);

            $filedslist = $this->organizeConditionsFields($conditionsfields);

            $requestedfieldlist = $this->organizeRequestedFields($requestedfields);

            $orderbyfieldlist = $this->organizeOrderByFields($orderby);

            try {
                // building the query
                $query = 'SELECT ' . $requestedfieldlist . ' FROM ' . $this::DB_TABLE . ' ';
                $query .= 'WHERE ' . $fieldname . ' IN (' . $ids_string . ') ';
                if ($filedslist != '') {
                    $query .= 'AND ' . $filedslist;
                }
                $query .= $orderbyfieldlist;

                $STH = $this->DBH->prepare($query);

                foreach ($conditionsfields as $key => &$value) {
                    $STH->bindParam($key, $value);
                }

                $STH->execute();

                # setting the fetch mode
                $STH->setFetchMode(\PDO::FETCH_OBJ);

                $out = array();
                while ($item = $STH->fetch()) {
                    $id = $item->{$this::DB_TABLE_PK};
                    $out[$id] = $item;
                }

                return $out;
            } catch (\PDOException $e) {
                $this->logger->write($e->getMessage(), __FILE__, __LINE__);
                throw new \Exception('General malfuction!!!');
            }
        } else {
            return array();
        }
    }

    /**
     * This is the basic function for getting one element from a table.
     * Once you created a instance of the DAO object you can do for example:
     *
     * $tododao->getOneByFields( array( 'id' => '42' ) );
     * this will get the field having id = 42
     *
     * you can set more then a search parameter (evaluated in AND)
     * $tododao->getOneByFields( array( 'id' => '42', 'open' => '1' ) );
     *
     * you can even request few specific fields and not the whole table fields
     * $tododao->getOneByFields( array( 'id' => '42' ), array('id', 'name', 'description') );
     */
    public function getOneByFields($conditionsfields, $requestedfields = 'none') {
        $filedslist = $this->organizeConditionsFields($conditionsfields);

        $requestedfieldlist = $this->organizeRequestedFields($requestedfields);

        try {
            // building the query
            $query = 'SELECT ' . $requestedfieldlist . ' FROM ' . $this::DB_TABLE . ' ';
            if ($filedslist != '') {
                $query .= 'WHERE ' . $filedslist;
            }

            $STH = $this->DBH->prepare($query);
            foreach ($conditionsfields as $key => &$value) {
                $STH->bindParam($key, $value);
            }
            $STH->execute();

            # setting the fetch mode
            $STH->setFetchMode(\PDO::FETCH_OBJ);
            $obj = $STH->fetch();

            if ($obj == null) {
                $obj = $this->getEmpty();
            }

            return $obj;
        } catch (\PDOException $e) {
            $this->logger->write($e->getMessage(), __FILE__, __LINE__);
            throw new \Exception('General malfuction!!!');
        }
    }

    /**
     * This is the basic function for getting an array of elements from a table.
     * The returned array will have the entity id as index
     * Once you created a instance of the DAO object you can do for example:
     *
     * $tododao->getByFields( array( 'open' => '0' ) );
     * this will get all the row having the field open = 0
     *
     * you can set more then a search parameter (evaluated in AND)
     * $tododao->getByFields( array( 'open' => '0', 'handling' => '1' ) );
     *
     * you can even specify how to order the rows you requested
     * $tododao->getByFields( array( 'id' => '42' ), array('name', 'description') );
     *
     * you can even request few specific fields and not the whole table fields
     * $tododao->getByFields( array( 'id' => '42' ), array('name', 'description'), array('id', 'name', 'description') );
     */
    public function getArrayByFields($conditionsfields, $orderby = 'none', $requestedfields = 'none') {
        $filedslist = $this->organizeConditionsFields($conditionsfields);

        $requestedfieldlist = $this->organizeRequestedFields($requestedfields);

        $orderbyfieldlist = $this->organizeOrderByFields($orderby);

        try {
            // building the query
            $query = 'SELECT ' . $requestedfieldlist . ' FROM ' . $this::DB_TABLE . ' ';
            if ($filedslist != '') {
                $query .= 'WHERE ' . $filedslist . ' ';
            }
            $query .= $orderbyfieldlist;

            $STH = $this->DBH->prepare($query);
            foreach ($conditionsfields as $key => &$value) {
                $STH->bindParam($key, $value);
            }
            $STH->execute();

            # setting the fetch mode
            $STH->setFetchMode(\PDO::FETCH_OBJ);

            $out = array();
            while ($item = $STH->fetch()) {
                $id = $item->{$this::DB_TABLE_PK};
                $out[$id] = $item;
            }

            return $out;
        } catch (\PDOException $e) {
            $this->logger->write($e->getMessage(), __FILE__, __LINE__);
            throw new \Exception('General malfuction!!!');
        }
    }

    /**
     * This method allow to count the number of rows, contained in a table, that
     * respect given conditions.
     *
     * Once you created a instance of the DAO object you can do for example:
     *
     * $tododao->getByFields( array( 'open' => '0' ) );
     * this will get all the row having the field open = 0
     *
     * you can set more then a search parameter (evaluated in AND)
     * $tododao->getByFields( array( 'open' => '0', 'handling' => '1' ) );
     */
    public function countByFields( $conditionsfields ) {
        $filedslist = $this->organizeConditionsFields($conditionsfields);

        try {
            // building the query
            $query = 'SELECT COUNT(' . $this::DB_TABLE_PK . ') as countresult FROM ' . $this::DB_TABLE . ' ';
            if ($filedslist != '') {
                $query .= 'WHERE ' . $filedslist . ' ';
            }

            $STH = $this->DBH->prepare($query);
            foreach ($conditionsfields as $key => &$value) {
                $STH->bindParam($key, $value);
            }
            $STH->execute();

            # setting the fetch mode
            $STH->setFetchMode(\PDO::FETCH_OBJ);

            $out = 0;
            if ( $item = $STH->fetch() ) {
                $out = $item->countresult;
            }

            return $out;
        } catch (\PDOException $e) {
            $this->logger->write($e->getMessage(), __FILE__, __LINE__);
            throw new \Exception('General malfuction!!!');
        }
    }

    /**
     * This method allow to count the number of rows, contained in a table, that
     * respect given conditions.
     *
     * @param $fieldname                name of field that needs to be confronted with the array of ids
     * @param $ids                      array of ids
     * @param $conditionsfields
     * @param string $orderby
     * @param string $requestedfields
     * @return array|PDOStatement
     * @throws\Exception
     */
    public function countByFieldList($fieldname, $ids, $conditionsfields, $orderby = 'none', $requestedfields = 'none') {
        if (count($ids) > 0) {
            $ids_string = join(',', $ids);

            $filedslist = $this->organizeConditionsFields($conditionsfields);

            $orderbyfieldlist = $this->organizeOrderByFields($orderby);

            try {
                // building the query
                $query = 'SELECT ' . $fieldname . ', COUNT(' . $this::DB_TABLE_PK . ') as countresult FROM ' . $this::DB_TABLE . ' ';
                $query .= 'WHERE ' . $fieldname . ' IN (' . $ids_string . ') ';
                if ($filedslist != '') {
                    $query .= 'AND ' . $filedslist;
                }
                $query .= ' GROUP BY ' . $fieldname;
                $query .= $orderbyfieldlist;

                $STH = $this->DBH->prepare($query);

                foreach ($conditionsfields as $key => &$value) {
                    $STH->bindParam($key, $value);
                }

                $STH->execute();

                # setting the fetch mode
                $STH->setFetchMode(\PDO::FETCH_OBJ);

                $out = array();
                while ($item = $STH->fetch()) {
                    $out[$item->{$fieldname}] = $item->countresult;
                }

                return $out;
            } catch (\PDOException $e) {
                $this->logger->write($e->getMessage(), __FILE__, __LINE__);
                throw new \Exception('General malfuction!!!');
            }
        } else {
            return array();
        }
    }

    /**
     * This method get just one filed from a table
     *
     * @param $fieldname                name of field to get
     * @param $conditionsfields         conditions evaluated in AND
     * @return the field content
     * @throws\Exception
     */
    public function getOneField($fieldname, $conditionsfields) {
        $filedslist = $this->organizeConditionsFields($conditionsfields);

        try {
            // building the query
            $query = 'SELECT ' . $fieldname . ' as countresult FROM ' . $this::DB_TABLE . ' ';
            if ($filedslist != '') {
                $query .= 'WHERE ' . $filedslist;
            }

            $STH = $this->DBH->prepare($query);

            foreach ($conditionsfields as $key => &$value) {
                $STH->bindParam($key, $value);
            }

            $STH->execute();

            # setting the fetch mode
            $STH->setFetchMode(\PDO::FETCH_OBJ);

            while ($item = $STH->fetch()) {
                $out = $item->countresult;
            }

            if ( isset($out) ){
                return $out;
            } else {
                return '';
            }

        } catch (\PDOException $e) {
            $this->logger->write($e->getMessage(), __FILE__, __LINE__);
            throw new \Exception('General malfuction!!!');
        }
    }

    public function getEmpty() {
        return null;
    }

    public function organizeConditionsFields($conditionsfields) {
        $filedslist = '';
        foreach ($conditionsfields as $key => $value) {
            $filedslist .= $key . ' = :' . $key . ' AND ';
        }
        $filedslist = substr($filedslist, 0, -4);
        return $filedslist;
    }

    public function organizeRequestedFields($requestedfields) {
        if (is_array($requestedfields)) {
            $requestedfieldlist = '';
            foreach ($requestedfields as $value) {
                $requestedfieldlist .= $value . ', ';
            }
            $requestedfieldlist = substr($requestedfieldlist, 0, -2);
        } else {
            $requestedfieldlist = '*';
        }
        return $requestedfieldlist;
    }

    public function organizeOrderByFields($orderby) {
        if (is_array($orderby)) {
            $orderbyfields = ' ORDER BY ';
            foreach ($orderby as $value) {
                $orderbyfields .= $value . ', ';
            }
            $orderbyfields = substr($orderbyfields, 0, -2);
        } else {
            $orderbyfields = '';
        }
        return $orderbyfields;
    }

    public function putCache($query, $key, $stuff) {
        $dirname = 'cache/' . $query;
        if (!file_exists($dirname)) {
            mkdir($dirname, 0777, true);
        }
        $filename = $dirname . '/' . $key . '.data';
        file_put_contents($filename, serialize($stuff), LOCK_EX);
    }

    public function getCache($query, $key) {
        $filename = 'cache/' . $query . '/' . $key . '.data';
        $cached = NULL;
        if (file_exists($filename) AND ( time() - filemtime($filename) < 2 * 3600)) { // 2 h
            $cached = unserialize(file_get_contents($filename));
        }
        return $cached;
    }

}


