Autoloading
===========

The Stash library conforms to the PSR-0 autoloading standard. If your project doesn't already use a PSR-0 compliant autoloader, you can simply include the `autoload.php` file at the root of the project to load Stash classes.

All of Stash's classes live in the Stash namespace. 

Creating Stash Objects
======================

Creating a basic Stash object is simple:

.. code-block:: php

    <?php
    $stash = new Stash\Cache();

    // Set the "key", which is the path the Stash object points to.
    $stash->setupKey('path/to/data');

This will create a cache object with no cross request storage, meaning the data will only be cached for the lifetime of that one script or request. In order to store cache results across requests, we need a handler. Each handler interfaces with a specific form of persistent storage.


.. code-block:: php

    <?php
    // Create Handler with default options
    $stashFileSystem = new Stash\Handler\FileSystem();

    // Create the actual cache object, injecting the backend
    $stash = new Stash\Cache($stashFileSystem);

    // Set the "key", which is the path the Stash object points to. This will be discussed in depth later,
    // but for now just know it's an identifier.
    $stash->setupKey('path/to/data');

Each handler object can be used by many different cache objects, so that any initial setup and overhead (database connections, file handlers, etc.) can be done only once per request. In order to simplify this process, the Pool class automates the process of handler creation to ensure that all cache objects use the same handlers.

.. code-block:: php

    <?php
    // Create Handler with default options
    $stashFileSystem = new Stash\Handler\FileSystem();

    // Create pool and inject handler
    $pool = new Stash\Pool();
    $pool->setHandler($stashFileSystem);

    // Retrieve a single cache item
    $pool->getCache('path/to/data');

    // Retrieve an iterator containing multiple cache items
    $pool->getCacheIterator('path/to/data', 'path/to/more/data');

Identifying Stored Data Using Keys
==================================

Stash identifies items in the cache pool using keys. Keys are simple: a set of strings, delimited by the '/' character.

The best way to think about keys is to think of them like a filesystem. Filesystems have different folders that contain more folders and files. Folders can be nested to virtually unlimited levels, allowing files to be organized according to various criteria. Stash uses the same principal- different nodes can contain both data and more nodes, allowing developers to group data together just like they would files. This makes clearing groups of items in the cache as simple as clearing their parent node, just like deleting a directory would erase all the files underneath.

A project that had different models, each identified by an id and a type, might have it's keys for those models start with models/type/id, with individual pieces of data stored in keys inside of those. If the user "bob" had the id "32", the path to his data in the cache would be "models/users/32".

Stash methods that accept keys can accept them in two forms: as a slash-delimited string, or as a series of arguments.

.. code-block:: php

    <?php
    // Pass the key as a string
    $stash = $pool->getCache('models/users/32/info');

    // Pass the key as a series of arguments
    $stash = $pool->getCache('models', 'users', $id, 'info');

Storing and Retrieving Data
===========================

Storing data in Stash (and retrieving it in future requests) is easy. Three functions do the bulk of the work: 

* *get()* - Returns data that was previously stored, or null if nothing stored. (Since it is possible to store null values it is very important not to rely on a null return to check for a cache miss.)
* *isMiss()* - Returns true if no data is stored or the data is stale; returns false if fresh data is present.
* *store($data, $expiration = null)* - Stores the specified data in the handler's persistent storage.

Using these three functions, you can create simple cache blocks -- pieces of code where you fetch data, check to see if it's fresh, and then regenerate and store the data if it was stale or absent.

.. code-block:: php

    <?php
    // Attempt to "get"
    $data = $stash->get();

    // Check to see if the data was a miss.
    if($stash->isMiss())
    {
        // Run intensive code
        $data = codeThatTakesALongTime();

        // Store data.
        $stash->store($data);
    }

    // Continue as normal.
    return $data;

The *store* function can take the expiration as an additional argument. This expiration can be a time, in seconds, that the cache should live or it can be a DateTime object that represents the time the cached item should expire. (This argument can be negative, which will result in an immediately stale cache.) 

.. code-block:: php

    <?php
    // Using an age.
    $data = $stash->get();
    if($stash->isMiss())
    {
        $data = expensiveFunction();
        // Cache expires in one hour.
        $stash->store($data, 3600);
    }


    // Using a DateTime.
    $data = $stash->get();
    if($stash->isMiss())
    {
        $data = expensiveFunction();

        // Cache expires January 21, 2012.
        $expiration = new DateTime('2012-01-21');
        $stash->store($data, $expiration);
    }

The expiration sets the *maximum* time a cached object can remain fresh. In order to distribute cache misses, the Stash system tries to vary the expiration time for items by shortening a random amount; some handlers may also have size restrictions or other criteria for removing items early, and items can be cleared manually before they expire. Items will never be reported as fresh *after* the expiration time passes, however.

Stampede Protection
===================

