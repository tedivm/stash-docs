===========================
Basic Usage
===========================

Stash is a simple to use library with powerful features.

Autoloading
===========

The Stash library conforms to the PSR-0 autoloading standard. If your project doesn't already use a PSR-0 compliant autoloader, you can simply include the `autoload.php` file at the root of the project to load Stash classes.

All of Stash's classes live in the Stash namespace. 


Creating Stash Objects
======================

Creating a basic Stash object is simple:

.. code-block:: php

    <?php
    $stash = new Stash\Pool();

    // Set the "key", which is the path the Stash object points to.
    $item = $stash->getCache('path/to/data');

This will create a cache object with no cross request storage, meaning the data will only be cached for the lifetime of that one script or request. In order to store cache results across requests, we need a driver. Each driver interfaces with a specific form of persistent storage.


.. code-block:: php

    <?php
    // Create Driver with default options
    $stashFileSystem = new Stash\Driver\FileSystem();

    // Create the actual cache object, injecting the backend
    $stash = new Stash\Pool($stashFileSystem);

    // Set the "key", which is the path the Stash object points to. This will be discussed in depth later,
    // but for now just know it's an identifier.
    $item = $stash->getItem('path/to/data');

Each driver object can be used by many different cache objects, so that any initial setup and overhead (database connections, file drivers, etc.) can be done only once per request. In order to simplify this process, the Pool class automates the process of driver creation to ensure that all cache objects use the same drivers.

.. code-block:: php

    <?php
    // Create Driver with default options
    $stashFileSystem = new Stash\Driver\FileSystem();

    // Create pool and inject driver
    $pool = new Stash\Pool($stashFileSystem);

    // Retrieve a single cache item
    $item = $pool->getItem('path/to/data');

    // Retrieve an iterator containing multiple cache items
    $items = $pool->getItemIterator('path/to/data', 'path/to/more/data');


Identifying Items Using Keys
==================================

Stash identifies items in the cache pool using keys, which are just simple strings. Stash has a special kind of grouping system that allow cache items to be nested, similar to how folders are nested in filesystems, by adding a slash to Keys. The slash tells the Implementing Library where the nesting points are. If no nesting is used, Stacks behave exactly like the standard Cache interfaces. Nesting allows developers to organize items just like they would files and folders on a computer. This makes clearing groups of items in the cache as simple as clearing their parent node, just like deleting a directory would erase all the files underneath.

A project that had different models, each identified by an id and a type, might have it's keys for those models start with models/type/id, with individual pieces of data stored in keys inside of those. If the user "bob" had the id "32", the path to his data in the cache would be "models/users/32".

Stash accepts keys in two forms: as a slash-delimited string, or as a series of arguments representing each level of nesting. Both work in exactly the same way.

.. code-block:: php

    <?php
    // Pass the key as a string
    $stashItem = $pool->getItem('models/users/' . $id . '/info');

    // Pass the key as a series of arguments
    $stashItem = $pool->getItem('models', 'users', $id, 'info');

Storing and Retrieving Data
===========================

Storing data in Stash (and retrieving it in future requests) is easy. Three functions do the bulk of the work: 

* *get()* - Returns data that was previously stored, or null if nothing stored. (Since it is possible to store null values it is very important not to rely on a null return to check for a cache miss.)
* *isMiss()* - Returns true if no data is stored or the data is stale; returns false if fresh data is present.
* *store($data, $expiration = null)* - Stores the specified data in the driver's persistent storage.

Using these three functions, you can create simple cache blocks -- pieces of code where you fetch data, check to see if it's fresh, and then regenerate and store the data if it was stale or absent.

.. code-block:: php

    <?php

    // Get cache item.
	$stashItem = $pool->getItem('path/to/item');
	
    // Attempt to "get"
    $data = $stashItem->get();

    // Check to see if the data was a miss.
    if($stashItem->isMiss())
    {
        // Run intensive code
        $data = codeThatTakesALongTime();

        // Store data.
        $stashItem->set($data);
    }

    // Continue as normal.
    return $data;

The *store* function can take the expiration as an additional argument. This expiration can be a time, in seconds, that the cache should live or it can be a DateTime object that represents the time the cached item should expire. (This argument can be negative, which will result in an immediately stale cache.) 

.. code-block:: php

    <?php

    // Get cache item.
	$stashItem = $pool->getItem('path/to/item');

    // Using an age.
    $data = $stash->get();
    if($stashItem->isMiss())
    {
        $data = expensiveFunction();
        // Cache expires in one hour.
        $stashItem->set($data, 3600);
    }


    // Using a DateTime.
    $data = $stashItem->get();
    if($stashItem->isMiss())
    {
        $data = expensiveFunction();

        // Cache expires January 21, 2012.
        $expiration = new DateTime('2012-01-21');
        $stashItem->set($data, $expiration);
    }

The expiration sets the *maximum* time a cached object can remain fresh. In order to distribute cache misses, the Stash system tries to vary the expiration time for items by shortening a random amount; some drivers may also have size restrictions or other criteria for removing items early, and items can be cleared manually before they expire. Items will never be reported as fresh *after* the expiration time passes, however.


Clearing Data
=============

Clearing data is just as simple as getting it. As with the *get* and *store* functions, the *clear* function takes a set key - if one isn't set then the entire cache is cleared. Note that clearing a key will clear that key *and any keys beneath it in the hierarchy.*

.. code-block:: php

    <?php
    // Clearing a key.
    $stashItem = $pool->getCache('path/to/data/specific/123')
    $stashItem->clear();

    // Clearing a key with subkeys
    $stashItem = $pool->getCache('path/to/data/general') // clears 'path/to/data/*'
    $stashItem->clear();

The Pool class can also empty the entire cache:

.. code-block:: php

    <?php
    $pool->flush();


Purging Data
============

The *purge* function removes stale data from the cache backends while leaving current data intact. Depending on the size of the cache and the specific drivers in use this can take some time, so it is best called as part of a separate maintenance task or as part of a cron job. 

.. code-block:: php

    <?php
    $pool->purge();