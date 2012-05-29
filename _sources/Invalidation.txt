===========
Invalidation
===========


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

