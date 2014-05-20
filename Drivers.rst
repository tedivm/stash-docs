.. _drivers:

=======
Drivers
=======

System
======

FileSystem
----------

The Filesystem driver stores each item in a php script, as native php.
Unsurprisingly this is the fastest backend for small to medium sites, as it is
just as good as simply including a configuration file (and can even get stored
in opcode caches). The downside is that clear and purge actions- those which
have to recursively scan the cache's filesystem- take extraordinarily long
compared to the other drivers.

* *dirSplit*
    Defines how many subdirectories to divide each key into. Larger cache pools
    run the risk of hitting file system limits on how many files can exist in a
    directory, so they need to divide the data up. Larger numbers can have
    performance issues, so testing should be done.

* *path*
    This optional parameter points the driver to the directory it should use for
    storage. If it isn't passed then an install specific directory is created
    inside the systems temporary directory.

* *filePermissions*
    The unix permission for new files. Defaults to 0660.

* *dirPermissions*
    The unix permission for new directories. Defaults to 0770.

.. code-block:: php

    <?php
    // Uses a install specific default path if none is passed.
    $driver = new Stash\Driver\FileSystem();

    // Setting a custom path is done by passing an options array to the constructor.
    $options = array('path' => '/tmp/myCache/');
    $driver->setOptions($options);


Sqlite
------

An alternative file based caching backend is the Sqlite driver. It can use
either the PDO or native sqlite extensions, and can use either sqlite2 or
sqlite3.

* *extension*
    Either "pdo" or "sqlite". By default PDO is used and it falls back to
    sqlite, but if specified then only the passed extension will be used and no
    fallback action will be taken.

* *version*
    This takes either 2 or 3, with the default behavior being to use 3 and
    fallback to 2 if it isn't available.

* *nesting*
    Defaults to 0, this option defines how many levels of keys get their own
    sqlite file. A value of 0 uses one file, while a value of 1 will use one
    file for each unique first key and a value of two will use both the first
    and second key for creating sqlite files. For larger sites this can improve
    performance by spreading locks to multiple files, but for most sites this
    should be left at it's default.

* *path*
    This optional parameter points the driver to the directory it should use for
    storage. If it isn't passed then an install specific directory is created
    inside the systems temporary directory.

* *filePermissions*
    The unix permission for new files. Defaults to 0660.

* *dirPermissions*
    The unix permission for new directories. Defaults to 0770.

.. code-block:: php

    <?php
    // StashSqlite

    // Uses a install specific default path if none is passed.
    $driver = new Stash\Driver\Sqlite();

    // Setting a custom path is done by passing an options array to the constructor.
    $options = array('path' => '/tmp/myCache/');
    $driver->setOptions($options);


APC
---

The APC extension is one of the most well known php caching extensions, allowing
for both php opcode caching and memory storage of php values. The StashApc
driver uses its userspace libraries to store data directly in memory for scripts
to use.

* *ttl*
    This is the maximum time an item can live in memory. This is to keep memory
    pruned to small amounts, particularly when there is another driver backing
    this one.

* *namespace*
    This stores the data under a namespace in case other scripts are using APC
    to store data as well. If this isn't passed the driver creates an install
    specific namespace, so this is really only needed if two scripts need their
    own caches but are using the same Stash install.

.. code-block:: php

    <?php
    // Uses a install specific default path if none is passed.
    $driver = new Stash\Driver\Apc();

    // Setting a custom path is done by passing an options array to the constructor.
    $options = array('ttl' => 3600, 'namespace' = md5(__file__));
    $driver->setOptions($options);


Server
======

Memcached
---------

Memcached is a client/server application which allows machines to pool their
memory together as one large memory cache. The Memcached driver is a feature
complete driver for Memcached, complete with hierarchical caching.

* *servers*
    An array of memcached servers, hosts and (optionally) weights for memcache.
    Each server is represented by an array- array(server, port, weight). If no
    servers are passed then the default of 127.0.0.1:11211 will be used.

* *extension*
    Which php extension to use, 'memcache' or 'memcached'. The default is to use
    the newer memcached and fallback to memcache if it is not available.

