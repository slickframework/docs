.. Template module documentation

Template
~~~~~~~~

Template is a wrapper to the very powerful template engine for PHP
created by `Sensio Labs <http://sensiolabs.com>`_,
the `Twig template engine <http://twig.sensiolabs.org/>`_.

Using a template
""""""""""""""""

It is very simple to create a template engine a use it.

Lets look to an example:

.. code-block:: php

    <?php

    use Slick\Template;

    Template::addPath("./views");

    $template = new Template();
    $engine = $template->initialize();

    print $engine->parse("myView.twig")
        ->process(['var' => 'foo']);


In the code above we have set the path where our template files will live, initialize
an engine (if you do not set, the default is Twig) and print out the result of this
template processed with an array of dynamic data.

Setting multiple template paths
-------------------------------

It is possible to set multiple paths where the template engine will look for when you
parse a provided template file.

The template engine maintains a priority list with all the