Preparation
===========

Requirements
------------

For the FreeIPA workshop you will need to:

- Install Vagrant_ (using VirtualBox provider) and the
  *vagrant-hostmanager* plugin

- Clone the repository containing the ``Vagrantfile``

- Fetch the Vagrant *box* for the workshop

- Add entries for the guest VMs to your hosts file (so you can
  access them by their hostname)


.. _Vagrant: https://www.vagrantup.com/

Please set up these items **prior to the workshop**.  More detailed
instructions follow.


Install Vagrant
---------------

On Fedora 22::

  $ sudo dnf install -y vagrant VirtualBox
  $ sudo akmods
  $ sudo modprobe vboxdrv vboxnetadp


On Mac OS X::

 TODO


On Ubuntu::

 TODO


On Windows::

  TODO


Install vagrant-hostmanager
---------------------------

::

  $ vagrant plugin install vagrant-hostmanager


Fetch Vagrant box
-----------------

Please fetch the Vagrant box prior to the workshop.  It is > 500MB
so it may not be feasible download it during the workshop.

::

  $ vagrant box add ftweedal/freeipa-workshop


Add hosts file entries
----------------------

Add the following entries to your hosts file::

  192.168.33.10   server.ipademo.local
  192.168.33.20   client.ipademo.local

On Unix systems including Mac OS X, the hosts file is ``/etc/hosts``
(you need elevated permissions to edit it.)

On Windows, edit ``C:\system32\system\drivers\etc\hosts`` as
*Administrator*.


Introduction
============

The FreeIPA workshop will take you through installing a FreeIPA
server, enrolling client machines, and managing users, services and
access policies.

The workshop is designed to be carried out in Vagrant_ environment
consisting of several hosts:

``server``
  The host on which you will install the FreeIPA server.  Its
  hostname is ``server.ipademo.local``.

``client``
  A host to be enrolled in the FreeIPA domain, from which a user can
  obtain a Kerberos ticket and access services.  Its hostname is
  ``client.ipademo.local``.


Explanatory notes
-----------------

This guide contains many command examples.  Commands will be
executed on a variety of servers, including the virtual machines as
well as the virutalisation host.

For clarity, commands are annotated with the host on which they are
meant to be executed::

  $ echo "Run it on virtualisation host (no annotation)"

  [server]$ echo "Run it on FreeIPA server"

  [client]$ echo "Run it on IPA-enrolled client"

The *vagrant-hostmanager* plugin is used to manage the
``/etc/hosts`` file on the virtual machines, so that they can find
each other.  In an industrial test or production deployment of
FreeIPA DNS should provide this information (including PTR records).


Module 1: FreeIPA server installation
=====================================

In this module you will install the FreeIPA server which you will
use for the rest of the workshop.

First bring up the Vagrant environment (all hosts will come up)::

  $ vagrant up --provider virtualbox


From the directory containing the ``Vagrantfile`` SSH into the
``server`` machine::

  $ vagrant ssh server


On ``server``, start the FreeIPA server installation program::

  [server]$ sudo ipa-server-install --no-host-dns

The ``--no-host-dns`` argument is needed because there is no DNS PTR
resolution for the Vagrant environment.  For production deployment
this important sanity check should not be skipped.

You will be asked a series of questions.
Accept defaults for most questions, except as outlined
below.

Configure FreeIPA's DNS server::

  Do you want to configure integrated DNS (BIND)? [no]: yes

  Existing BIND configuration detected, overwrite? [no]: yes

Accept default values for server hostname, domain name and realm::

  Enter the fully qualified domain name of the computer
  on which you're setting up server software. Using the form
  <hostname>.<domainname>
  Example: master.example.com.


  Server host name [server.ipademo.local]: 

  Warning: skipping DNS resolution of host server.ipademo.local
  The domain name has been determined based on the host name.

  Please confirm the domain name [ipademo.local]: 

  The kerberos protocol requires a Realm name to be defined.
  This is typically the domain name converted to uppercase.

  Please provide a realm name [IPADEMO.LOCAL]: 


Enter passwords for *Directory Manager* (used to manage the
directory server) and *admin* (the main account used for FreeIPA
administration).  Use something simple that you're not going to
forget during the workshop!

