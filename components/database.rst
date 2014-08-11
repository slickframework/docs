.. Slick database component

Database Access
===============

Slick Database Model is a simple database access layer that aims to abstract
the data access operations.

For this version it only supports the Mysql and Sqlite databases but it defines
a good set of interfaces that allow you do create your database adapters as well.

**Installation**

To install this component add the following line to your ``composer.json``

.. code-block:: json

    {
        "require": {
            "slick/database": "~1.1.0",
            ...
        }
    }

Create a database adapter
-------------------------

For accessing a database you will need a database adapter that provides basic
operations like connecting and querying a database server.

So lets start by creating an Mysql adapter for example::

    use Slick\Database\Adapter;

    $adapter = new Adapter([
        'driver' => 'Mysql',
        'options' => [
            'database' => 'test',
            'username' => 'root',
            'host' => 'localhost'
        ]
     ]);

The ``Slick\Database\Adapter`` factory allows you to create an adapter by passing
a driver name and an array of specific database options.

Right now you don't have an adapter. ``$adapter`` is only a factory object that
generalises the database adapter construction.

To create the adapter you need to initialize it::

    $mysql = $adapter->initialize();

``Adapter::initialize()`` will create and connect the database adapter for you.
It may throw an ``Slick\Database\Exception`` if it can't connect to the the
database server, so it wised to wrap it in a try... catch block::

    try {
        $mysql = $adapter->initialize();
    } catch (Slick\Database\Exception $exp) {
        // Handle the exception
    }

Database adapter usage
----------------------

Now that we have our database we can start using it. The implemented driver here
use the `PHP Data Objects (PDO) <http://php.net/manual/en/book.pdo.php>`_ extension
so it proxies some basic methods and as a getter for the PDO object after
initialization.

**Query data**
So lets query some data out of our database::

    $sql = "SELECT * FROM users WHERE id = :id";
    $params = [':id' => 2];

    $data = $mysql->query($sql, $params);

``$data`` is a ``Slick\Database\RecordList``, a special object that implements
``Countable``, ``ArrayAccess``, ``IteratorAggregate`` and ``Serializable`` interfaces
and holds a list of records.

.. tip::

    Although ``$data`` be an object it can be used as an array as it implements
    ``ArrayAccess`` interface. You can use things like::

        $first = reset($data); // first item of the list.

    It also implements ``Serializable`` in order to be used with
    ``Slick\Cache\DriverInterface``

As you may notice the binding of the parameters in the query method is similar
to the ``PDOStatement`` witch can use the ? placeholder of ``:name`` as a
named placeholder.

.. note::

    In background the query method is creating a PDO prepared statement and
    binding the values to its placeholders. So if you never used prepared
    statements before you should read more about it in the
    `Prepared statements and stored procedures <http://php.net/manual/en/pdo.prepared-statements.php>`_
    manual.

Other adapter methods
---------------------

Database adapter has other methods:

 * ``execute($sql, $params)`` Executes a SQL query but returns the affected rows::

    $affected = $mysql->execute(
        'UPDATE users SET active=? WHERE id = ?',
        [0, 1]
    );
    print $result; // will output 1

 * ``getAffectedRows()`` Returns the affected rows by last executed SQL statement

 * ``getLastInsertId()`` Returns the ID of the last row to be inserted::

    $mysql->execute(
        'INSERT INTO users (name, email) VALUES (:name, :email),
        [
            ':name' => 'Jon Doe',
            ':email' => 'jon.doe@example.com'
        ]
    );
    print $mysql->getLastInsertId(); // Will output something like 43

 * ``getLastError()`` The description of the latest error

.. note::

    The ``execute()`` and ``query()`` methods may throw
    ``Slick\Database\Exception\SqlQueryException`` exceptions so its wised
    to wrap these methods in a ``try{...} catch(){}`` block.


Using the PDO object
--------------------

As seen above, the adapter uses the PHP PDO extension to deal with the database
and you can always use it in your. Look at the following code::

    $pdo = $mysql->getHandler();

    $pdo->beginTransaction();

    /* Change the database schema and data */
    $sth = $pdo->exec("DROP TABLE fruit");
    $sth = $pdo->exec("UPDATE dessert SET name = 'hamburger'");

    /* Recognize mistake and roll back changes */
    $pdo->rollBack();

In this script we use the PDO object to perform a transaction in the database.

.. toctree::
    :maxdepth: 1

    database/sql