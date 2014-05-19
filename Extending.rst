.. _extending:

=========
Extending
=========

Getting Started
===============

The process of extending Stash is pretty straight forward-

* Create the new class, either extending the existing one or implementing the Interface.
* Extend the AbstractTest and configure it to use the newly created class.
* Run the test suite and fix any bugs that appear.
* Add additional tests for any custom behavior that was added.


Interfaces
==========

In the Integration section it's recommended that developers typehint against the built in interfaces, and this allows
any code written that impliments get used by all of those developers. It also makes it clear to developers extending
Stash which changes are breaking expectations of users of the library. The interfaces are extensively commented, and
reading them directly is by far the best way to understand what is expected of the things that implement them.

When extending the Pool and Item classes it's highly recommended that the built in classes be utilized
and built off of, but any class that passes the test suite and impliments the relevant interfaces should be able to be
swapped in for the equivalent Stash class without problem. When building Drivers it is often easier to start with a
fresh slate and simply build out the DriverInterface.

Relevant Interfaces:

* *Stash\\Interfaces\\PoolInterface*
* *Stash\\Interfaces\\ItemInterface*
* *Stash\\Interfaces\\DriverInterface*


AbstractTests
=============

While Interfaces define an API contract, Tests make sure they're enforced. To make development easier there is an
AbstractTest for every Interface with a full suite of tests to ensure proper behavior. Using each suite is done by
creating the new test class and extending it off of the AbstractTest (rather than the typical
PHPUnit_Framework_TestCase), and then setting the appropriate class attribute to let the new test know which class they
should be testing.

* *Stash\\Tests\\AbstractPoolTest*- Set 'poolClass'.
* *Stash\\Tests\\AbstractItemTest*- Set 'itemClass'.
* *Stash\\Tests\\Driver\\AbstractDriverTest*- Set 'driverClass'.


Replacing The Item Class
========================

When extending the Pool it's easy to also change the Item class by overriding the "itemClass" protected variable with
the new Item class name.

To change the Item class without having to make a new Pool class as well the Pool->setItemClass() method can be used. It
has to be called for each new Pool instance, and the new Item class must implement the ItemInterface or an exception
will be thrown.