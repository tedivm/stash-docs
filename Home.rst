===========================
Stash
===========================

Every Project Needs A Stash
===========================

Stash makes it easy to speed up your code by caching the results of expensive functions or code. Certain actions, like database queries or calls to external APIs, take a lot of time to run but tend to have the same results over short periods of time. This makes it much more efficient to store the results and call them back up later.

Features
========

* *Stores all PHP Datatypes*
    Stash can store all of the php native datatypes- integers, booleans, null, strings, arrays and objects that can be serialized.

* *Hierarchal Cache*
    Stored items can be nested, like the folders of a filesystem. This allows for related items to be groups together and erased when changed. Storing a user's basic information can be nested 'users/userId/info', allow all of the user's information to be removed quite easily.

* *Interchangeable Back Ends*
    Stash can use a number of different storage engines to persist cache items between requests. Current handlers include Filesystem, APC, Memcached, and Sqlite handlers.

* *Staggered Handlers*
    It occasionally makes sense to use multiple backends- for example, you may have a small amount of memory to allot but a large piece of filesystem, in which case using APC and the FileSystem handler together is an ideal solution. Stash allows you to do this using the special backend, MultiHandler, which can take an unlimited number of handlers.

* *Stampede Protection*
    When a particularly expensive to generate item misses it can cause a chain reaction- the system slows down as it generates the cache, but the longer it takes the more processes that miss and start regenerating it. Stash gives the developer the ability to limit cache regeneration to a single process, as well as an assortment of ways to handle misses.

* *Regenerates Before Expiration*
    Stash gives developers the option to regenerate a cached item before it misses, making sure that up to data is always available while limiting expensive code to running one instance at a time.

* *Automatic Memoization*
    Even though cache handlers are designed to be fast  it is still possible to further increase performance by keeping a copy of previous cache hits in the script's local memory. This technique allows the cache to run even without a backend, allowing data that is picked up multiple times in a single request to be cached even without a persistence layer behind it.

* *Distributed Cache Misses*
    In order to reduce sudden spikes on a system Stash alters the expiration times by lowering the cache age a random amount, thus distributing the cache hits over a period of time.

* *Optimized Data Encoding*
    Care is taken to use the fastest possible encoding and decoding functions when storing data, with the preference being to store things in their native data type. Serialization is reserved only for objects or deep multidimensional arrays where breaking down each component individually would be too long.

* *Unit Tested*
    Every handler, class and wrapper is extensively tested. This includes using a variety of datatypes and ranges in order to ensure that the data you put in is the data you get it.

Installing
==========

Stash can be downloaded from `Github <https://github.com/tedivm/Stash>`_ or through `tedivm's pear channel <http://pear.tedivm.com/>`_. 

Example
=======

Stash functions as a key-value store: you place things into the cache using a key and you retrieve them using the same key. 

.. code-block:: php

    <?php
    $stash = $pool->getCache('fruit');
    $stash->store('apple');

    var_dump($stash->get());
    // string(6) "apples"

This works between requests as well.

.. code-block:: php

    <?php
    // First Request
    $stash = $pool->getCache('fruit');
    $stash->store('apple');

    // Second Request
    $stash = $pool->getCache('fruit');
    var_dump($stash->get());
    // string(6) "apples"

Putting this together with the rest of Stash allows for a simple yet flexible way to speed up scripts by reusing data.

.. code-block:: php

    <?php
    function getUserInfo($userId)
    {
        $pool = $this->cachePool;   

        // Get a Stash object from the cache pool.
        $stash = $pool->getCache('user', $userId, 'info');

        // Get the date from it, if any happens to be there.
        $userInfo = $stash->get();

        // Check to see if the cache missed, which could mean that it either didn't exist or was stale.
        if($stash->isMiss())
        {
            // Run the relatively expensive code.
            $userInfo = loadUserInfoFromDatabase($userId);

            // Store the expensive code so the next time it doesn't miss.
            $stash->store($userInfo);
        }

        return $userInfo;
    }

    function saveUserInfo($userId, $infoArray)
    {
        // Save the data- dumped behind a function just for the example.
        saveDataToDatabase($userId, $infoArray);

        // Clear out the now invalid data from the cache.
        $this->cachePool->clear('user', $userId, 'info');
    }

For an in-depth look at using Stash take a look at :doc:`Usage <Usage>` and :doc:`Handlers <Handlers>`.

License
=======

Stash is licensed under the New BSD License. This means you are free to use it in any of your projects, proprietary or open source. While you aren't obligated to contribute back, any bug fixes or enhancements are appreciated -- besides, getting your code into the main branch is so much easier than maintaining your own fork.
