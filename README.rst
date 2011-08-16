User Management with Bcfg2
==========================

Overview
--------

The files in this repository implement basic user management with Bcfg2_.
This allows for adding users and groups, modifying their attributes, and
managing ``~/.ssh/authorized_keys`` files according to an XML specification.
Users and groups which exist on the managed Bcfg2_ clients but are not
defined in the specification are ignored.

The ``bcfg2-lint(8)`` tool available in Bcfg2_ 1.2.0 and newer can be used
to validate the XML specification syntactically.

Prerequisites
-------------

* The Bundler, Properties, and Probes plugins must be enabled in the Bcfg2_
  server configuration.

* The following tools must be available on the managed Bcfg2_ clients:

  - ``useradd(8)``
  - ``usermod(8)``
  - ``groupadd(8)``
  - ``groupmod(8)``
  - ``getent(1)``

  The `Bundler/accounts.genshi`_ template expects to find the first four of
  these commands in the ``/usr/sbin`` directory of the client.  If they are
  installed to another location, the ``<BoundAction>`` entries in the
  `Bundler/accounts.genshi`_ template must be modified accordingly.

Installation
------------

* Copy `Bundler/accounts.genshi`_, `Probes/accounts`_,
  `Properties/accounts.xsd`_, and `Properties/keys.xsd`_ to your Bcfg2_
  configuration repository (which is located at ``/var/lib/bcfg2`` by
  default).

* Add ``<Bundle name='accounts'/>`` to the desired group(s) in your
  `Metadata/groups.xml`_ file.

Usage
-----

Users and groups are specified in the `Properties/accounts.xml`_ file, SSH
public keys are grouped in the `Properties/keys.xml`_ file.  The content of
both files must be enclosed in ``<Properties>`` root tags.  ``<Group>`` and
``<Client>`` tags (and their ``negate`` attribute) can be used as in other
places of the Bcfg2_ configuration (see the `Bundler documentation`_ for
details on how they are parsed).

This repository includes `examples`_ of both files, which should be mostly
self-explanatory.

Properties/accounts.xml
~~~~~~~~~~~~~~~~~~~~~~~

In the `Properties/accounts.xml`_ file, the desired users and groups are
specified using ``<UnixUser>`` and ``<UnixGroup>`` entries.

The supported ``<UnixUser>`` attributes are:

============ ==============================================================
Attribute    Description
============ ==============================================================
name         user name *(required!)*
uid          user's UID *(required!)*
group        primary group name *(default: user name)*
gid          group's GID *(default: uid)*
gecos        comment *(default: capitalized user name)*
home         home directory *(default: /home/name or /root)*
shell        login shell *(default: specified in Bundler/accounts.genshi)*
extra_groups space-separated list of supplementary groups *(default: none)*
key_group    see below *(default: don't touch the authorized_keys file)*
============ ==============================================================

The supported ``<UnixGroup>`` attributes are:

============ ==============================================================
Attribute    Description
============ ==============================================================
name         group name *(required!)*
gid          group's GID *(required!)*
============ ==============================================================

Note that the user's primary group will be created automatically if it
doesn't exist on the client, whereas any specified ``extra_groups`` must be
defined explicitly using ``<UnixGroup>`` entries if they don't already exist
on all clients in question.

Properties/keys.xml
~~~~~~~~~~~~~~~~~~~

First of all, the SSH public key files which will be referenced in the
specification must be copied into the directory ``/var/lib/bcfg2/keys``.  To
change this directory path, the ``key_directory`` setting at the top of the
`Bundler/accounts.genshi`_ file must be modified.

In the Properties/keys.xml file, SSH public key files are grouped using
``<PubKey file='foo.pub'/>`` entries within ``<KeyGroup name='bar'>`` tags.
These ``<KeyGroup>`` tags may also include other ``<KeyGroup>`` tags such as
``<KeyGroup name='inherited'/>`` in order to include the members of the
``inherited`` group.

The ``<KeyGroup>`` name can then be referenced using the ``key_group``
attribute of ``<UnixUser>`` tags in the `Properties/accounts.xml`_ file.
This adds the content of all ``<PubKey>`` files in this ``<KeyGroup>`` to
the ``~/.ssh/authorized_keys`` file of the ``<UnixUser>`` in question.  Any
other data will be removed from the ``~/.ssh/authorized_keys`` file.

Author
------

`Holger Weiss`_

Copyright and License
---------------------

Copyright (c) 2011 `Freie Universitaet Berlin`_

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to
deal in the Software without restriction, including without limitation the
rights to use, copy, modify, merge, publish, distribute, sublicense, and/or
sell copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.  IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS
IN THE SOFTWARE.

.. References

.. _examples:
   https://github.com/weiss/bcfg2-accounts/tree/master/Properties
.. _Bundler/accounts.genshi:
   https://raw.github.com/weiss/bcfg2-accounts/master/Bundler/accounts.genshi
.. _Probes/accounts:
   https://raw.github.com/weiss/bcfg2-accounts/master/Probes/accounts
.. _Properties/accounts.xml:
   https://raw.github.com/weiss/bcfg2-accounts/master/Properties/accounts.xml
.. _Properties/accounts.xsd:
   https://raw.github.com/weiss/bcfg2-accounts/master/Properties/accounts.xsd
.. _Properties/keys.xml:
   https://raw.github.com/weiss/bcfg2-accounts/master/Properties/keys.xml
.. _Properties/keys.xsd:
   https://raw.github.com/weiss/bcfg2-accounts/master/Properties/keys.xsd
.. _Metadata/groups.xml:
   http://docs.bcfg2.org/server/plugins/grouping/metadata.html
.. _Bundler documentation:
   http://docs.bcfg2.org/server/plugins/structures/bundler/
.. _Bcfg2:
   http://www.bcfg2.org/
.. _Freie Universitaet Berlin:
   http://www.fu-berlin.de/
.. _Holger Weiss:
   holger@zedat.fu-berlin.de

.. vim:set joinspaces textwidth=76:
