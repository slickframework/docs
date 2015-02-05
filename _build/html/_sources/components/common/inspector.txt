.. Slick common component - inspector

Class inspector
===============

.. contents:: Table of Contents
    :depth: 3

Inspect and Annotations
-----------------------

The ``Slick\Common`` component also has an ``Inspector`` utility class that can hel
you with class or object inspection and ``Annotations`` parsing. It is also possible
to define custom annotations to use with your classes. The base class uses the
``Slick\Common\Inspector`` to determine the visibility and access to its properties.

Inspect classes or objects
""""""""""""""""""""""""""

``Slick\Common\Inspector::__construct()`` needs only an object or a full qualified class
name to retrieve metadata from that class. Lets consider the following class: ::

    namespace Models;

    use Slick\Common\Base;

    class User extends Base
    {
        /**
         * @readwrite
         * @column type=text, size=tiny
         * @var string
         */
        protected $_name;

        /**
         * @readwrite
         * @var int
         */
        protected $_age;

        /**
         * @readwrite
         * @var string
         */
        protected $_active;

        /**
         * Check if age is under 18 years old
         *
         * @return boolean
         */
        public function isTeenager()
        {
            return $this->_age < 18;
        }
    }

Given the above class definition lets inspect its metadata with ``Slick\Common\Inspector``: ::

    use Slick\Common\Inspector;

    $inspector = new Inspector('\Models\User');

    // Getting the list of properties
    $propertyList = $inspector->getClassProperties();   // -> ['_name', '_age', '_active']

    // Getting the list of methods
    $methodList = $Inspector->getClassMethods();        // -> ['isTeenager']

    $inspector->hasMethod('getName');                   // FALSE
    $inspector->hasProperty('_age');                    // TRUE

Those are the simplest methods of ``Slick\Common\Inspector``. Just retrieve the property
and method names.

Retrieve the annotations
""""""""""""""""""""""""

