========
Handlers
========

FileSystem
==========

The StashFileSystem handler stores each item in a php script, as native php. Unsurprisingly this is the fastest backend for small to medium sites, as it is just as good as simply including a configuration file (and can even get stored in opcode caches). The downside is that clear and purge actions- those which have to recursively scan the cache's filesystem- take extraordinarily long compared to the other handlers.

* *dirSplit*
    Defines how many subdirectories to divide each key into. Larger cache pools run the risk of hitting file system limits on how many files can exist in a directory, so they need to divide the data up. Larger numbers can have performance issues, so testing should be done.
* *path*
    This optional parameter points the handler to the directory it should use for storage. If it isn't passed then an install specific directory is created inside the systems temporary directory.
* *filePermissions*
    The unix permission for new files. Defaults to 0660.
* *dirPermissions*
    The unix permission for new directories. Defaults to 0770.

.. code-block:: php
    // Uses a install specific default path if none is passed.
    $handler = new StashFileSystem();

    // Setting a custom path is done by passing an options array to the constructor.
    $options = array('path' => '/tmp/myCache/');
    $handler = new Stash\Handler\FileSystem($options);


Sqlite
======

An alternative file based caching backend is the StashSqlite handler. It can use either the PDO or native sqlite extensions, and can use either sqlite2 or sqlite3. 

* *extension*
    Either "pdo" or "sqlite". By default PDO is used and it falls back to sqlite, but if specified then only the passed extension will be used and no fallback action will be taken.
* *version*
    This takes either 2 or 3, with the default behavior being to use 3 and fallback to 2 if it isn't available.
* *nesting*
    Defaults to 0, this option defines how many levels of keys get their own sqlite file. A value of 0 uses one file, while a value of 1 will use one file for each unique first key and a value of two will use both the first and second key for creating sqlite files. For larger sites this can improve performance by spreading locks to multiple files, but for most sites this should be left at it's default.
* *path*
    This optional parameter points the handler to the directory it should use for storage. If it isn't passed then an install specific directory is created inside the systems temporary directory.
* *filePermissions*
    The unix permission for new files. Defaults to 0660.
* *dirPermissions*
    The unix permission for new directories. Defaults to 0770.

.. code-block:: php

    // StashSqlite

    // Uses a install specific default path if none is passed.
    $handler = new Stash\Handler\Sqlite();

    // Setting a custom path is done by passing an options array to the constructor.
    $options = array('path' => '/tmp/myCache/');
    $handler = new Stash\Handler\Sqlite($options);


APC
===

The APC extension is one of the most well known php caching extensions, allowing for both php opcode caching and memory storage of php values. The StashApc handler uses its userspace libraries to store data directly in memory for scripts to use.

* *ttl*
    This is the maximum time an item can live in memory. This is to keep memory pruned to small amounts, particularly when there is another handler backing this one.
* *namespace*
    This stores the data under a namespace in case other scripts are using APC to store data as well. If this isn't passed the handler creates an install specific namespace, so this is really only needed if two scripts need their own caches but are using the same Stash install.

.. code-block:: php

    // Uses a install specific default path if none is passed.
    $handler = new StashApc();

    // Setting a custom path is done by passing an options array to the constructor.
    $options = array('ttl' => 3600, 'namespace' = md5(__file__));
    $handler = new Stash\Handler\Apc($options);



Xcache (experimental)
=====================

The Xcache handler is currently experimental.

Like the APC handler, the Xcache handler stores data directly in memory for use by other scripts.


Memcached
=========

Memcached is a client/server application which allows machines to pool their memory together as one large memory cache. The StashMemcached is a feature complete handler for Memcached, complete with  hierarchal caching.

* *servers*
    An array of memcached servers, hosts and (optionally) weights for memcache. Each server is represented by an array- array(server, port, weight). If no servers are passed then the default of 127.0.0.1:11211 will be used.
* *extension*
    Which php extension to use, 'memcache' or 'memcached'. The default is to use the newer memcached and fallback to memcache if it is not available.
* *Options*
    Extension options can be passed to the "memcached" handler by adding them to the options array. The memcached extension defined options using contants, ie Memcached::OPT%. By passing in the % portion ('compression' for Memcached::OPT_COMPRESSION) and its respective option. Please see the `php manual for memcached <http://us2.php.net/manual/en/memcached.constants.php>`_ for the specific options.

.. code-block:: php

    // One Server
    $handler = new Stash\Handler\Memcache(array('servers' => array('127.0.0.1', '11211')));


    // Multiple Servers
    $servers = array();
    $servers[] = array('127.0.0.1', '11211', 60);
    $servers[] = array('10.10.10.19', '11211', 20);
    $servers[] = array('10.10.10.19', '11211', 20);

    $handler = new Stash\Handler\Memcache(array('servers' => $servers));

    // Using memcached options
    $options = array();
    $options['servers'][] = array('mem1.example.net', '11211');
    $options['servers'][] = array('mem2.example.net', '11211');

    $options['prefix_key'] = 'application_name';
    $options['libketama_compatible'] = true;
    $options['cache_lookups'] = true;
    $options['serializer'] = 'json';

    $handler = new Stash\Handler\Memcache($options);



MulitiHandler
=============

The StashMultiHandler acts as a wrapper around one or more handlers, allowing different handlers to work together in a single cache.

Upon creation the handler takes in an array of handlers as an option, with each handler after the first having a lower and lower priority. When get requests are run the handlers are checked by highest priority (first, second, third, etc) until the item is found. When an item is found in the cache the handlers that previously missed it are repopulated so they will hit on it next time. The store, clear and purge operations are run in reverse order to prevent stale data from being placed back into a cleared subhandler.

.. code-block:: php

    $subHandlers = array();
    $subHandlers[] = new Stash\Handler\Apc();
    $subHandlers[] = new Stash\Handler\FileSystem();
    $subHandlers[] = new Stash\Handler\Memcached();

    $options = array('handlers' => $subHandlers);
    $handler = new Stash\Handler\MultiHandler($options);

    $stash = new Stash\Cache($handler);
    $stash->makeKey('test');

    // First it checks StashApc. If that fails it checks StashFileSystem. If that succeeds it stores the returned value
    // from StashFileSystem into StashApc and then returns the value.
    $data = $stash->get();

    // First the data is stored in StashFileSystem, and then it is put into StashApc.
    $stash->store($data);

    // As with the store, function, the data is first removed from StashFileSystem before being cleared from StashApc.
    $stash->clear();