::

  Certain directory server operations require an administrative user.
  This user is referred to as the Directory Manager and has full
  access
  to the Directory for system management tasks and will be added to
  the
  instance of directory server created for IPA.
  The password must be at least 8 characters long.

  Directory Manager password: 
  Password (confirm): 

  The IPA server requires an administrative user, named 'admin'.
  This user is a regular system account used for IPA server
  administration.

  IPA admin password: 
  Password (confirm): 


Configure DNS forwarders and the reverse zone::

  Do you want to configure DNS forwarders? [yes]: 
  Enter the IP address of DNS forwarder to use, or press Enter to
  finish.
  Enter IP address for a DNS forwarder: <something from your resolv.conf>
  DNS forwarder <as above> added
  Enter IP address for a DNS forwarder: <press ENTER to end list>
  Checking forwarders, please wait ...
  Do you want to configure the reverse zone? [yes]: 
  Please specify the reverse zone name [33.168.192.in-addr.arpa.]: 
  Using reverse zone(s) 33.168.192.in-addr.arpa.


Next, you will be presented with a summary of the server
configuration and asked for final confirmation.  Affirm to begin the
server installation::

  The IPA Master Server will be configured with:
  Hostname:       server.ipademo.local
  IP address(es): 192.168.33.10
  Domain name:    ipademo.local
  Realm name:     IPADEMO.LOCAL

  BIND DNS server will be configured to serve IPA domain with:
  Forwarders:    10.0.2.3
  Reverse zone(s):  33.168.192.in-addr.arpa.

  Continue to configure the system with these values? [no]: yes

The installation takes a few minutes; you will see output indicating
the progress.

When it completes, run ``kinit admin`` and enter your *admin*
password to obtain a Kerberos ticket granting ticket (TGT) for the
``admin`` user::

  [server]$ kinit admin
  Password for admin@IPADEMO.LOCAL:  <enter password>

Run ``klist`` to view your current Kerberos tickets::

  [server]$ klist
  Ticket cache: KEYRING:persistent:1000:1000
  Default principal: admin@IPADEMO.LOCAL

  Valid starting     Expires            Service principal
  10/15/15 01:48:59  10/16/15 01:48:57
  krbtgt/IPADEMO.LOCAL@IPADEMO.LOCAL

The FreeIPA server is now set up and you are ready to begin
enrolling client machines, creating users, managing services and
more!


Module 2: Client enrolment
==========================

In this module, you will enrol a *host* as a client of your FreeIPA
domain.  This means that *users* in your FreeIPA realm (or Active
Directory realms for which there is a trust with FreeIPA) can log
into the client machine (subject to access policies) and *services*
on the client can leverage FreeIPA's authentication and
authorisation services.

From the directory containing the ``Vagrantfile`` SSH into the
``client`` machine::

  $ vagrant ssh client


On ``client``, start the FreeIPA client enrolment program::

  [client]$ sudo ipa-client-install

The FreeIPA server should be detected through DNS autodiscovery.
(If DNS discovery fails, e.g. due to client machine having incorrect
``/etc/resolv.conf`` configuration, you would be prompted to
manually enter the domain and server hostname instead).

The autodetected server settings will be displayed; confirm to
proceed::

  [client]$ sudo ipa-client-install
  Discovery was successful!
  Hostname: client.ipademo.local
  Realm: IPADEMO.LOCAL
  DNS Domain: ipademo.local
  IPA Server: server.ipademo.local
  BaseDN: dc=ipademo,dc=local

  Continue to configure the system with these values? [no]: yes


The client machine's clock will be synchronised to the server's (the
Kerberos protocol requires this).  You will then be prompted to
enter credentials of a user authorised to enrol hosts (``admin``)::

  Synchronizing time with KDC...
  Attempting to sync time using ntpd.  Will timeout after 15 seconds
  User authorized to enroll computers: admin
  Password for admin@IPADEMO.LOCAL: 

The enrolment now proceeds; no further input is required.  You will
see output detailing the operations being completed.  Unlike
``ipa-server-install``, client enrolment only takes a few seconds.

Users in your FreeIPA domain can now log onto FreeIPA-enrolled
hosts, subject to *Host-based access control* (HBAC) rules.  Users
logged onto the host can also acquire Kerberos tickets for accessing
*services* in your domain.


Module 3: User management
=========================

This module introduces the ``ipa`` CLI program and the web
interface.  We will perform some simple administrative tasks: adding
groups and users and managing group membership.

