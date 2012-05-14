===========================
Basic Usage
===========================

Stash is a simple to use library with powerful features.

Identifying Items Using Keys
==================================

Stash identifies items in the cache pool using keys, which are just simple strings. Stash has a special kind of grouping system that allow cache items to be nested, similar to how folders are nested in filesystems, by adding a slash to Keys. The slash tells the Implementing Library where the nesting points are. If no nesting is used, Stacks behave exactly like the standard Cache interfaces. Nesting allows developers to organize items just like they would files and folders on a computer. This makes clearing groups of items in the cache as simple as clearing their parent node, just like deleting a directory would erase all the files underneath.

A project that had different models, each identified by an id and a type, might have it's keys for those models start with models/type/id, with individual pieces of data stored in keys inside of those. If the user "bob" had the id "32", the path to his data in the cache would be "models/users/32".

Stash accepts keys in two forms: as a slash-delimited string, or as a series of arguments representing each level of nesting. Both work in exactly the same way.

.. code-block:: php

    <?php
    // Pass the key as a string
    $stash = $pool->getCache('models/users/' . $id . '/info');

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

    // Get cache item.
	$stash = $pool->getCache('path/to/item');
	
    // Attempt to "get"
    $data = $stash->get();

    // Check to see if the data was a miss.
    if($stash->isMiss())
    {
        // Run intensive code
        $data = codeThatTakesALongTime();

        // Store data.
        $stash->set($data);
    }

    // Continue as normal.
    return $data;

The *store* function can take the expiration as an additional argument. This expiration can be a time, in seconds, that the cache should live or it can be a DateTime object that represents the time the cached item should expire. (This argument can be negative, which will result in an immediately stale cache.) 

.. code-block:: php

    <?php

    // Get cache item.
	$stash = $pool->getCache('path/to/item');

    // Using an age.
    $data = $stash->get();
    if($stash->isMiss())
    {
        $data = expensiveFunction();
        // Cache expires in one hour.
        $stash->set($data, 3600);
    }


    // Using a DateTime.
    $data = $stash->get();
    if($stash->isMiss())
    {
        $data = expensiveFunction();

        // Cache expires January 21, 2012.
        $expiration = new DateTime('2012-01-21');
        $stash->set($data, $expiration);
    }

The expiration sets the *maximum* time a cached object can remain fresh. In order to distribute cache misses, the Stash system tries to vary the expiration time for items by shortening a random amount; some handlers may also have size restrictions or other criteria for removing items early, and items can be cleared manually before they expire. Items will never be reported as fresh *after* the expiration time passes, however.


Clearing Data
=============

Clearing data is just as simple as getting it. As with the *get* and *store* functions, the *clear* function takes a set key - if one isn't set then the entire cache is cleared. Note that clearing a key will clear that key *and any keys beneath it in the hierarchy.*

.. code-block:: php

    <?php
    // Clearing a key.
    $stash = $pool->getCache('path/to/data/specific/123')
    $stash->clear();

    // Clearing a key with subkeys
    $stash = $pool->getCache('path/to/data/general') // clears 'path/to/data/*'
    $stash->clear();

    // Clearing everything.
    $pool->purge();

The Pool class can also clear the entire cache:

.. code-block:: php

    <?php
    $pool->flush();


Purging Data
============

The *purge* function removes stale data from the cache backends while leaving current data intact. Depending on the size of the cache and the specific handlers in use this can take some time, so it is best called as part of a separate maintenance task or as part of a cron job. 

.. code-block:: php

    <?php
    $pool->purge();