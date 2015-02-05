.. Slick common component - Singletons

Other usages
------------

.. contents:: Table of Contents
    :depth: 2

Creating Singletons
...................

If you need to create a singleton object and have all the ``Slick\Common\Base`` behavior
this Slick component also have a ``Slick\Common\BaseSingleton`` that implements the
``Slick\Common\SingletonInterface`` witch has the getInstance() method you need to define.

Lets see an example: ::

    use Slick\Common\BaseSingleton;

    class MyClass extends BaseSingleton
    {
        /**
         * @readwrite
         * @var string
         */
        protected $_name;

        private static $_instance;

        public static function getInstance($options = array())
        {
            if (is_null(static::$_instance)) {
                static::$_instance = new Static($options);
            }
            return static::$_instance;
        }
    }

    $myClass = MyClass::getInstance(['name' => 'Foo']);
    echo $myClass->getName();   // This will print out "Foo"

All the features are present in SingletonBase class and you only need to implement the
``Slick\Common\SingletonInterface::getInstance()`` method.

Using "Base" behavior in other classes
......................................

Is is also possible to add the ``Slick\Common\Base`` behavior to a class that you
didn't define or you cannot extend. Slick common component has a trait called
``Slick\Common\BaseMethods`` that adds the same features to the class you are
working with. Check out this example: ::

    use DateTime as PHPDateTime;
    use Slick\Common\BaseMethods;

    class DateTime extends PHPDateTime
    {
        /**
         * @read
         * @var string
         */
        protected $_stamp = 'foo';

        // Add base class behavior and methods
        use BaseMethods;
    }

    $date = new DateTime('now');
    echo $date->getStamp();        // -> this will print out 'foo'

.. important::

    You should use the magic getters/setters/properties only with the properties you
    define in your class. Using it in the super class properties can lead to
    unpredictable results.