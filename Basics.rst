.. _basics:

===========
Basic Usage
===========

Concepts
========

Pools and Items
---------------

The Pool class represents the caching system as a whole, while the Item class represents the pieces of data that are
stored in it. Item classes get pulled out of the Pool when they are needed (in other words, Pools are Item Factories).


Drivers
-------

Drivers allow Stash to interact with different caching systems by abstracting away the system specific APIs.

For the most part Drivers are invisible. Once they're created and injected into the Pool they can be forgotten about.
If you're using Stash with a framework, such as the Symfony Stash Bundle, you can ignore drivers altogether.


Keys
----

Keys are unique strings that map to data in the caching system. Each Item is associated with a Key. At the simplest
level Keys can be looked at as alphanumeric strings with a one to one mapping. This works like most traditional caching
systems, with each key mapping to a single piece of data in the caching system.

Stash also has a special kind of grouping system that allow cache items to be nested by adding a slash to Keys (the same
syntax as identifying folders on a filesystem). These slashes tells Stash where the nesting points are. Nesting allows
developers to organize items just like they would files and folders on a computer. This makes clearing groups of items
in the cache as simple as clearing their parent node, just like deleting a directory would erase all the files underneath.

A project that had different models, each identified by an id and a type, might have it's keys for those models start
with models/type/id, with individual pieces of data stored in keys inside of those. If the user "bob" had the id "32",
the path to his data in the cache would be "models/users/32".


Namespaces
----------

Namespaces provide a simple way of grouping cache data together and isolating it from other data in the system.
Namespaces are set on the Pool, with each subsequent Item retrieved from the Pool being a member of that Namespace. Two
Items with the same key but different Namespaces will refer to different data, and actions performed on a Pool with a
set Namespace only affects that Namespace.


Examples
========


Creating Items
--------------

Creating an Item with Stash is simple-

.. code-block:: php

    <?php
    $pool = new Stash\Pool();

    // Set the "key", which is the path the Stash object points to.
    $item = $pool->getItem('path/to/data');

This will create a Pool using the default "Ephemeral" driver. Data will be cached for the lifetime of the request, but
will not be available to future requests. In order to cache data across requests a drivers is needed.

.. code-block:: php

    <?php
    // Create Driver with default options
    $driver = new Stash\Driver\FileSystem();
    $driver->setOptions(array());

    // Inject the driver into a new Pool object.
    $pool = new Stash\Pool($driver);

    // New Items will get and store their data using the same Driver.
    $item = $pool->getItem('path/to/data');

Each driver object can be used by many different Pool objects, and each of those Pools will then have access to the
same data.

Items can be retrieved from the Pool class individually or in groups.

.. code-block:: php

    <?php
    // Create Driver with default options
    $driver = new Stash\Driver\FileSystem();

    // Create pool and inject driver
    $pool = new Stash\Pool($driver);

    // Retrieve a single cache item
    $item = $pool->getItem('path/to/data');

    // Retrieve an iterator containing multiple cache items
    $items = $pool->getItemIterator('path/to/data', 'path/to/more/data');


Identifying Items
-----------------

The examples so far show how to get Items using a Key represented as a string. It is also possible to represent a Key
as an array or a series of function arguments. Each of these methods will retrieve the same Item, this is just some
syntactic sugar developers can use.

.. code-block:: php

    <?php
    // Pass the key as a string
    $item = $pool->getItem('models/users/' . $id . '/info');

    // Pass the key as an array
    $item = $pool->getItem(array('models', 'users', $id, 'info'));

    // Pass the key as a series of arguments
    $item = $pool->getItem('models', 'users', $id, 'info');


Caching Data
------------

Storing and Retrieving Data is done through the Item class. Four functions do the bulk of the work:

* *get()* - Returns data that was previously stored, or null if nothing stored. (Since it is possible to store null
  values it is very important not to rely on a null return to check for a cache miss.)

* *isMiss()* - Returns true if no data is stored or the data is stale; returns false if fresh data is present.

* *lock()* - This is used to let other processes know that this process is generating new data.

* *set($data, $ttl = null)* - Stores the specified data for use by other requests.

Using these four functions, you can create simple cache blocks -- pieces of code where you fetch data, check to see if
it's fresh, and then regenerate and store the data if it was stale or absent.

.. code-block:: php

    <?php

    // Get a cache item.
	$item = $pool->getItem('path/to/item');

    // Attempt to get the data
    $data = $item->get();

    // Check to see if the data was a miss.
    if($item->isMiss())
    {
        // Let other processes know that this one is rebuilding the data.
        $item->lock();

        // Run intensive code
        $data = codeThatTakesALongTime();

        // Store the expensive to generate data.
        $item->set($data);
    }

    // Continue as normal.
    useDataForStuff($data);


The *set* function can take an optional "expiration" argument, which allows developers to set either a TTL in seconds
or an explicit expiration using a DateTime object.

.. code-block:: php

    <?php

    // Get cache item.
	$item = $pool->getItem('path/to/item');

    // Using an age.
    $data = $pool->get();
    if($item->isMiss())
    {
        $data = expensiveFunction();
        // Cache expires in one hour.
        $item->set($data, 3600);
    }


    // Using a DateTime.
    $data = $item->get();
    if($item->isMiss())
    {
        $data = expensiveFunction();

        // Cache expires January 21, 2015.
        $expiration = new DateTime('2015-01-21');
        $item->set($data, $expiration);
    }


.. NOTE::
    The expiration time only sets a maximum time the Item can be considered fresh, not a minimum. Items can be
    invalidated from the cache before their expiration time for a number of reasons. Stash will also attempt to
    distribute cache misses to normalize system load. At no point will an Item be considered fresh after the expiration
    or ttl is reached.


Clearing Data
-------------

Clearing data is just as simple as getting it. Note that clearing a key will clear that key *and any keys beneath it in
the hierarchy.*

.. code-block:: php

    <?php
    // Clearing a key.
    $item = $pool->getItem('path/to/data/specific/123')
    $item->clear();

    // Clearing a key with subkeys
    $item = $pool->getItem('path/to/data')
    $item->clear(); // clears 'path/to/data/*' as well as 'path/to/data'


Emptying the Entire Cache
-------------------------

The Pool class can also empty the entire cache:

.. code-block:: php

    <?php
    $pool->flush();


Running Maintenance
-------------------

Some caching systems require maintenance actions to occur. This can include things such as removing stale data or
reindexing for improved performance.

.. code-block:: php

    <?php
    $pool->purge();

.. NOTE::
    Depending on the size of the cache and the specific drivers in use this can take some time, so it is best called as
    part of a separate maintenance task or as part of a cron job.