Sometimes, when a cache item expires, multiple requests might come in for that item before it can be regenerated. If the process of generating it is very slow or expensive, these requests might stack up, each slowing down the system enough that previous requests can't complete -- this is a cache stampede. Stash has a stampede prevention function that's fairly easy to use:

.. code-block:: php

    <?php
    // Get the data from the cache using the "STASH_SP_OLD" technique for dealing with stampedes
    $userInfo = $stash->get(Stash\Cache::STASH_SP_OLD);

    // Check to see if the cache missed, which could mean that it either didn't exist or was stale.
    if($stash->isMiss())
    {
        // Mark this instance as the one regenerating the cache. Because our protection method is
        // STASH_SP_OLD other Stash instances will use the old value and count it as a hit.
        $stash->lock();

        // Run the relatively expensive code.
        $userInfo = loadUserInfoFromDatabase($id);

        // Store the expensive code so the next time it doesn't miss. The store function marks the
        // stampede as over for now, so other Stash items will begin working as normal.
        $stash->store($userInfo);
    }

Invalidation Methods
====================

Stash's stampede protection gives developers multiple ways to deal with stale data. Old values can be reused, new values set, or the cache can even be refreshed before it gets stale. Different methods can be set by passing the appropriate constant to Stash's "get" function.

STASH_SP_NONE
-------------

By default Stash simply returns true for the "isMiss" function whenever the cache is invalid, meaning multiple cache misses can occur at once and stampede protection is not enabled. While not needed, this method can be explicitly set.

.. code-block:: php

    <?php
    // preserves backward compatibility.
    $stash->get();

    // recommended if this method is explicitly wanted as the default value may change in the future.
    $stash->get(STASH_SP_NONE);

    // returns false if the item is missing or expired, no exceptions.
    $stash->isMiss();

STASH_SP_PRECOMPUTE
-------------------

The personal favorite method of the Stash developers, this method causes Stash to recalculate the cached item _before_ it misses.

When this method is used Stash->get takes one additional argument, the amount of time (in seconds) before the expiration when it should regenerate the cache.

.. code-block:: php

    <?php
    // five minutes before the cache expires one instance will return a miss, causing the cache to regenerate.
    $stash->get(STASH_SP_PRECOMPUTE, 300);

STASH_SP_OLD
------------

When this method is enabled and a different instance has called the lock function, Stash will return the existing value in the cache even if it is stale.

.. code-block:: php

    <?php
    $stash->get(STASH_SP_OLD);

    // return false if another Stash instance is rebuilding the cached item even though the returned item is stale
    $stash->isMiss();

STASH_SP_VALUE
--------------

When this method is enabled and a different instance has called the lock function Stash will return the supplied value.

This method takes one additional argument, the value to be returned while stampede protection is on.

.. code-block:: php

    <?php
    $stash->get(STASH_SP_VALUE, 'Return this if stampede protection stops a miss');

    // returns true only if the value is stale and no other processes have stated rebuilding the value.
    $stash->isMiss();

STASH_SP_SLEEP
--------------

When this method is enabled and a different instance has called the lock function Stash will sleep and attempt to load the value upon waking up. This is not a website friendly method, but is potentially useful for cli or long running scripts.

When this method is used Stash->get takes two additional arguments, the time (in microseconds) to sleep before reattempting to load the cache and the amount of times to try and reload it before giving up. The maximum amount of time spent sleeping is the product of these two numbers.

.. code-block:: php

    <?php
    // sleeps for .5 seconds, reattempts to load the cache,
    // then sleeps again for another .5 seconds before making it's last attempt
    $stash->get(STASH_SP_SLEEP, 500, 2);

Clearing Data
=============

Clearing data is just as simple as getting it. As with the *get* and *store* functions, the *clear* function takes a set key - if one isn't set then the entire cache is cleared. Note that clearing a key will clear that key *and any keys beneath it in the hierarchy.*

.. code-block:: php

    <?php
    // Clearing a key.
    $stash = new Stash\Cache($handler);
    $stash->setupKey('path/to/data/specific/123')
    $stash->clear();

    // Clearing a key with subkeys
    $stash = new Stash\Cache($handler);
    $stash->setupKey('path/to/data/general') // clears 'path/to/data/*'
    $stash->clear();

    // Clearing everything.
    $stash = new Stash($handler);
    $stash->clear();

The Pool class can also clear the entire cache:

.. code-block:: php

    <?php
    $pool->flush();


Purging Data
============

The *purge* function removes stale data from the cache backends while leaving current data intact. Depending on the size of the cache and the specific handlers in use this can take some time, so it is best called as part of a separate maintenance task or as part of a cron job. 

.. code-block:: php

    <?php
    $stashFileSystem = new Stash\Handler\FileSystem();

    // Purge the FileSystem
    $stash = new Stash\Cache($stashFileSystem);
    $stash->purge();

The Pool class can also purge the cache:

.. code-block:: php

    <?php
    $pool->purge();
