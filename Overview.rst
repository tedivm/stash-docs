.. _overview:

========
Overview
========

Requirements
============

Stash requires a minimum version of PHP5.3 or HHVM 3.0 in order to run. Individual
drivers and caching systems will have their own requirements.


Installation
============

Composer
--------

Composer is the preferred method of installation. Stash is present
on `Packagist <https://packagist.org/packages/tedivm/stash>`_, making
installation as easy as adding a directive to your project's composer.json file.

Until Stash reaches a stable API with version 1.0 it is recommended that you
review changes before even Minor updates, although bug fixes will always be
backwards compatible.

.. code-block:: yaml

      "require": {
        "tedivm/stash": "0.12.*"
      }


For project looking to take advantage of the latest features it's also possible
to install directly against the main development line. This is ideal for
testing, but should be limited to non-production code.

.. code-block:: yaml

      "require": {
        "tedivm/stash": "dev-master"
      }

For more details on how to use composer with your projects, visit the composer
website at http://getcomposer.org/.


Direct Download
---------------

Versions of Stash are available as `Releases <https://github.com/tedivm/Stash/releases>`_.
on Github.


Autoloading
===========

If you installed using composer then you simply need to include it's autoloader.
This is the preferred method of loading Stash.

Stash confirms to PSR-0 and places it's code in the src folder, making it easy
to include in a custom autoloader. This means "Stash\Pool" can be found in
"src\Stash\Pool.php"..

If all else fails there is an autoloader that is packaged with Stash itself.
Just include "autoload.php".


License
=======

Stash is licensed under the New BSD License. This means you are free to use it
in any of your projects, proprietary or open source. While you aren't obligated
to contribute back, any bug fixes or enhancements are appreciated -- besides,
getting your code into the main branch is so much easier than maintaining your
own fork.