Now we have the knowledge of what properties and methods the class have lets retrieve
the DockBlock annotations present in the code. ::

    $inspector = new Inspector('Models\User');

    $annotations = $inspector->getPropertyAnnotations('_name');

    // Annotations is an ArrayObject that contains all defined annotations of a given
    // comment block. In this case the comment block for _name property.

    if ($annotations->hasAnnotation('column') {
        $columnAnnotation = $annotations->getAnnotation('column');

        $columnAnnotation->getName();           //-> returns 'column'
        $columnAnnotation->getValue();          //-> returns TRUE
        $columnAnnotation->getParameter('size') //-> It will have 'tiny'
    {

    // Check another annotation
    // Here you can optionally use the @ as this may seem clean in your code
    if ($annotations->hasAnnotation('@var') {
        $var = $annotations->getAnnotation('@var');

        $vat->getValue();   // This will return the 'string' value
        // if the first parameter of the annotations is a string it will be used as
        // the value for that annotation.
    }

There hare tree methods that you can use to retrieve the annotations present in comments:

Slick\\Common\\Inspector::getClassAnnotations()
...............................................

    Retrieves a list of annotations from the class comment block

.. code-block:: php

    public Inspector\AnnotationsList Inspector::getClassAnnotations();

+--------------------------------------------+------------------------------+
| Return                                     | Description                  |
+============================================+==============================+
| ``Slick\Common\Inspector\AnnotationsList`` | A list of Annotation objects |
+--------------------------------------------+------------------------------+

Slick\\Common\\Inspector::getPropertyAnnotations()
..................................................

    Retrieve a list of annotations from a given property

.. code-block:: php

    public Inspector\AnnotationsList Inspector::getPropertyAnnotations(string $property);

+---------------+------------+------------------------------+
| Parameters    | Type       | Description                  |
+===============+============+==============================+
+ ``$property`` | ``string`` | The property name to inspect |
+---------------+------------+------------------------------+

+--------------------------------------------+------------------------------+
| Return                                     | Description                  |
+============================================+==============================+
| ``Slick\Common\Inspector\AnnotationsList`` | A list of Annotation objects |
+--------------------------------------------+------------------------------+

Slick\\Common\\Inspector::getMethodAnnotations()
................................................

    Retrieves a list of annotations form a given method.

.. code-block:: php

    public Inspector\AnnotationsList Inspector::getMethodAnnotations(string $method);

+-------------+------------+----------------------------+
| Parameters  | Type       | Description                |
+=============+============+============================+
+ ``$method`` | ``string`` | The method name to inspect |
+-------------+------------+----------------------------+

+--------------------------------------------+------------------------------+
| Return                                     | Description                  |
+============================================+==============================+
| ``Slick\Common\Inspector\AnnotationsList`` | A list of Annotation objects |
+--------------------------------------------+------------------------------+

Working with annotations list
"""""""""""""""""""""""""""""

The annotation list is a collection of ``Slick\Common\Inspector\Annotation`` objects that
extends from ``ArrayObject``. Therefore it can be used as an array or be iterated in
``foreach`` loops.

Slick\\Common\\Inspector\\AnnotationsList::hasAnnotation()
..........................................................

    Checks if current list contains an annotation with the provided name

.. code-block:: php

    public bool Inspector\AnnotationsList::hasAnnotation(string $name);


+------------+------------+---------------------------------+
| Parameters | Type       | Description                     |
+============+============+=================================+
+ ``$name``  | ``string`` | Annotation name. Ex.: '@column' |
+------------+------------+---------------------------------+

+----------+-------------------------------------------------------------------------------------+
| Return   | Description                                                                         |
+==========+=====================================================================================+
| ``bool`` | True if current list contains an annotation with the provided name, False otherwise |
+----------+-------------------------------------------------------------------------------------+

Slick\\Common\\Inspector\\AnnotationsList::getAnnotation()
..........................................................

    Returns the annotation with the provided name.

.. code-block:: php

    public Inspector\AnnotationInterface Inspector\AnnotationsList::getAnnotation(string $name);

+------------+------------+---------------------------------+
| Parameters | Type       | Description                     |
+============+============+=================================+
+ ``$name``  | ``string`` | Annotation name. Ex.: '@column' |
+------------+------------+---------------------------------+

+---------------------------------------+----------------------+
| Return                                | Description          |
+=======================================+======================+
| ``Slick\Common\Inspector\Annotation`` | An annotation object |
+---------------------------------------+----------------------+

.. warning::

    If you try to get an annotation that is not in the annotations list an
    ``Slick\Common\Exception\InvalidArgumentException`` will be thrown. It is very
    important that you test if the annotation exists before you retrieve it from
    the list.

Annotation object
"""""""""""""""""

Annotation object is a set of attributes that a given annotation has in the comment
DocBlock when it is parsed. The values set in the comment for the annotation can be
retrieved by their names with the methods described bellow.

Slick\\Common\\Inspector\\AnnotationInterface::getValue()
.........................................................

    Returns the value in the tag for single value annotations. Ex.: @before test

.. code-block:: php

    public mixed Inspector\AnnotationInterface::getValue();

+-----------+----------------------------------------+
| Return    | Description                            |
+===========+========================================+
| ``mixed`` | The parsed value of the annotation tag |
+-----------+----------------------------------------+

Slick\\Common\\Inspector\\AnnotationInterface::getParameter()
.............................................................

    Returns the value of the provided parameter name for multiple value annotations.
    Ex:. @after foo=3, bar=true.

.. code-block:: php

    public mixed Inspector\AnnotationInterface::getParameter(string $name);

+------------+------------+------------------------------------+
| Parameters | Type       | Description                        |
+============+============+====================================+
| ``$name``  | ``string`` | The parameter name                 |
+------------+------------+------------------------------------+

+-----------+----------------------------------------------+
| Return    | Description                                  |
+===========+==============================================+
| ``mixed`` | The parsed value of the annotation parameter |
+-----------+----------------------------------------------+

Annotations Parser
""""""""""""""""""

Annotations parse is a very simple yet versatile parser that can understand a wide range of
annotations and translate them into list, objects, numbers, etc.

Scalar values
.............

.. code-block:: php

    /**
     * Simple strings, integers and float numbers
     *
     * @annotation foo=bar, baz=14, size=2.3
     */

     $annotation->getParameter('size'); // will return 2.3 float number

Arrays and lists
................

.. code-block:: php

    /**
     * Arrays and lists
     *
     * @annotation foo=[one, two, three], array=[foo=[other, list]]
     */

     $annotation->getParameter('foo'); // will return ['one', 'two', 'three']
     $annotation->getParameter('array'); // will return ['foo' => ['other', 'list']]

Objects (json notation)
.......................

.. code-block:: php


    /**
     * Objects
     *
     * @annotation foo={"bar": [1, 2, 3]}
     */

     $annotation->getParameter('foo'); // will return {"bar" => [1, 2, 3]}

Booleans
........

.. code-block:: php

    /**
     * Booleans
     *
     * @annotation foo=true, bar=false
     */

     $annotation->getParameter('foo'); // will return TRUE

Defining your own annotations
"""""""""""""""""""""""""""""

It is possible to define you own annotations that will have a default set of parameters
and will help you when you don't set the parameters in the comment block and need a value
for those parameters.

Lets see an example: ::

    use Slick\Common\Inspector;
    use Slick\Common\Inspector\Annotation;

    // A class to inspect
    class Foo
    {

        /**
         * @custom test, foo=true
         */
         private $_name;

    }

    // Our custom annotation
    class CustomAnnotation extends Annotation
    {
        protected $_parameters = [
            'foo' => false,
            'bar' => true
        ];
    }


    // Register our new annotation
    Inspector::addAnnotationClass('custom', '\CustomAnnotation');

    $inspector = new Inspector('\Foo');

    $annotation = $inspector->getPropertyAnnotations('_name')->getAnnotation('@custom');

    $annotations->getParameter('foo');  // will return TRUE
    $annotations->getParameter('bar');  // will return TRUE
    $annotations->getValue();           // will return 'test' string

As you can see it is very simple to have an annotation that you can define all parameter
with default values and use them safely even if they are not present in the DocBlock.
You only need to add the class to the inspector class map once, in your application
bootstrap for example.