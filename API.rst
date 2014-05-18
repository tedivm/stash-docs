.. _coreapi:

========
Core API
========

Drivers
=======

Stash works by storing values into various backend systems, like APC and Memcached, and retrieving them later. With the
exception their creation and setup, drivers don't have any "public" functions- they are used by the Pool and Item
classes themselves to interact with the underlying cache system.

The :ref:`Drivers` page contains a list of all drivers and their options.

setOptions
----------

*setOptions(array $options)*

Passes an array of options to the Driver. This can include things like server addresses or directories to use for cache
storage.


Pool
====

The Pool class represents the entire caching system and all of the items in it. Objects of this class are used to
retrieve Items from the cache, as well as to perform bulk operations on the system such as Purging or Clearing the
cache.


setDriver
---------

*setDriver($driver)*

Sets the driver for use by the caching system. This driver handles the direct interface with the caching backends,
keeping the system specific development abstracted out.


setLogger
---------

*setLogger($logger)*

Sets a \PSR\Log\LoggerInterface style logging client to enable the tracking of errors.


setNamespace
------------

Places the Pool inside of a "namespace". All Items inside a specific namespace should be completely segmented from all
other Items.


getNamespace
------------

Retrieves the current namespace, or false if one isn't set.


getItem
-------

*getItem($key)*

The getItem function takes in a key and returns an associated Item object. The structure of keys can be found on the
:ref:`basics` page.


getItemIterator
---------------

*getItemIterator(array $keys)*

The getItemIterator function takes in an array of keys and returns an Iterator object populated by Item objects for
those keys. The structure of keys can be found on the :ref:`basics` page.


flush
-----

*flush()*

The flush function completely empties all items associated with the Pool. After calling this every Item will be considered a miss and will have to be regenerated.


purge
-----

*purge()*

The Purge function allows drivers to perform basic maintenance tasks, such as removing stale or expired items from
storage. Not all drivers need this, as many interact with systems that handle that automatically.

It's important that this function is not called from inside a normal request, as the maintenance tasks this allows can
occasionally take some time.


Item
=====

The Item class represents specific pieces of data in the caching system. Item
objects are created by the Pool class.


get
---

*get($invalidation, [$args])*

The get function the data mapped to this particular Item. If no value is stored
at all then this function will return null. Since this can return false or null
as a correctly cached value, the return value should not be used to determine
successful retrieval of data- for that use the "isMiss()" function after calling
this one.

The get function can take a series of optional arguments defining how it handles
cache misses. The first of these options is the invalidation method to be used,
while the other options all provide invalidation specific options. The
:ref:`invalidation` page contains much more information about hos to use this
functionality.


isMiss
------

*isMiss()*

The isMiss function returns true when the current Item has either no data or
stale data. Since Stash is capable of storing both null and false values and
returning them via the get function, this is the only real way to test whether
a cached value is usable or not.

The exact behavior used to define a cache miss is defined by the invalidation
method used for the object. The :ref:`invalidation` page contains much more
information about hos to use this functionality.


set
---

*set($data, $ttl = null)*

The set function is used to populate the cache with data. The first argument
can be any type of data that is able to be serialized- essentially everything
except resources and classes which can't be serialized. All data put into Stash
will either come back exactly as inserted or not at all.

The second argument defines how long the item will be stored in the cache. This
is a maximum time, as items can be cleared or removed earlier depending on a
number of factors. This argument can be an integer representing the time, in
seconds, that the item will be considered valid. It can also be passed a
DateTime object signifying the exact moment the data should expire. Finally, for
items that are regenerated some other way, or which will never change, a null
value can be passed.


clear
-----

*clear()*

The clear function removes the current Item's data from the backend storage.

If hierarchical or "stackable" caching is being used this function will also
remove children Items. The Key section of the :ref:`basics` document goes into
more detail about how that works.


lock
----

*lock($ttl = null)*

The lock function is used to tell other processes and requests that this
particular item is being regenerated. This is typically called after the
"isMiss" function returns true, before the Item is regenerated and stored using
"set". Depending on the Invalidation method set for this Item, this prevents
multiple requests from attempting to regenerate the data in question,
potentially preventing a cache stampede (also known as the dogpile effect) as
multiple scripts attempt to run the same expensive code.

The exact effect of this function depends on which invalidation method is being
used. The :ref:`invalidation` page contains much more information about hos to
use this functionality.


disable
-------

*disable()*

The disable function disables all access to the "Driver" and forces the Item
class to gracefully fail most of it's calls.


getKey
------

*getKey()*

The getKey function returns this Item's key as a string. This is particularly
useful when the Item is returned as a group of Items in an Iterator.