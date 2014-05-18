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