Web UI
------

Visit ``https://server.ipademo.local/``.  You'll get a TLS
*untrusted issuer* warning which you can dismiss (add a temporary
exception).  Login as ``admin``.

Welcome to the FreeIPA web UI.  Most management activities can be
performed here, or via the ``ipa`` CLI program.  See if you can work
out how to add a *User Group* (let's call it ``sysadmin``) and a
*User* (give her the username ``alice``).  Make ``alice`` a member
of the ``sysadmin`` group.


CLI
---

On ``server``, make sure you have a Kerberos ticket for ``admin``
(reminder: ``kinit admin``).

Most FreeIPA adminstrative actions can be carried out using the
``ipa`` CLI program.  Let's see what commands are available::

  [server]% ipa help commands
  automember-add                    Add an automember rule.
  automember-add-condition          Add conditions to an automember rule.
  automember-default-group-remove   Remove default (fallback) group for all unmatched entries.
  automember-default-group-set      Set default (fallback) group for all unmatched entries.
  automember-default-group-show     Display information about the default (fallback) automember groups.
  ...

Whoa!  There's almost 300 of them!  We'll only be using a handful of
these today.

You'll notice that commands are grouped by *plugin*.  You can get a
general overview of a plugin by running ``ipa help <plugin>``, and
specific information on a particular command by running ``ipa help
<command>``.

Let's add the user *bob* from the CLI.  See if you can work out how
to do this using the CLI help commands.  (**hint**: the plugin name
is ``user``).


User authentication
-------------------

We have seen how to authenticate as ``admin``.  The process is the
same for regular users - just ``kinit <username>``!

Try to authenticate as ``bob``::

  [server]$ kinit bob
  kinit: Generic preauthentication failure while getting initial credentials

If you did *not* encounter this error, congratulations - you must be
a disciplined reader of documentation!  To set an initial password
when creating a user via the ``ipa user-add`` command you must
supply the ``--password`` flag (the command will prompt for the
password).

Use the ``ipa passwd`` command to (re)set a user's password::

  [server]$ ipa passwd bob
  New Password:
  Enter New Password again to verify:
  ----------------------------------------
  Changed password for "bob@IPADEMO.LOCAL"
  ----------------------------------------

Whenever a user has their password reset (including the first time),
the next ``kinit`` will prompt them to enter a new password::

  [server]$ kinit bob
  Password for bob@IPADEMO.LOCAL: 
  Password expired.  You must change it now.
  Enter new password: 
  Enter it again: 


Now ``bob`` has a TGT (run ``klist`` to confirm) which can use to
log into other hosts and services.  Try logging into
``client.ipademo.local``::

  [server]$ ssh bob@client.ipademo.local
  -sh-4.3$

You are now logged into the client, as ``bob``.  Hit ``^D`` or type
``exit`` to log out and return to the ``server`` shell.  If you run
``klist`` again you will see not only the TGT but a *service ticket*
which was automatically acquired to log into
``client.ipademo.local`` without prompting for a password.  Kerberos
is a true *single sign-on* protocol!

::

  [server]$ klist
  Ticket cache: KEYRING:persistent:1000:krb_ccache_dYtyLyU
  Default principal: bob@IPADEMO.LOCAL

  Valid starting     Expires            Service principal
  15/10/15 07:15:11  16/10/15 07:15:02  host/client.ipademo.local@IPADEMO.LOCAL
  15/10/15 07:15:03  16/10/15 07:15:02  krbtgt/IPADEMO.LOCAL@IPADEMO.LOCAL



Module 4: Host-based access control
===================================

FreeIPA's *host-based access control* (HBAC) feature allows you to
define policies that restrict access to hosts or services based on
the user attempting to log in and that user's groups, the host which
they are trying to access (or its *host groups*), and (optionally)
the service being accessed.

In this module we will define an HBAC policy that will restrict
access to ``client.ipademo.local`` to members of the
``sysadmin`` user group.


Adding a host group
-------------------

Instead of defining the HBAC rule to directly talk about
``client.ipademo.local``, create a *host group* called
``webservers`` and make ``client.ipademo.local`` a member.

Explore the Web UI to work out how to do this, or use the CLI (you
will need to ``kinit admin``; see if you can work out what plugin
provides the host group functionality).

**Hint:** if you use the CLI will need to run two commands - one to
create the host group, and one to add ``client.ipademo.local`` as a
member.


Disabling the ``allow_all`` HBAC rule
-------------------------------------

HBAC rules are managed via the ``hbacrule`` plugin.  You can
complete the following actions via the Web UI as well, but we will
cover the CLI commands.

List the existing HBAC rules::

  [server]$ ipa hbacrule-find
  -------------------
  1 HBAC rule matched
  -------------------
    Rule name: allow_all
    User category: all
    Host category: all
    Service category: all
    Description: Allow all users to access any host from any
    host
    Enabled: TRUE
  ----------------------------
  Number of entries returned 1
  ----------------------------

The FreeIPA server is installed with a single default ``allow_all``
rule.  It needs to be disabled for other HBAC rules to have any
effect.  Look for a command that can do this, and run it.


Creating HBAC rules
-------------------

HBAC rules are built up incrementally.  The rule is created, then
users or groups, hosts or hostsgroups and HBAC services are added to
the rule.  The following transcript details the process::

  [server]$ ipa hbacrule-add sysadmin_webservers
  -------------------------------------
  Added HBAC rule "sysadmin_webservers"
  -------------------------------------
    Rule name: sysadmin_webservers
    Enabled: TRUE

  [server]$ ipa hbacrule-add-host sysadmin_webservers --hostgroup webservers
    Rule name: sysadmin_webservers
    Enabled: TRUE
    Host Groups: webservers
  -------------------------
  Number of members added 1
  -------------------------

  [server]$ ipa hbacrule-add-user sysadmin_webservers --group sysadmin
    Rule name: sysadmin_webservers
    Enabled: TRUE
    User Groups: sysadmin
    Host Groups: webservers
  -------------------------
  Number of members added 1
  -------------------------

  [server]$ ipa hbacrule-mod sysadmin_webservers --servicecat=all
  ----------------------------------------
  Modified HBAC rule "sysadmin_webservers"
  ----------------------------------------
    Rule name: sysadmin_webservers
    Service category: all
    Enabled: TRUE
    User Groups: sysadmin
    Host Groups: webservers

The ``--servicecat=all`` option applies this rule all services on
matching hosts.  It could have been set during the ``hbacrule-add``
command instead.


Testing HBAC rules
------------------

You can test HBAC rule evaluation using the ``ipa hbactest``
command::

  [server]$ ipa hbactest --user bob --host client.ipademo.local --service sshd
  ---------------------
  Access granted: False
  ---------------------
    Not matched rules: sysadmin_webservers

Poor ``bob``.  He won't be allowed in because he is not a member of
the ``sysadmin`` group.  What about ``alice``?

``kinit`` as ``bob`` and try to log into the client::

  [server]$ kinit bob
  Password for bob@IPADEMO.LOCAL: 
  [server]$ ssh bob@client.ipademo.local
  Connection closed by UNKNOWN

Then try ``alice``::

  [server]$ kinit alice
  Password for alice@IPADEMO.LOCAL: 
  [server]$ ssh alice@client.ipademo.local
  Last login: Fri Oct 16 01:09:10 2015 from 192.168.33.10
  -sh-4.3$ 


Module 5: Web App External Authentication
=========================================

You can configure many kinds of applications to rely on FreeIPA's
centralised authentication, including web applications.  In this
module you will configure Apache to use Kerberos authentication to
authenticate user, PAM to enforce HBAC rules and
``mod_lookup_identity`` to populate the request environment with
user attributes.

All activities in this module take place on ``client`` unless
otherwise specified.

**TODO**: ship the WSGI application and apache config OOTB


Create a service
----------------

Create a *service* representing the web application on
``client.ipademo.local``.  A service principal name has the service
type as its first part, separated from the host name by a slash,
e.g.  ``HTTP/www.example.com``.  The host part must correspond to an
existing host in the directory.

You must be getting the hang of FreeIPA by now, so I'll leave the
rest of this step up to you.  (It's OK to ask for help!)

**Note:** use the ``--force`` flag to force the service to be added
if FreeIPA complains that the *Host does not have corresponding DNS
A/AAAA record*.


Retrieve Kerberos keytab
------------------------

The service needs access to its Kerberos key in order to
authenticate users.  We retrieve the key from the FreeIPA server and
store it in *keytab* file::

