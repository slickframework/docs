.. Dependency injector container documentation

Dependency Injection Container
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. contents:: Table of Contents
    :depth: 2

Dependency injection and dependency injection containers are tow different things. Dependency injection is a way
of doing better code as you can read in this Fabien Pontencier's
`article about dependency injection <http://fabien.potencier.org/article/11/what-is-dependency-injection>`_.
In other hand *Dependency Injection Container* is a tool that will help you injectic dependencies.

This component for Slick is a very simple library where you can define dependencies and have them injected
in your object as you call for them.

Installation
------------

To install this component add the following line to your ``composer.json``

.. code-block:: json

    {
        "require": {
            "slick/di": "*",
        }
    }

Defining injections
-------------------

Container builder needs an array of definitions to build a dependency container. Definition class helps you
define those injections.

**Value definition**

A value definition is used to store a scalar value or object. This allows you to use the dependency injector
as a *registry* component storing all your values where you can reuse them.

Lets see an example:

.. code-block:: php

    <?php

    return [
        'myValue' => 1024
    ];

As you can see this is the simplest way of defining a value to be stored as ``myValue`` in the container
you will create.

**Factory definition**

A callable or *Factory* definition is used when you want to calculate the value when you call the container for that
definition. You need to define a callable or clojure for this definition.

Lets see an example:

.. code-block:: php

    <?php

    return [
        'myCalculatedValue' => \Slick\Di\Definition::factory(function(){
            return new \DateTime("now", new \DateTimeZone('UTC'));
        });
    ];

Here we use a factory or helper class ``\Slick\Di\Definition`` witch will create the container definition
for you. The factory method expects a callable or clojure, as we set in this example.

**Object definition**

This is the most flexible and powerful of all definitions. Object definitions does what its name states for:
define an object creation.

See the following code:

.. code-block:: php

    <?php

    return [
        'myCoolObject' => \Slick\Di\Definition::object('MyNamespace\MyCoolClass')
            ->constructor(['name' => 'Foo'])
            ->property('isActive', true)
            ->method('startCode', ['some', 'code'])
    ];

You have all you need to instantiate, set properties and call methods of an object that will be created
when you call ``Slick\Di\Container::get()`` method.
The helper is very verbose:

- ``Slick\Di\Definition\Helper\ObjectDefinitionHelper::constructor()``: used to set the list of parameters
  you want to pass to the constructor on object creation.
- ``Slick\Di\Definition\Helper\ObjectDefinitionHelper::property()``: used to set property values after object creation
- ``Slick\Di\Definition\Helper\ObjectDefinitionHelper::method()``: used to call specific methods after object creation.

Its likely that you can use almost every object with this definition.


**Link definition**

This is avery helpful definition. If you already have a definition in place and you want to access it with
other name you only need to create a link.

Lets see:

.. code-block:: php

    <?php

    return [
        'myObject' => \Slick\Di\Definition::link('myCoolObject');
    ];

Now if you call Container::get("myObject") the same object of "myCoolObject" will be returned.

Using dependency container
--------------------------

Now that we have our definitions in place, lets use them. You will need to create a container and
pass the definitions to it.

Lets see a full example:

.. code-block:: php

    <?php

    return [
        'myValue' => 1024,
        'myObject' => \Slick\Di\Definition::link('myCoolObject'),
        'myCalculatedValue' => \Slick\Di\Definition::factory(function(){
            return new \DateTime("now", new \DateTimeZone('UTC'));
        }),
        'myCoolObject' => \Slick\Di\Definition::object('MyNamespace\MyCoolClass')
            ->constructor(['name' => 'Foo'])
            ->property('isActive', true)
            ->method('startCode', ['some', 'code'])
    ];

Lets assume that the above code is in a file called ``definition.php`` lets create the container:

.. code-block:: php

    <?php

    use SLick\Di\ContainerBuilder;

    $definitions = include('definition.php');
    $container = ContainerBuilder::build($definitions);

    // Now lets use our container to get a new myCoolObject
    $object = $container->get('myCoolObject);

Now we have a full working container fed with our definitions ready to be used.

Object scope
------------

When defining your dependencies you can se the scope of the object:

- **Singleton**: The object will be instantiated just once (the first time you call it) and then will be reused
  every time you call it from your container;
- **Prototype**: The object will be instanciated every time you call it from your container.

All the definitions you use have a default scope. See the table bellow fo details

+-----------+-------+------+---------+--------+
|           | Value | link | factory | object |
+-----------+-------+------+---------+--------+
| Singleton |   X   |      |    X    |        |
+-----------+-------+------+---------+--------+
| Prototype |       |      |         |    X   |
+-----------+-------+------+---------+--------+

The container has a static property that is used to store those values and is the container that
uses the scope to determine if a new object is to be created or not.

Nevertheless you can call ``Container::make()`` witch forces the scope to be prototype.

You cal also set the scope when you define your injection by using the method ``DefinitionHelper::scope()``
and passing the scope you want along with it. See the example bellow:

.. code-block:: php

    <?php

    return [
        'myCoolObject' => \Slick\Di\Definition::object('MyNamespace\MyCoolClass')
            ->constructor(['name' => 'Foo'])
            ->property('isActive', true)
            ->method('startCode', ['some', 'code'])
            ->scope(\Slick\Di\Definition\Scope::PROTOTYPE())
    ];

