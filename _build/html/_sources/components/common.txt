.. Common library

Common
~~~~~~
Slick common component library is a set of useful classes and traits that are in almost every class
in the entire Slick framework. They form a solid base to develop on top of and help
you remove the tedious work of create getters, setters, allow read and/or write access
to the properties and inspect classes and properties.

.. contents:: Table of Contents
    :depth: 3

.. toctree::
    :maxdepth: 2

    common/singleton
    common/inspector

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
         "slick/common": "*"
      }
   }

Understanding the "Base" class
------------------------------
The ``Slick\Common\Base`` class is one of the most important classes of Slick. It is
responsible for the "magic" in classes that extends it. It is very important that you
have a clear understanding of what this class does to really speed up your development.

Defining class properties
"""""""""""""""""""""""""
When defining properties in classes that extend ``Slick\Common\Base`` you have to follow
a simple convention:

    * properties must be prefixed with an "_" (underscore);
    * property visibility is defined as ``protected``;
    * property names are camel cased;
    * use one of ``@read``, ``@write`` or ``@readwrite`` annotations to define the public
      access mode to those properties.

Lets see an example: ::

    use Slick\Common\Base;

    class MyClass extends Base
    {
        /**
         * @readwrite
         * @var string
         */
        protected $_name;

        // More code follows
    }

    $myClass = new MyClass();
    $myClass->name = "Foo";
    echo $myClass->name;        // This will print out "Foo"

As you can see in this example, defining property ``MyClass::$_name`` is in fact very
simple. The ``@readwrite`` annotation grants read and write access to it and it is a
protected property. Did you notice the way we access the ``MyClass::$_name``? Yes, we
access it as it was a ``public`` property! This is the first feature of
``Slick\Common\Base``: to expose this properties in a ``public`` fashion using the PHP
``__get()`` and ``__set()`` magic methods.

.. important::

    The property visibility is set to ``protected`` so that the methods from
    ``Slick\Common\Base`` can have access to it and it cannot be accessed publicly
    from other objects.

+--------------------------------------------------------------------------------+
| The annotations used are defined as follows                                    |
+================+===============================================================+
| ``@read``      | The property can be accessed as a read only.                  |
+----------------+---------------------------------------------------------------+
| ``@write``     | A value can be assigned to the property.                      |
+----------------+---------------------------------------------------------------+
| ``@readwrite`` | The property can be read or changed as a ``public`` property. |
+----------------+---------------------------------------------------------------+

.. attention::

    Any attempt to assign a value to a ``@read`` only property will raise a
    ``Slick\Common\Exception\ReadOnlyException`` exception. This is also the behavior
    if no annotation is provided.

    Any attempt to read a value from a ``@write`` only property will
    raise a ``Slick\Common\Exception\WriteOnlyException`` exception.

    Any attempt to assign or read an undefined property on classes that extends
    ``Slick\Common\Base`` will raise a ``Slick\Common\Exception\UndefinedPropertyException``
    exception. This is a great way to avoid dynamic change of objects.

.. tip::

    As a way of improve IDE development, it is useful to add the public properties to
    the class DocBlock. This will help IDEs to "see" the properties as if they were
    defined as ``public``. The code bellow shows how to achieve this result::

        /**
         * Class Foo
         *
         * @property       string $foo Read/Write property
         * @property-read  int    $bar Read only property
         * @property-write mixed  $baz Write only property
         */
        Class Foo extends Base {
        // The class definition

Getters and Setters
"""""""""""""""""""
``Slick\Common\Base`` class also has a public getter and setter for the properties
defined as we have seen above. Consider the following example::

    use Slick\Common\Base;
    class MyClass extends Base
    {
        /**
         * @readwrite
         * @var string
         */
        protected $_name;

        // Rest of the class code
    }

    $myClass = new MyClass();
    $myClass->setName("Foo");
    echo $myClass->getName();      // This will print out "Foo"

If you follow the convention you will have getters and setters for all defined
properties. This is another cool feature of ``Slick\Common\Base`` class: Expose
getters and setters to all the properties using PHP ``__call()`` magic method.

It is possible to chain calls with setters as they will always return the object
they are called in. For example::

    $myClass = new MyClass();
    $myClass->setName("Bar")
        ->getName();            // This will return "Bar"


In this code example the getter method was chained in the return of the setter as it
will return the ``$myClass`` instance.

The setter/getter is always in the way
""""""""""""""""""""""""""""""""""""""
Another benefit of extending the Base class it that you can define the way a given
property gets its value even when you assign the value as a ``public`` property. ::

    use Slick\Common\Base;

    class MyClass extends Base
    {
        /**
         * @readwrite
         * @var string
         */
        protected $_name;

        public function setName($name)
        {
            $this->_name = strtoupper($name);
            return $this;
        }

        public function getName()
        {
            if (is_null($this->_name)) {
                $this->_name = "No Name";
            }
            return $this->_name;
        }
    }

    $myClass = new MyClass();
    $myClass->name = "Foo Bar";
    echo $myClass->getName();   // This will print out FOO BAR

In the code above we define the ``MyClass::setName()`` method, overriding the
behavior that ``Slick\Common\Base`` class implements and, as you can see, by assign
a value to the ``MyClass::$name`` property, the ``MyClass::setName()`` method is
executed. In the same way, the ``MyClass::getName()`` is executed
when you retrieve the property value. This is in fact a good way of doing "lazy
load" of expensive properties and ensure that even if you call the property the
method and behavior are always the same.

.. tip::

    This feature allows us to force properties to be of a certain type if you
    define the setter and use the `PHP type hinting`__ feature. ::

        use Slick\Common\Base;

        class MyClass extends Base
        {
            /**
             * @readwrite
             * @var \DateTime
             */
            protected $_date;

            public function setDate(\DateTime $date)
            {
                $this->_date = $date;
                return $this;
            }
        }

    When you try to assign a value to ``MyClass::$date`` property, the correspondent
    setter is called and the value is used to pass as argument to the setter. If
    it isn't from the specified type, a catchable fatal error is triggered.


.. _phpTypeHint: http://php.net/manual/en/language.oop5.typehinting.php

__ phpTypeHint_

.. note::

    In the above example we add the ``return $this;`` line to maintain the same
    behavior of the setters as it is defined by ``Slick\Common\Base`` class.
    If you don't put that line as the setter return you will no longer be able
    to chain other method calls.

Easy construction
"""""""""""""""""
When creating an object you will probably need to set some properties before you
use that object. The way of doing this is by adding the needed parameters to the
constructor or by calling the setters you need to properly set the state of your
object before using it.

But adding parameters to the constructor leads to a question of what parameters
to use, in what order and will they have a default value? You will end up with a big,
ugly constructor signature. Calling the the setters is more elegant but still is
not as clean as it can be. The ``Slick\Common\Base`` class has a constructor that
accepts an array or an object as the values for its properties. Lets see the
following code: ::

    use Slick\Common\Base;

    class MyClass extends Base
    {
        /**
         * @readwrite
         * @var string
         */
        protected $_name;

        ...
    }

    $myClass = new MyClass(['name' => 'Foo']);
    echo $myClass->getName();      // This will print out "Foo"

As you can see, passing an array of associative key/value pairs where keys are the
property names you want to set when creating the object is a more elegant and
flexible way of setting the object state before using it.

.. warning::

    If in your class you need to define a constructor always remember that you will
    need to call ``parent::__construct()`` otherwise it will raise an exception
    if you do not do so.
