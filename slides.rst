
Margarita Shotgun
=================

Automating Memory Capture
=========================

Joel Ferrier

----

#whoami
-------

Joel Ferrier

SysAdmin

ThreatResponse Team Member

----

Important Things
----------------

- Everything in this talk is FOSS
- These are my opinions not the opinions of my employer

----

Agenda
======

Why Memory Capture
------------------

Memory Capture Tools
--------------------

Memory Capture Automation
-------------------------

----

Why Memory Capture
==================

- Don't pull the plug
- Scope of compromise unknown
- Capture now, analyize later
- Malware encrypted on disk
- Malware non persistent on disk
- State information

.. note:: state information, current connections, process info

----

Tools (Linux)
=============

`LiME <https://github.com/504ensicsLabs/LiME>`__
------------------------------------------------

- loadable kernel module
- Multiple output mechanisms

.. note:: LiME - Roots in android memory capture
.. note:: outputs, file, socket

`LinPMem <http://www.rekall-forensic.com/docs/Tools/>`__
--------------------------------------------------------

- Loadable kernel module
- PMem family of tools
- Rekall Project

----

Using LiME (Best Case)
----------------------

1. Connect to host
2. Identify Kernel Version, Distro, etc
3. Fetch pre-compiled LiME module
4. Load Kernel Module
5. Dump Memory

----

Using LiME (Worst Case)
-----------------------

1. Connect to host
2. Identify kernel version, distro, etc
3. Locate/install same distro
4. Install gcc toolchain, kernel headers
5. Clone the LiME Repository
6. Compile LiME
7. Load Kernel Module
8. Dump Memory

----

Best Case
---------

- Steps performed by hand
- External Dependencies

.. note:: kernel modules - external depends

Worst Case
----------

- Each step performed by hand
- External Dependencies
- Development tools required
- Slow

----

Introducing Margarita Shotgun
=============================

----

Margarita Shotgun
-----------------

- Python module
- Compatible with Python 2 and 3
- Install with pip
- Automatic

.. note:: quick installation
   metadata collection
   log of all actions taken agains instance

----

Goals
-----

- Support complex networks
- Secure memory capture over the wire
- Multiple storage backends
- Capture memory in parallel

----

Tech
----

Paramiko
~~~~~~~~

Multiprocessing
~~~~~~~~~~~~~~~

S3 Storage
~~~~~~~~~~

----

Paramiko
========

- Python SSH Client Library
- Low Level client access
- Nested SSH Transports
- SSH Tunnel support

.. note:: nest == jump hosts, direct tcp-ip connection
   no shell outs with paramiko for tunnels, ssh command execution

----

Multiprocessing
---------------

- GIL per process
- Classically Parallel
- Process Pool


.. note:: I used the multiprocessing library to bypass a shortcoming of cpython,
   for those of you who are not familiar with the Global Interpreter Lock
   it is a Lock that prevents more than one thread from executing at a time.
   If we create a process per memory capture we can use the resources of our
   memory capture host more efficiently and complete captures faster

----

S3 Storage Backend
==================

s3fs
----

- Write files directly to s3
- Multipart upload
- Zero local disk space required

----

What About The Kernel Modules?
==============================

----

Introducing lime-compiler
=========================

----

Lime Compiler
-------------

Ruby Gem
~~~~~~~~

Docker Integration
~~~~~~~~~~~~~~~~~~

Supports many common distros
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. note:: gem listed in rubygems for next release, docker, use containers to install and build against kernel headers in official repositories

----

Why Build Ahead Of Time?
------------------------

- Module compilation at run time is slow
- External dependencies
- Building against the comprimised host contaminates the instance

.. note:: why compile ahead of time, instead of runtime?
   building at run time assumes much about the responder's environment


----

Public Repository
-----------------

- Hosted in s3
- Direct output from lime compiler
- `Open to the world <https://threatresponse-lime-modules.s3.amazonaws.com/>`__

.. note:: we use lime-compiler to build kernel modules for a public repository
   our repository is availible as a convinience, but you don't have to trust us, you can run your own
   daily builds coming soon

----

Next Release
~~~~~~~~~~~~

- Coming in a week or two
- Repository Metadata
- Module Signing

.. note:: current implementation relies on s3 xml bucket listing
   introduce metadata files, host in s3 or apache/nginx on prem

----

Caveats
-------

Amazon Linux
~~~~~~~~~~~~

- No docker images
- Special case for building modules

.. note:: amazon linux doesn't have docker support
   build against local server if you run on amazon linux

----

Margarita Shotgun
-----------------

Kernel Module Repository disabled by default
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Override repository with your own
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. note:: margarita shotgun itegrates with the lime compiler repository
   disabled by default, don't make requests unless you want to
   override our repository, no need to trust us

----

Recap
=====

Using LiME (Worst Case)
-----------------------

1. Connect to host
2. Identify kernel version, distro, etc
3. Locate/install same distro
4. Install gcc toolchain, kernel headers
5. Clone the LiME Repository
6. Compile LiME
7. Load Kernel Module
8. Dump Memory

----

Demo
----

 .. raw:: html

      <div id="player-container" style="width: 100%;"></div>
      <script>
          asciinema.player.js.CreatePlayer('player-container',
              'casts/marsho.json',
              {
                  speed : 3,
              }
              );
      </script>

----

Did I mention it's a library?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


.. code:: python

   >>> import margaritashotgun
   >>> config = dict(aws dict(bucket = 'case-bucket'),
   ...               hosts = [ dict(addr = '10.10.12.10',
   ...                              port = 22,
   ...                              username = 'ec2-user',
   ...                              key = '/path/to/private-key') ]
   ...               workers = 'auto',
   ...               logging = dict(log_dir = 'logs/',
   ...                              prefix = 'casenumber-10.10.12.10'),
   ...               repository = dict(enabled = true,
   ...                                 url = 'repo.module-repo.io'))
   ...
   >>> capture_client = margaritashotgun.client(name='mem-capture',
   ...                                          config=config,
   ...                                          library=True,
   ...                                          verbose=False)
   ...
   >>> response = capture_client.run()
   >>> print(response)
   {'total':1,'failed':[],'completed':['10.10.12.10']}

----

Future Work
-----------

Two Factor Authentication
~~~~~~~~~~~~~~~~~~~~~~~~~

Delivery Methods
~~~~~~~~~~~~~~~~

- SSM

Windows?
~~~~~~~~

.. note:: Open a github issue and let us know what you want

----

Thanks
------

 Andrew Krug @andrewkrug

 Alex McCormack @amccormack

 Jeff Parr @jparr

 Kevin Hock @KevinHock2

 Amazon Security

----

Joel Ferrier
------------

`Margarita Shotgun <https://github.com/ThreatResponse/margaritashotgun/>`_
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

https://github.com/ThreatResponse/margaritashotgun

`Lime Compiler  <https://github.com/ThreatResponse/lime-compiler/>`_
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

https://github.com/ThreatResponse/lime-compiler


Check them out and contribute!

