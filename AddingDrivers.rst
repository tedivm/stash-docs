.. _addingdrivers:

==============
Adding Drivers
==============

Although Stash comes with a variety of built in Drivers there are plenty more that can be built. New Drivers are always
appreciated, so please feel free to issue a pull request to get it included in the core library.


Keys
====

Stash provides a variety of methods for developers to define keys, but it normalizes those keys into an array before
passing it to the driver. For the purposes of driver development a key is always an indexed array.

For example, where a user can represent a key as "path/to/data", it will always come to the Driver as
array('path', 'to', 'data).


DriverInterface
===============

setOptions
---------

*setOptions(array $options = array())*

Sets the driver for use by the caching system. This driver handles the direct interface with the caching backends,
keeping the system specific development abstracted out.

It takes an array which is used to pass option values to the driver. As this is the only required function that is used
specifically by the developer is is where any engine specific options should go. An engine that requires authentication
information, as an example, should get them here.

When defining this function it's important to use Key => Value pairs rather than indexed arrays, as it makes
standardizing configuration builders easier for people integrating this library.


getData
-------
*getData($key)*

Returns the previously stored data as well as it's expiration date in an associative array. This array contains two
keys- a 'data' key and an 'expiration' key. The 'data' key should be exactly the same as the value passed to storeData.

In the event that data is not present simply return false.


storeData
---------
*storeData(array $key, mixed $data, $expiration)*

Takes in data from the exposed Stash libraries and stored it for later retrieval.

* *Key* an array which should map to a specific, unique location for that array, This location should also be able to
  handle recursive deletes, where the removal of an item represented by an identical, but truncated, key causes all of
  the 'children' keys to be removed.

* *Data* is the value meant to be stored. This is an array which contains the raw storage as well as meta data about the
  data. The meta data can be ignored or used by the driver but entire data parameter must be retrievable exactly as it
  was placed in.

* *Expiration* is a timestamp containing representing the date and time the item will expire. This should also be
  stored, as it is needed by the getData function.


clear
-----
clear(array $key = null)

Clears the cache tree using the key array provided as the key. If called with no arguments the entire cache gets cleared.


purge
-----
*purge()*

This function provides a space for maintenance actions in the cache. It was originally meant for removing stale items in
storage engines that needed manual actions to do so (sqlite, filesystem) but can be used for any maintenance that should
not occur during regular user operations.


isAvailable
-----------
*isAvailable()*

Returns whether the driver is able to run in the current environment or not. Any system checks - such as making sure any
required extensions are missing - should be done here. This is a general check; if any instance of this driver can be
used in the current environment it should return true.
