.. _invalidation:

============
Invalidation
============

Dog Pile Effect
===============

The common method for dealing with a cache miss is pretty straight forward- when the cache is checked for a value and it
is stale or not present the value is regenerated. The flaw to this approach is that it can mean many different requests
will see that cache miss at the same time and also attempt to regenerate the potentially expensive code. This in turn
can use up system resources and even cause a cache stampede, which is a type of cascading failure where the system is
placed under too much load to regenerate the cache which causes the load to further increase as more requests for the
data are made.

A good example of this is Javascript Minification. Minifying javascript is an expensive process, and for larger
libraries can take a few seconds to occur (lets say five for this example). If a server receiving 30 requests a second
for a javascript file that is hidden behind a minifier and there is a cache miss then there will be 30 requests the
first second, 60 requests generating it the second, 90 the third, and ultimately by the fifth second there will be 150
different requests regenerating the data. Unfortunately by this point the system load will also have risen considerably,
making that five second processing time rise to higher and higher numbers.


Stampede Protection
===================

Stash refers to it's methods for preventing the Dog Pile Effect as Stampede Protection. These features are all ways to
deal with stale data in such a way that only one request has to regenerate it. The default method is by triggering a
single cache miss in advance so that a single process will regenerate the data before the data actually expires. There
are other methods as well, such as reusing old values or simply disabling Stampede Protection altogether.

Defining which method to use done by passing an Invalidation constant to the Item->get() class when it retrieves it's
value. This tells the Item class how it should handle a miss, and what it should do while another class is processing
data.

The example below tells the Item class that it should switch from the default method of precomputing a value before it
expires and that it should instead wait until the expiration and continue using stale values while another process
regenerates the data.

.. code-block:: php

    <?php
	$item = $pool->getItem('Test');

    // Get the data from the cache using the "Invalidation::OLD" technique for
    // dealing with stampedes
    $userInfo = $item->get(Stash\Invalidation::OLD);

    // Check to see if the cache missed, which could mean that it either didn't
    // exist or was stale. If another process is regenerating this value and
    // there is a stored value available then this function will return a hit.
    if($item->isMiss())
    {
        // Mark this instance as the one regenerating the cache. Because our
        // protection method is Invalidation::OLD other Stash instances will
        // use the old value and count it as a hit.
        $item->lock();

        // Run the relatively expensive code.
        $userInfo = loadUserInfoFromDatabase($id);

        // Store the expensive code so the next time it doesn't miss. The store
        // function marks the stampede as over for now, so other Stash items
        // will begin working as normal.
        $item->set($userInfo);
    }

Invalidation Methods
====================

Stash's stampede protection gives developers multiple ways to deal with stale data. Old values can be reused, new values
set, or the cache can even be refreshed before it gets stale. Different methods can be set by passing the appropriate
constant to the Item::get function.


Precompute
----------

The default behavior of Stash, this method causes Stash to recalculate the cached item *before* it misses. This reduces
cache misses down to almost zero, as the only ones that occur are for new data or the arterial ones this creates.

When this method is used Item->get takes one additional argument, the amount of time (in seconds) before the expiration
when it should regenerate the cache. If this isn't pass a default is chosen of 10% of the Item's TTL

.. code-block:: php

    <?php
    // five minutes before the cache expires one instance will return a miss,
    // causing the cache to regenerate.
    $item->get(Invalidation::PRECOMPUTE, 300);


None
----

This removes stampede protection and provides no special behavior to limit multiple multiple cache misses. When a value
is stale isMiss will always return true. This is useful for code that has special behavior on misses or that does not
get called by a large number of users, such as by the Session Handler. Before Stash v0.12 this was the default behavior.

.. code-block:: php

    <?php
    $item->get(Invalidation::NONE);

    // returns false if the item is missing or expired, no exceptions.
    $item->isMiss();


Old
----

When this method is enabled and a different instance has called the lock function, Stash will return the existing value
in the cache even if it is stale.

.. code-block:: php

    <?php
    $item->get(Invalidation::OLD);

    // return false if another Item instance is rebuilding the cached item even
    // though the returned item is stale
    $item->isMiss();


Value
-----

When this method is enabled and a different instance has called the lock function Stash will return the supplied value.

This method takes one additional argument, the value to be returned while stampede protection is on.

.. code-block:: php

    <?php
    $item->get(Item::SP_VALUE, 'Use this value while regenerating cache.');

    // returns true only if the value is stale and no other processes have
    // stated rebuilding the value.
    $item->isMiss();


Sleep
-----

When this method is enabled and a different instance has called the lock function Stash will sleep and attempt to load
the value upon waking up. This is not a website friendly method, but is potentially useful for cli or long running
scripts.

When this method is used Stash->get takes two additional arguments, the time (in microseconds) to sleep before
reattempting to load the cache and the amount of times to try and reload it before giving up. The maximum amount of time
spent sleeping is the product of these two numbers.

.. code-block:: php

    <?php
    // sleeps for .5 seconds, reattempts to load the cache, then sleeps again
    // for another .5 seconds before making it's last attempt
    $item->get(Invalidation::SLEEP, 500, 2);
