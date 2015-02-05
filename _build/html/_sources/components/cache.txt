.. Slick cache component

Cache
=====

Slick cache component deals with cache providing services installed on your system.

I comes with support for *Memcached* (``memcached`` daemon) and *File* (caching data into files) out of the box,
but it also defines a driver interface that allow you to add your own drivers to your project.

.. contents:: Table of Contents
    :depth: 2

Installing via Composer
-----------------------

The recommended way to install Slick cache component is through `Composer <https://getcomposer.org/>`_.

.. code-block:: bash

    # Install Composer
    curl -sS https://getcomposer.org/installer | php

You can add Slick cache as a dependency using the composer.phar CLI:

.. code-block:: bash

    php composer.phar require slick/cache

Alternatively, you can specify Slick cache as a dependency in your project's
existing composer.json file:

.. code-block:: js

    {
      "require": {
         "slick/cache": "*"
      }
   }

Using a cache driver
--------------------

To use a cache driver you simple need to call the ``Cache::get()`` static method
to get an initialized cache driver. Check the following example::

    use Slick\Cache\Cache;

    $cache = Cache::get();
    $data = $cache->get('data', false);

    if (!$data) {
        $data = file_get_contents("http://www.example.com/api/call.json");
        $cache->set('data', $data);
    }



In this example we are using the default cache driver with default options, to store
some expensive API call data.

.. note::
    The default driver is Memcached with the following default options:
        * ``duration: 120``
        * ``host: '127.0.0.1'``
        * ``port: 11211``

Changing cache expire time
~~~~~~~~~~~~~~~~~~~~~~~~~~

The expire amount of time is always set when you set a value on the cache driver.
As mention above, the default is set to 120 seconds. Using the above example, we will
set the time expire amount to 3 minutes for the data from our fictitious API call::

    use Slick\Cache\Cache;

    $cache = Cache::get();
    $data = $cache->get('data', false);
    if (!$data) {
        $data = file_get_contents("http://www.example.com/api/call.json");
        // Set expire to 3 minutes
        $cache->set('data', $data, 3*60);
    }

It is also possible to define a global expire amount of time for all ``Cache::set()``
like this::

    use Slick\Cache\Cache;

    $cache = Cache::get();
    // Set global cache expire to 10 minutes
    $cache->duration = 10*60;

    $data = $cache->get('data', false);
    if (!$data) {
        $data = file_get_contents("http://www.example.com/api/call.json");
        // This will use the 10 minutes setting from above
        $cache->set('data', $data);
    }

Slick\\Cache\\DriverInterface::set()
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    Set/stores a value with a given key. If no value is set in the expire parameter the
    default ``Cache::duration`` will be used.

.. code-block:: php

    public DriverInterface DriverInterface::set(string $key, mixed $value [, int $expire = -1])

+------------+--------+------------------------------------+
| Parameters | Type   | Description                        |
+============+========+====================================+
| $key       | string | The key where value will be stored |
+------------+--------+------------------------------------+
| $value     | mixed  | The value to store                 |
+------------+--------+------------------------------------+
| $expire    | int    | The live time of cache in seconds  |
+------------+--------+------------------------------------+

+-------------------------------+-----------------------------------------------------------+
| Return                        | Description                                               |
+===============================+===========================================================+
| Slick\\Cache\\DriverInterface | A ``DriverInterface`` instance for chaining method calls. |
+-------------------------------+-----------------------------------------------------------+


Slick\\Cache\\DriverInterface::get()
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    Retrieves a previously stored value. You can optionally set the value returned
    in case of cache driver has no value for provided key.

.. code-block:: php

    public mixed DriverInterface::get(string $key [, mixed $default = false])

+------------+--------+------------------------------------------------------------------+
| Parameters | Type   | Description                                                      |
+============+========+==================================================================+
| $key       | string | The key where value was stored                                   |
+------------+--------+------------------------------------------------------------------+
| $default   | mixed  | The value returned if cache driver has no value for provided key |
+------------+--------+------------------------------------------------------------------+

+--------+-----------------------------------------------------------+
| Return | Description                                               |
+========+===========================================================+
| mixed  | The stored value or the default value if cache driver has |
|        | no value for provided key                                 |
+--------+-----------------------------------------------------------+


Slick\\Cache\\DriverInterface::erase()
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    Erase the value stored with a given key.
    You can use the "?" and "*" wildcards to delete all matching keys.
    The "?" means a place holders for one unknown character, the "*" is
    a place holder for various characters.

.. code-block:: php

    public DriverInterface DriverInterface::erase(string $key)

