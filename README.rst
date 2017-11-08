============================================
Linux distro: development/operations culture
============================================

------------
Introduction
------------

Technically GNU/Linux distro is an operating system made from a software
collection based upon the Linux kernel, GNU utilities, and a package
management system. Technically Linux is just another UNIX, and there used
to be technically superior UNIX systems (Solaris_). Yet Linux has
successfully obliterated its competitors in many fields, including, but
not limited to

* High performance computing, see Top500_
* Embedded systems. Everyone knows what Android is.
  Does anyone remeber Symbian or BlackBerry OS?
* Cloud computing

Why? Because GNU/Linux is not just a set of technologies and pile of
a software. Distro is a community with certain development and communication
culture. It's the culture which makes Linux superior.

.. _Top500: http://www.top500.org
.. _Solaris: http://en.wikipedia.org/wiki/Solaris_(operating_system)


---------------------------
Sociology of a Linux distro
---------------------------

* Users: the ones who use the software to perform their tasks
* Maintainers: the ones who integrate the software into the distro
* Developers: the ones who creates the software

Everyone is a user: developers and maintainers use compilers, source code
management systems, build systems, etc, without detailed understanding of
the internals of those tools.


----
User
----

The one who uses the software to perform his/her tasks.
Typical actions:

* `Installing the program`_
* `Running the program`_
* `Side note: program layout`_
* `Configuring the program`_
* `Troubleshooting the program`_
* `Upgrading software`_


Installing the program
======================

* Even if the software in question is open source, 99.9% of users
  won't bother to compile it.
 
* Try the version provided by the distro::

    apt-get install -y kdenlive

  or::

    yum install -y kdenlive

* If the version shipped with distro is not good enough, try distro's
  experimental packages or packages provided by the project::

    sudo apt-add-repository ppa:kdenlive/kdenlive-stable
    sudo apt-get update
    sudo apt-get install -y kdenlive

* Look for an equivalent software which provides binary packages for one's distro


Running the program
===================

* The binary *must* be in the default ``PATH``.
* The binary *must* run normally without any environment variables,
  from any directory::

    kdenlive &

* A daemon *must* use the system service supervision, logging facilities::

    systemctl start mysqld

* The program can make use of environment variables to change its
  behavior. In particular interactive programs can use ``LANG`` environment
  variable to present the UI in the correct (human) language.

* The behavior of program can be changed via configuration file(s).
  However the program should run reasonably without any configuration files.


Troubleshooting the program
===========================

* Failures are clear and immediate:

    - non-interactive program must log fatal errors and exit with a non-zero code

* The program can be profiled and/or debugged out of the box::

    apt-get install kdenlive-dbg
    perf record -g dwarf -p `pgrep kdenlive`

* The maintainer's contact info should be clearly stated::
  
    less /usr/share/doc/progname/changelog.Debian.gz

* The program is expected to integrate with distro's bug reporting tools

* User must not be forced to use proprietary protocols in order to communicate
  with the maintainer (or the upstream author).


Side note: program layout
=========================

A program consists of ``static`` and ``variable`` data, and configuration files. 
These components must be clearly separated.


Configuration files
-------------------

Configuration files is the interface for a user to change the program behavior.
Therefore the configuration files are expected to be changed by the user.
Also the user does not need to change either static or variable data in order
to configure the program.

* System wide configuration files *must* be located at

    - /etc/``program_name``.conf
    - /etc/``program_name``/\*.conf

* Per user configuration files (if applicable):

    - ~/.``program_name``
    - ~/.config/``program_name`` .conf
    - ~/.config/``program_name``/\*.conf

The package managment system is supposed to perserve user's changes of configuration
files during the upgrade.


Static data
-----------

* user commands (binaries, scripts): must be in ``/usr/bin/progname``
* plugins, helper binaries/scripts: ``/usr/libexec/progrname/libfrob.so``
* architecture independent data: ``/usr/share/progname/foo.xml``
* documentation: ``/usr/share/doc/progname/README``

The user is not supposed to change the static data.
The user don't have to change the static data in order to run the program.
The package manager is free to overwrite static data during the upgrade.
 

Variable data
-------------

