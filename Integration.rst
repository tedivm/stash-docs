.. _integration:

===========
Integration
===========

Logging
-------

Stash supports logging through third party logging libraries, such as monolog.
It does this by following  the `PSR-3
LoggerAwareInterface <http://www.php-fig.org/psr/3/>`_. Caching events of all
varieties- from simple hit or miss numbers to driver errors and other exceptions-
can be exposed to analysis by injecting a compliant logging object into any Pool
instance.


Standards Compliance
--------------------

Stash complies with `PSR-0 <http://www.php-fig.org/psr/0/>`_.,
`PSR-1 <http://www.php-fig.org/psr/1/>`_., `PSR-2
<http://www.php-fig.org/psr/2/>`_. and `PSR-3 <http://www.php-fig.org/psr/3/>`_.
This makes integration with other projects easier, as it shares a common defined
code standard, follows naming scemes which allow for shared autoloaders, and
opens itself up to integration with other libraries.