+------------+--------+------------------------------------------------------------------+
| Parameters | Type   | Description                                                      |
+============+========+==================================================================+
| $key       | string | The key where value was stored                                   |
+------------+--------+------------------------------------------------------------------+

+-------------------------------+-----------------------------------------------------------+
| Return                        | Description                                               |
+===============================+===========================================================+
| Slick\\Cache\\DriverInterface | A ``DriverInterface`` instance for chaining method calls. |
+-------------------------------+-----------------------------------------------------------+

.. warning::
    The use of "?" and "*" placeholder is only implemented in the drivers that are
    provided by Slick cache component. If you create your own cache driver you need
    to handle the placeholders key search implementation.

.. tip::
    If you are implementing your own cache driver and want to have the "?" and "*"
    placeholders search you can extend ``Slick\Cache\Driver\AbstractDriver`` witch
    uses the ``DriverInterface::get()`` and ``DriverInterface::set()`` methods to
    achieve the wildcards key search feature.


Slick\\Cache\\DriverInterface::flush()
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    Flushes all values controlled by this cache driver.

.. code-block:: php

    public DriverInterface DriverInterface::flush()

+-------------------------------+-----------------------------------------------------------+
| Return                        | Description                                               |
+===============================+===========================================================+
| Slick\\Cache\\DriverInterface | A ``DriverInterface`` instance for chaining method calls. |
+-------------------------------+-----------------------------------------------------------+

Initializing cache with Slick\\Configuration
--------------------------------------------

Probably you are using Slick components in your application and therefor you must
probably have the Slick\\Configuration component.

It is normal to have a settings file for your application and use its values to
initialize resources such as Slick\\Cache component.

So lets start by creating our ``settings.php`` file::

    return [
        'driver' => 'memcached',
        'options' => [
            'host' => '127.0.0.1',
            'port' => 11211,
            'prefix' => 'my.app'
        ]
     ];

Now that we have set the settings lets use them::

    use Slick\Cache\Cache;
    use Slick\Configuration\Configuration;

    // Retrieve the settings
    Configurations::addPath(dirname(__FILE__));
    $settings = Configuration::get('settings');

    $cache = Cache::get(
        $settings->get('driver'),
        $settings->get('options')
    );

    // Use your cache driver
    $cache->get('someKey', false);

The Memcached cache driver
--------------------------
The Memcached cache driver uses the `memcached <http://www.memcached.org/>`_, an
high-performance, distributed memory object caching system.

Memcached is an in-memory key-value store for small chunks of arbitrary data (strings,
objects) from results of database calls, API calls, or page rendering.

.. warning::

    You must have the Memcached PECL extension installed on your system to be able to use the
    Memcached cached diver provided by Slick cache component.

    If you need help on have your system installed with Memcached extension please visit the
    `PHP Memcached manual page <http://www.php.net/manual/en/memcached.installation.php>`_ for
    more information.

To use this driver, as we already saw before, you need to call the Cache::get() static method
and pass the driver name and options.

The following example illustrates a possible way of doing it::

    use Slick\Cache\Cache;

    $cache = Cache::get('memcached', [
        'host' => '0.0.0.0',
        'port' => '11211',
        'duration' => 300, // 5 minutes
        'prefix' => 'my.cache'
    ]);


Here we pass the host and port of the memcached daemon for PHP Memcached to connect.

If you were paying attention to the last code block you will notice that we add the
``duration`` param to the diver initialization. It is possible to do that on all
cache drivers.


The File cache driver
---------------------

The File cache driver uses the file system to store the cached values. So for every
data key that you want to store in cache the driver will create a file with it.

.. note::

    Storing cache data into files is not the best way of doing cache and therefor you
    should consider using a better driver like Memcached.

    This driver was created for those situations where you don't have access to your
    and still want to cache *expensive* resources.

Lets look at the following example::

    use Slick\Cache\Cache;

    $cache = Cache::get('file', ['path' => './tmp/', 'prefix' => 'my.cache']);
    $data = $cache->get('data', false);
    if (!$data) {
        $data = file_get_contents("http://www.example.com/api/call.json");
        $cache->set('data', $data);
    }

In this case we are setting a different path where we want to save our cache files.

.. raw::html

    <div id="disqus_thread"></div>
    <script type="text/javascript">
        /* * * CONFIGURATION VARIABLES: EDIT BEFORE PASTING INTO YOUR WEBPAGE * * */
        var disqus_shortname = 'slick-framework'; // required: replace example with your forum shortname

        /* * * DON'T EDIT BELOW THIS LINE * * */
        (function() {
            var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
            dsq.src = '//' + disqus_shortname + '.disqus.com/embed.js';
            (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
        })();
    </script>
    <noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>

