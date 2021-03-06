Getting started
===============

.. contents::
   :local:


OpenLDAP features enabled by default
------------------------------------

The ``debops.slapd`` role enables and configures some of the OpenLDAP features
that otherwise are enabled dynamically and could have different names in the
LDAP directory on different installations due to the order in which they were
enabled. If you are planning to apply the role on an existing installation, you
should review the configuration before doing so - the OpenLDAP server usually
refuses the incorrect configuration outright, which should not affect the
existing installation, but that's not a 100% guarantee.

Sync Provider overlay
~~~~~~~~~~~~~~~~~~~~~

The role will by default enable the `Sync Provider`__ (``syncprov``) dynamic
module and overlay, in both the ``cn=config`` configuration database, and the
main OpenLDAP database.

The Sync Provider functionality is used in different `data replication`__
strategies. Enabling it by default, even on a standalone OpenLDAP server,
should be harmless - the replication requires additional configuration defined
in each OpenLDAP database. The overlay is enabled first to keep the
``X-ORDERED`` index number consistent between the ``cn=config`` database and
the main database.

.. __: http://www.zytrax.com/books/ldap/ch6/syncprov.html
.. __: https://www.openldap.org/doc/admin24/replication.html

Password Policy overlay
~~~~~~~~~~~~~~~~~~~~~~~

The :ref:`debops.slapd` role will by default import the ``ppolicy`` LDAP
schema, load the ``ppolicy`` dynamic module and enable the Password Policy
overlay in the main OpenLDAP database.

The `Password Policy`__ overlay is used to maintain the security and quality of
various passwords stored in the LDAP database. By default the overlay will
ensure that the cleartext passwords passed to the OpenLDAP server are hashed
using the algorithms specified in the ``olcPasswordHash`` parameter (salted
SHA-512 via :man:`crypt(3)` function is set by default by the
:ref:`debops.slapd` role).

The LDAP administrators can define default and custom Password Policies in the
main database, which can enforce additional password requirements, like minimum
password length, different types of characters used, lockout policy, etc.

.. __: http://www.zytrax.com/books/ldap/ch6/ppolicy.html


Access to service allowed by default
------------------------------------

The default configuration allows access to the OpenLDAP service from anywhere
through the firewall and TCP Wrappers. Keep that in mind while designing the
LDAP Access Control List and password policies. You can control the default
behaviour using the :envvar:`slapd__accept_any` boolean variable. Another
option is to use an external firewall with IDS/IPS systems that can analyze
LDAP traffic.


Example inventory
-----------------

To install and manage the OpenLDAP server on a host, you need to add it to the
``[debops_service_slapd]`` Ansible inventory group:

.. code-block:: none

   [debops_service_slapd]
   hostname


Example playbook
----------------

If you are using this role without DebOps, here's an example Ansible playbook
that uses the ``debops.slapd`` role:

.. literalinclude:: ../../../../ansible/playbooks/service/slapd.yml
   :language: yaml

The included :ref:`debops.ferm` and :ref:`debops.tcpwrappers` Ansible roles are
optional. They can be used for managing firewall and access rules to the LDAP
service.

If you further want to enable LDAP transport layer security in ``debops.slapd``
role, the :ref:`debops.pki` and :ref:`debops.dhparam` roles must also be
applied on the host. The ``debops.slapd`` role will automatically detect and
use their configured environments if available.


Ansible tags
------------

You can use Ansible ``--tags`` or ``--skip-tags`` parameters to limit what
tasks are performed during Ansible run. This can be used after host is first
configured to speed up playbook execution, when you are sure that most of the
configuration has not been changed.

Available role tags:

``role::slapd``
  Main role tag, should be used in the playbook to execute all of the role
  tasks as well as role dependencies.


Other resources
---------------

List of other useful resources related to the ``debops.slapd`` Ansible role:

- Manual pages: :man:`slapd(8)`, :man:`slapd-config(5)`

- `LDAP for Rocket Scientists`__, an excellent book about LDAP and OpenLDAP

  .. __: http://www.zytrax.com/books/ldap/

- `Debian LDAP Portal`__ page in the Debian Wiki

  .. __: https://wiki.debian.org/LDAP
