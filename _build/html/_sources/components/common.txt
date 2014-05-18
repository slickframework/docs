.. Common library

Common library
~~~~~~~~~~~~~~
Common library is a set of useful classes and traits that are in almost every class
in the entire Slick framework. They form a solid base to develop on top of and help
you remove the tedious work of create getters, setters, allow read end or write access
to the properties and inspect classes and properties.

Understanding the "Base" class
------------------------------
The ``Slick\Common\Base`` class is one of the most important classes of Slick. It is responsible for the
"magic" in classes that extends it. It is very important that you have a clear understanding of what
this class does to really speed up your development.

Defining class properties
"""""""""""""""""""""""""
When defining properties in classes that extend ``Slick\Common\Base`` you have to follow a simple convention:

- properties must be prefixed with an "_" (underscore);
- property visibility is defined as ``protected``;
- property names are camel cased;
- use one of ``@read``, ``@write`` or ``@readwrite`` notations to define the public access mode to those properties.

Lets see an example:

.. code-block:: php

    <?php

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

    $myClass = new MyClass();

    $myClass->name = "Foo";
    echo $myClass->name;        // This will print out "Foo"


As you can see in this example, defining property ``MyClass::$_name`` is in fact very simple. The ``@readwrite``
notation says that it has read/write access to it and it is a protected property.

Did you notice the way we access the ``MyClass::$_name``? Yes, we access it as it was a ``public`` property!

This is the first feature of ``Slick\Common\Base``: to expose this properties in a ``public`` fashion
using the PHP ``__get()`` and ``__set()`` magic methods.

The notations used are defined as followed:

* ``@read`` The property can be accessed as a read only. Any attempt to assign a value to a read only property will raise a ``Slick\Common\Exception\ReadOnlyException`` exception. This is the default behavior if no notation is provided;
* ``@write`` A value can be assigned to the property. Any attempt to read the property value will raise a ``Slick\Common\Exception\WriteOnlyException`` exception;
* ``@readwrite`` The property can be read or changed as a ``public`` property.


.. note::

    This feature can be also achieved using ``private`` as visibility, however it will not work with objects
    of classes that extend the one using ``private`` visibility as it cannot be accessed by them.

.. warning::

    Any attempt to assign or read an undefined property on classes that extends ``Slick\Common\Base`` will
    raise a ``Slick\Common\Exception\UndefinedPropertyException`` exception. This is a great way to avoid
    dynamic change of objects.


Getters and Setters
"""""""""""""""""""
``Slick\Common\Base`` class also has a public getter and setter for the properties defined we seen above.

Consider the following example:

.. code-block:: php

    <?php

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

    $myClass = new MyClass();

    $myClass->setName("Foo");
    echo $myClass->getName();      // This will print out "Foo"

If you follow the convention you will have getters and setters for all defined properties.

This is another cool feature of Base class: Expose getters and setters to all the properties using
PHP ``__call()`` magic method.

It is possible to chain calls with setters as they will always return the object they called in.
For example:

.. code-block:: php

    <?php

    $myClass = new MyClass();

    $myClass->setName("Bar")
        ->getName();            // This will output "Bar"


As you can see the getter method was chained in the return of the setter as it will return the ``$myClass`` object.

The setter is always in the way
"""""""""""""""""""""""""""""""
Another benefit of extending the Base class it that you can define the way a given property gets its value
even when you assign the value as a public property.

Lest see an example:

.. code-block:: php

    <?php

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
    }

    $myClass = new MyClass();

    $myClass->name = "Foo Bar";
    echo $myClass->getName();   // This will print out FOO BAR

In the code above we define the ``MyClass::setName()`` method and as you can see by assign a value to the
name property the setName() method is always called.
This is a good way of doing "lazy load" of properties and ensure that even if you use the property instead of
the method the same behavior is performed.

.. note::

    In the above example we add the ``return $this;`` line to maintain the same behavior of the setters
    as it is defined by Base class. If you don't put that line as the setter return you will no longer
    be able to chain other method calls.

Easy construction
"""""""""""""""""
When creating an object you will probably need to set some properties before you use that object. The way of
doing this is by adding the needed parameters to the constructor or by calling the setters you need to properly
set the state of your object before using it.

But adding parameters to the constructor leads to a question of what parameters to use, in what order and will they
have a default value? you will end up with a big, ugly constructor signature.
Calling the the setters is more elegante but still is not as clean as i can be.

The Base class has a constructor that accepts an array or object as the values for its properties.

Lets see the following code:

.. code-block:: php

    <?php

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

As you can see passing an array of associative key/value pairs array where keys are the property names you want to set
when creating the object is a more elegant and flexible way of setting the object state before using it.

.. warning::

    If in your class you need to define a constructor always remember that you will need to call
    ``parent::__construct()`` cause it will raise an exception if you do not do so.

Creating Singletons
-------------------
If you need to create a singleton object and have all the ``Slick\Common\Base`` behavior this Slick component
also have a ``Slick\Common\BaseSingleton`` that implements the ``Slick\Common\SingletonInterface`` witch has
the getInstance() method you need to define.

Lets see an example:

.. code-block:: php

    <?php

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

All the features are present in SingletonBase class and you only need to implement the getInstance() method.