* *Options*
    Extension options can be passed to the "memcached" driver by adding them to
    the options array. The memcached extension defined options using contants,
    ie Memcached::OPT%. By passing in the % portion ('compression' for
    Memcached::OPT_COMPRESSION) and its respective option. Please see the `php
    manual for memcached <http://us2.php.net/manual/en/memcached.constants.php>`_
    for the specific options.

.. code-block:: php

    <?php
    // One Server
    $driver = new Stash\Driver\Memcache();
    $driver->setOptions(array('servers' => array('127.0.0.1', '11211')));


    // Multiple Servers
    $driver = new Stash\Driver\Memcache();

    $servers = array();
    $servers[] = array('127.0.0.1', '11211', 60);
    $servers[] = array('10.10.10.19', '11211', 20);
    $servers[] = array('10.10.10.19', '11211', 20);

    $driver->setOptions(array('servers' => $servers));


    // Using memcached options
    $driver = new Stash\Driver\Memcache();

    $options = array();
    $options['servers'][] = array('mem1.example.net', '11211');
    $options['servers'][] = array('mem2.example.net', '11211');

    $options['prefix_key'] = 'application_name';
    $options['libketama_compatible'] = true;
    $options['cache_lookups'] = true;
    $options['serializer'] = 'json';

    $driver->setOptions($options);


Redis
-----

Redis is a high performing advanced caching and key/value storage system. This
driver uses the Redis PHP extension to enable Redis based caching, using one or
more servers.

* *servers*
    An array of Redis servers and (optionally) ports to connect to. Each server
    is represented by an array- array(server, port). If no servers are passed
    then the default of 127.0.0.1:6379 will be used.

.. code-block:: php

    <?php
    // One Server
    $driver = new Stash\Driver\Redis();
    $driver->setOptions(array('servers' => array('127.0.0.1', '6379')));


    // Multiple Servers
    $driver = new Stash\Driver\Redis();

    $servers = array();
    $servers[] = array('127.0.0.1');
    $servers[] = array('10.10.10.20');
    $servers[] = array('10.10.10.19', '6379');
    $servers[] = array('10.10.10.19', '6380');

    $driver->setOptions(array('servers' => $servers));


Specialized
===========

Ephemeral
---------

The Ephemeral driver is a special backend that only stores data for the lifetime
of the script, whether it be a longer running process or a web request. Items
pushed to this driver are stored in the script's running memory. This driver has
no options.

When combined with the Composite driver the Ephemeral driver can reduce the load
on the underlying caching services by storing returns in memory to reduce
duplicate lookups (caching the cache, in a way).

.. code-block:: php

    <?php
    $pool = new Stash\Pool(new Stash\Driver\Ephemeral())
    $item = $pool->getItem('test');
    $item->set('data');

    echo $item->get(); // Outputs "data".

On subsequent requests, however, the data is not there-

.. code-block:: php

    <?php
    $pool = new Stash\Pool(new Stash\Driver\Ephemeral())
    $item = $pool->getItem('test');

    var_dump($item->isMiss()); // Outputs "true";


Composite
---------

The Composite driver acts as a wrapper around one or more drivers, allowing
different drivers to work together in a single cache.

Upon creation the driver takes in an array of drivers as an option, with each
driver after the first having a lower and lower priority. When get requests are
run the drivers are checked by highest priority (first, second, third, etc)
until the item is found. When an item is found in the cache the drivers that
previously missed it are repopulated so they will hit on it next time. The
store, clear and purge operations are run in reverse order to prevent stale data
from being placed back into a cleared subdriver.

.. code-block:: php

    <?php
    $subDrivers = array();
    $subDrivers[] = new Stash\Driver\Apc();
    $subDrivers[] = new Stash\Driver\FileSystem();
    $subDrivers[] = new Stash\Driver\Memcached();

    $options = array('drivers' => $subDrivers);
    $driver = new Stash\Driver\Composite($options);

    $pool = new Stash\Pool($driver);
    $item = $pool->getItem('test');

    // First it checks Apc. If that fails it checks FileSystem. If that succeeds
    it stores the returned value
    // from FileSystem into Apc and then returns the value.
    $data = $stash->get();

    // First the data is stored in FileSystem, and then it is put into Apc.
    $stash->set($data);

    // As with the store function, the data is first removed from FileSystem
    before being cleared from Apc.
    $stash->clear();
