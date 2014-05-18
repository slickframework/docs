.. Slick components land page

Components
==========

Slick is organized in components that can be used independently and can be installed using composer.
When you install the full slick package (``slick/slick`` in `packagist <https://packagist.org>`_) all Slick
components will be downloaded and installed.

In this section we will cover all Slick modules separately.

Installing a module
-------------------

Lets assume the you want to use the ``slick/template`` and ``slick/configuration`` components in your project.
Your project's ``composer.json`` will need to have the following entries:

.. code-block:: json

    {
        "require": {
            "slick/template": "*",
            "slick/configuration": "*"
        }
    }

After updating your ``composer.json`` file you will need to run the following command:

.. code-block:: bash

    $ composer update

This will install the components in your vendor directory.

Slick components
----------------

.. toctree::
    :maxdepth: 2

    cache