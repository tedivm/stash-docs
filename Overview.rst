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

For most projects Composer is the ideal method of installation. Stash is present
on `Packagist <https://packagist.org/packages/tedivm/stash>`_, making
installation as easy as adding a directive to your project's composer.json file.

The simpliest way to use Composer is to include the latest version of Stash.

.. code-block:: none

      "require": {
        "tedivm/stash": "*"
      }


The downside to this is that API changes can occur which will break the project.
Composer allows specific versions or lines of development to be used. This
example, which is the preferred method, installs the latest version of the v0.10
line of development, allowing for the latest enhancements but without the need
to worry about backwards compatibility breaking changes occurring.

.. code-block:: none

      "require": {
        "tedivm/stash": "0.12.*"
      }

For project looking to take advantage of the latest features it's also possible
to install directly against the main development line. This is ideal for
testing, but should be limited to non-production code.

.. code-block:: none

      "require": {
        "tedivm/stash": "dev-master"
      }

For more details on how to use composer with your projects, visit the composer
website at http://getcomposer.org/.


Direct Download
---------------

Releases of Stash are available for direct download on `Github
<https://github.com/tedivm/Stash/releases>`_.


License
=======

Stash is licensed under the New BSD License. This means you are free to use it
in any of your projects, proprietary or open source. While you aren't obligated
to contribute back, any bug fixes or enhancements are appreciated -- besides,
getting your code into the main branch is so much easier than maintaining your
own fork.