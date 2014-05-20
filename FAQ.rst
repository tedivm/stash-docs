.. _faq:

====
FAQ
====

Packaging
=========

* **When is version 1.0 coming out?**

Right now the biggest thing holding back v1.0 is the completion of the PSR-Caching proposal. This proposal is meant to
define a common caching interface that libraries such as Stash can expose to any library that desires caching. Once this
is complete and we can be sure that Stash does not have anything in it that would prevent it from adopting this standard
a v1.0 release will be made and the Stash API will be stabilized.


* **What happened to the PEAR repositories?**

Frankly, the PEAR repositories were not getting nearly enough traffic to justify the work it took in maintaining them.
With modern packaging systems like Composer it's time to move on from PEAR.



Compatibility
=============

* **What's all this PSR stuff I keep heading about?**

PSR's are documents published by the PHP Framework Interoperability Group, of which Stash is a member. This group
attempts to make code more easily shared between libraries, frameworks and applications by creating standards that can
be used to ensure interoperability. Following these standards makes Stash more portable, and Stash can take advantage
of other libraries, such as monolog, that expose functionality through their own PSRs.


* **Does Stash work on Windows and OSX?**

Yup! Some drivers may have special options that need to be set, but these are all documented.


Drivers
=======

* **Why don't you support xcache?**

Unfortunately xcache does not allow itself to be loaded when being run from the command line. Although this makes sense
in theory, as cli scripts are not going to share memory between runs, it makes testing extremely difficult and without
testing it's hard to make high quality drivers.


* **Why don't you support wincache?**

Because we don't have a windows box to test it on, although that may change.


* **Can you support X caching system?**

We're happy to take requests, but even happier when they're pull requests.