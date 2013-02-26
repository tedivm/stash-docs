.. _setup:

===========================
Getting Started
===========================

Composer
========

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
        "tedivm/stash": "0.10.*"
      }

For project looking to take advantage of the latest features it's also possible
to install directly against the main development line. This is ideal for
testing, but should be limited to non-production code.

.. code-block:: none

      "require": {
        "tedivm/stash": "dev-master"
      }

For more details on how to use composer with your projects, vist the composer
website at http://getcomposer.org/.


Github
======

Stash exists on `Github <https://github.com/tedivm/Stash>`_, where anyone can
contribute, fork, clone and download Stash. As Github is the primary location
where development occurs, it's the best place to check the latest changes and
enhancements.


Pear
====

Stash can be installed from the `tedivm pear channel <http://pear.tedivm.com>`_.

.. code-block:: none

    $ pear channel-discover pear.tedivm.com
    $ pear install tedivm/Stash

Further instructions can be found at
`pear.tedivm.com <http://pear.tedivm.com>`_.