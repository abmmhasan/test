.. _di.usage:

===================
Using the container
===================

Simply, initialize using,

.. code-block:: php
    :caption: Use either of these lines to initiate the Container
    :name: container.initiate.example
    $container = AbmmHasan\InterMix\container();
    $container = AbmmHasan\InterMix\Container::instance();
    $container = new AbmmHasan\InterMix\Container();

.. attention::

    By default,
    1. **Autowiring** is enabled
    2. **Attribute** resolution is disabled
    3. No default method set

The container have following options to play with as you see fit,

get() & has()
-------------

The container implements the
`PSR-11 <http://www.php-fig.org/psr/psr-11/>`__ standard. That means it
implements
```Psr\Container\ContainerInterface`` <https://github.com/php-fig/container/blob/master/src/ContainerInterface.php>`__:

.. code:: php

   namespace Psr\Container;

   interface ContainerInterface
   {
       public function get($id);
       public function has($id);
   }

set() & addDefinitions()
------------------------

You can set entries directly on the container using either of these:

.. code:: php
    :caption: add definition using set() method
    :name: set.example

    $container->set('foo', 'bar');
    $container->set('MyInterface', container('MyClass'));
    $container->set('myClosure', function() { /* ... */ });

.. code:: php
    :caption: add definition using addDefinitions() method
    :name: addDefinitions.example

    $container->addDefinitions([
        'foo' => 'bar',
        'MyInterface' => 'MyClass',
        'myClosure' => function() { /* ... */ }
    ]);

For details, see the :doc:`di.definitions`.

getReturn()
-----------

``get()`` & ``getReturn()`` does the same except, ``getReturn()`` prioritize method returns where ``get()`` prioritizes
class object.

call()
------

``get()`` & ``getReturn()`` both internally calls ``call()``. Differences are, ``get()`` & ``getReturn()`` results are
cached. In case of ``call()`` returns from method/closure are never cached (but class instances is cached as usual).

registerClass()
---------------

Normally, this method won't be needed unless you need to send in some extra parameter to the constructor.

.. code:: php
    :caption: you don't need registerClass() for this
    :name: registerClass.no-need.example

    class GithubProfile
    {
        public function __construct(ApiClient $client)
        ...
    }

.. code:: php
    :caption: but you will need here if the variable $user is not defined via set()/addDefinitions()
    :name: registerClass.required.example

    class GithubProfile
    {
        public function __construct(ApiClient $client, $user)
        ...
    }

    // define as below
    $container->registerClass('GithubProfile', [
        'user' => 'some value'
    ]);

registerClosure()
-----------------

Same as ``registerClass()`` but for Closure.

registerProperty(), registerMethod()
------------------------------------

While resolving through classes, container will look for any property value registered of that class (if **attribute** &
**property** resolutions is enabled) & will resolve it. During this if any custom property value is defined with
``registerProperty()`` it will resolve it as well.

.. code:: php
    :caption: register property by class
    :name: registerProperty.optional.example

    $container->registerProperty('GithubProfile', [
        'someProperty' => 'some value'
    ]);

Container will look for any method registered with ``registerMethod()`` & will resolve it. Even if it is not registered,
container still may resolve some method, check the container lifecycle for details.

.. code:: php
    :caption: register parameter in a method (also is default method to resolve for that class)
    :name: registerMethod.example

    $container->registerMethod('GithubProfile', 'aMethod', [
        'user' => 'some value'
    ]);

setOptions()
------------

Well, as you have seen above, the container provides lots of options. Obviously you can enable/disable them as your requirements.
Available options are,

- ``injection``: Enable/disable dependency injection (Enabled by default)
- ``methodAttributes``: Enable/disable attribute resolution on method
- ``propertyResolution``: Enable/disable property resolution
- ``propertyAttributes``: Enable/disable attribute resolution on property
- ``defaultMethod``: Set a default method to be called if method is not set already

.. attention::

    Defaults are; ``injection`` is enabled, rests are disabled. If ``injection`` is disabled rest of the options won't work.
    ``propertyAttributes`` also requires ``propertyResolution`` to be enabled.

.. note::

    When container scans through the classes, to resolve a method it follows below priority:
    - Method already provided, using ``call()``
    - Look for method, registered via ``registerMethod()``
    - Method provided via ``callOn`` constant
    - Method name found via ``defaultMethod``

split()
-------

Breakdown any recognizable formation to a recognizable callable format ``['class', 'method']`` or ``['closure']``.
Applicable formats are,

- ``class@method``
- ``class::method``
- ``closure()``
- ``['class', 'method']``
- ``['class']``

unset()
-------

Once container is created it can be chained/piped through (to add/edit method/property/options) till the process die.
But once **unset()** is called, no more chaining. Calling back will just simply initiate new container instance.

.. code:: php
    :caption: unset the internal instance
    :name: unset.example

    $container->unset();