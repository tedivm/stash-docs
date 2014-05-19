.. _groupings:

=========
Groupings
=========

Grouping is an extremely important feature of caching. By making it easier for developers to clear out related items
and for framework and application developers to isolate data among components, caching will become more common place and
application performance will improve as a result.

Stash offers two methods of grouping, Stacks and Namespaces. Although similar they are meant to serve distinct purposes-
Stacks are meant to give developers the ability to organize and invalidate their data in logical groups, while
Namespaces exist to let frameworks isolate different systems from each other. Combined they allow library and
application developers the ability to create Stacks and use Keys without having to worry key collision or what other
libraries may be storing in the cache.

Stacks
======

Stacks are a special kind of grouping system that allow cache items to be nested, similar to how folders are nested in
filesystems. Stacks work by adding a special character to Keys, the slash, which tells the Implementing Library where
the nesting points are. If the first character of the key is not a slash, Stacks behaves exactly like the standard Cache
interfaces.

An example key may look like "/Users/Bob/Friends", "/Users/Bob/Friends/Active" or just "/Users/Bob". The special thing
about Stacks is that clearing out "/Users/Bob" also clears out "/Users/Bob/Friends" and "/Users/Bob/Friends/Active".


Namespaces
==========

Namespaces can be used to separate the storage of different systems in the cache. This allows different sections to be
cleared on an individual level, while also preventing overlapping keys.

Namespaces are controlled by the Pool::setNamespace($name) function. Once a namespace is set all operations the Pool and
it's subsequent children Items do will in that isolated namespace (Items pulled out of the Pool before a namespace
change maintain their original namespace).