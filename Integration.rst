.. _integration:

===========
Integration
===========


Installation
------------

Stash is available as a direct download through `GitHub
<https://github.com/tedivm/Stash/releases>`_, through Pear via the `
pear.tedivm.com <http://pear.tedivm.com>`_ channel, or the preferred method of
using `Composer <http://getcomposer.org/>`_.

When using Composer it's important to remember that Stash follows semantic
versioning, but has still not reached it's 1.0.0 release. This means care should
be given when updating MINOR versions, but that PATCH versions will always be
backwards compatible. The recommended Composer requirement should look like-

    "tedivm/stash": "0.11.*"

Where 0.11 can be replaced with the version you've developed against.


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