.. Dependency injector container documentation

Dependency Injection Container
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Dependency injection and dependency injection containers are tow different things. Dependency injection is a way
of doing better code as you can read in this Fabien Pontencier's
`article about dependency injection <http://fabien.potencier.org/article/11/what-is-dependency-injection>`_.
In other hand *Dependency Injection Container* is a tool that will help you injection dependencies.

This component for Slick is a very simple library where you can define dependencies and have them injected
in your object as you call for them.

**Installation**

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

A callable definition is used when you want to calculate the value when you call the container for that
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