* Variable data:

  - state information (databases managed by RDBMS, VMs' definitions managed
    by hypervisor, etc): ``/var/lib/progname``
  - logs: ``/var/log/progname``
  - caches: ``/var/cache/progname``

The user is not supposed to change the variable data directly.
The user don't have to change the variable data in order to run the program.
Typically the package manager (or rather the maintainer) must be careful
to not touch the state information during upgrades.


Configuring the program
=======================

* The configuration files *must* reside in well known locations,
  as explained in `Configuration files`_.
* The configuration files are human readable and can be edited with
  any text editor.
* A daemon is supposed to re-read configuration and apply changes on
  receiving ``SIGHUP``


Upgrading software
==================

::

  apt-get update && apt-get upgrade

Package manager is expected to

* Check if the new version of the software can be installed without
  breaking any other software
* Install new dependencies (libraries)
* Preserve user's modifications of config files
* Preserve valuable state info (databases, web sites, source code repositories,
  logs, etc)


----------
Developers
----------

Developers are just special case of users.

* The compiler, linker, build system, IDE work out of the box,
  from any directory, without any environment variables
* Immediate and clear indication of failures::

    gcc: committee.c:123: error: Too many arguments to function
* Otherwise no noise unless have been asked for it (``-Wall``)

Consequences:

- Setting up a development enviroment takes a few minutes
- Uniform installation routines familiar to everyone 


Distributing software in the source form
========================================

* A brief yet clear description of the software: ``README`` file.

* Clearly state the license:

    - COPYING, LICENSE in the top level directory of the source.
    - Comment in the beginning of every file pointing to the COPYING file.

* Everything necessary to build the software is in the *one* tarball
  or SCM repository.
* However readily available 3rd party components should not be included.
* Clearly state the prerequisites (in particular 3rd party software).
  Use the ``INSTALL`` file for that.
* After unpacking tarball everything is under ``progname-x.y.z`` directory.
  Not just ``progname``, let alone dumping the content into the current
  directory


Building software from the source
=================================

These days we are well beyond the dark ages of "Edit this Makefile
so this software builds on your computer"::

  ./configure && make && make check && sudo make install

* The user don't need to modify the source in order to build it.
* The configure script must do the right thing automatically (using
  gnu auto\* tools is not mandatory).
* Separate build and installation phases.
* Test suite: unit, functional, and other tests which don't need
  external resources and take 5 -- 10 minutes.
* No mandatory environment variables, magic paths, etc.
* The configure script has option to disable and enable optional
  features, specify the compiler and compilation options::

    ./configure --disable-static CXX=g++-5.3 CXXFLAGS='-O2 -g -Wall'

* The build is not supposed to interact with user (except via
  the exit code).
* For a shared library:

   - proper SONAME_ and version info, see also dsoabivers_
   - compilation and linking flags can be quieried with `pkg-config`::

       pkg-config --cflags --libs gtk-2.0
  
     this prints compilation and linking flags (``-I``, ``-L`` directives, etc)
     both for the library itself and its dependencies

* The components gets installed to:

    - binaries: ``prefix``/bin
    - libraries: ``prefix``/lib
    - headers: ``prefix``/include/``libname``
    - configuration files: ``prefix``/etc
    - static data files: ``prefix``/share

.. _SONAME: http://en.wikipedia.org/wiki/Soname
.. _dsoabivers: https://github.com/asheplyakov/dsoabivers


-----------
Maintainers
-----------

APT_ or yum_ are not magic. The reason why package managers work as expected
is numerious policies describing various packaging related processes
(example: `Debian policy`_). A regular developer might not be aware of all
peculiarites. The job of a maintainer is to help the upstream developer
to follow these policies, and to mediate the interaction between users
and the upstream developer.

Processes:

* Packaging software:

    - Formally describing the dependencies and the build process
      (possibly recursively)
    - Uploading the prepared sources to the build/CI infrastructure

* Processing bug reports:

    - Handling packaging related issues (missing/misplaced files,
      conflicts with other packages)
    - Forwarding users' bug reports to the upstream developer

.. _APT: http://wiki.debian.org/Apt
.. _yum: http://yum.baseurl.org
.. _Debian policy: http://www.debian.org/doc/debian-policy
