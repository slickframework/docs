.. Slick database/sql component

Using the SQL object
====================

Slick database component also comes with a SQL object that helps you create
SQL statements regardless of the database dialect that you adapter uses.

The main goal is to write your SQL so it can be executed in any database server.

.. contents:: Table of Contents
    :depth: 3

Select statement
----------------

Creating a ``SELECT`` statement is very ease::

    use Slick\Database\Sql;
    use Slick\Database\Adapter;

    $adapter = new Adapter([
        'driver' => 'Mysql',
        'options' => [
            'database' => 'test',
            'username' => 'root',
            'host' => 'localhost'
        ]
    ]);
    $adapter = $adapter->initialize();

    $data = Sql::createSql($adapter)
        ->select('users', ['name', 'email'])
        ->where('active = ?', [true])
        ->order('name DESC')
        ->limit(10)
        ->all();

``$data`` is a record list with the first 10 rows of users with email and name
fields that are active and ordered by name, descending. Cool, isn't it!?

Filtering data with "where"
___________________________

The where method in the select SQL object mimics the ``WHERE`` clause in the SQL
language. So it is possible to set the filter conditions and use it with PDO
Prepared statement in a single call.

``Slick\Database\Sql\Select::where()`` method accepts two arguments: the conditions
and the values that need to be bind in the query call.

So lets see some examples::

    $select = Sql::create($adapter)->select('users');

    $select->where('active = 1'); // A literal condition that will be used as is.

    $select->where('active = ?', [1]) // use of positional ? placeholders

    $select->where(
        'active = :state',
        [
            ':state' => 1
        ]
    ); // use of named placeholders

All the call to the ``where()`` method above will result in the same query condition but allows
you to decide whenever you want the values to be bind and cleaned, using prepared statements
or enter other conditions that don't need to use these features.

.. note::

    The call to ``Slick\Database\Sql\Select::where()`` method is cumulative and by default
    it will use the ``AND`` boolean operation to aggregate all the conditions.

It is possible to add multiple conditions and the way they are combined with each other. The
following example will use the ``OR`` operation with 2 conditions::

    $select->where('active = 1')
        ->orWhere('name LIKE :name', [':name' => "%{$name}%]);

    // The resulting condition is
    // 'WHERE active = 1 OR name LIKE :name'

An ``andWhere()`` method is also available in order to allow more conditions to be aggregated.

.. tip::

    When using bind parameters and prepared statements it's preferable to use the named
    placeholders as they are more understandable and cab be reused. ? position placeholders
    are great for simple condition with fewer parameters.

Joining other tables
____________________

If you need to join other tables to que Select query you can use the ``join()``
method as follows::

    $select->join(
        'profiles',             // -> the table name
        'prof.uid = users.id',  // -> the join condition
        ['age', 'group'],       // -> (optional) the field list to retrieve
        'prof',                 // -> (optional) table alias
        Join::JOIN_LEFT         // -> (optional) the join type
    );

The above code will produce a left join with table profiles using the alias ``prof``
and retrieving the ``age`` and ``group`` fields on recodes where ``uid`` is equal to the user's
table primary key (``id`` field).
This way of create a join is very similar to the SQL it self witch leads to a very short
learning curve to use it and with the advantage of working if your database adapter
changes.

.. note::

    Some notes about the parameters:

    * If the field list is left blank it will use the ``*`` wildcard. if you just want to join the table but without the fields you need to set this parameter to ``null``;

    * If you enter an alias to the table in the 4th parameter you need to use that alias in the ``ON`` clause in the 2nd parameter;

    * Use the ``Slick\Database\Sql\Select\Join::JOIN_xxx`` constants to set the proper join type. Please see the available join types in the table bellow.

    +-------------+-----------------+
    | **Constant**| **SQL**         |
    +-------------+-----------------+
    | JOIN_INNER  | INNER JOIN      |
    +-------------+-----------------+
    | JOIN_LEFT   | LEFT JOIN       |
    +-------------+-----------------+
    | JOIN_RIGHT  | RIGHT JOIN      |
    +-------------+-----------------+
    | JOIN_FULL   | FULL OUTER JOIN |
    +-------------+-----------------+

Ordering the results
____________________

Adding order to the ``SELECT`` statement its similar to the SQL, as all the methods we have
seen so far. Just add the order clause as you would do in regular SQL query::

    $select->order("users.name DESC");

.. Note::
    The argument passed to the ``Slick\Database\Sql\Select::order()`` method is used directly
    in the SQL ``SELECT`` statement sent to the database adapter. The order clause in SQL is
    almost identical over the different database systems but you should validate it if you
    change the database adapter.