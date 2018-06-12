..
  Copyright 2015, 2016  Red Hat, Inc.

  This work is licensed under the Creative Commons Attribution 4.0
  International License. To view a copy of this license, visit
  http://creativecommons.org/licenses/by/4.0/.


Introduction
============

FreeIPA_ is a centralised identity management system.  In this
workshop you will learn how to deploy FreeIPA servers and enrol
client machines, define and manage user and service identities, set
up access policies, configure network services to take advantage of
FreeIPA's authentication and authorisation facilities and issue
X.509 certificates for services.

.. _FreeIPA: http://www.freeipa.org/page/Main_Page


Curriculum overview
-------------------

Mandatory:

- `Unit 1: Installing the FreeIPA server <1-server-install.rst>`_
- `Unit 2: Enrolling client machines <2-client-install.rst>`_
- `Unit 3: User management and Kerberos authentication <3-user-management.rst>`_
- `Unit 4: Host-based access control (HBAC) <4-hbac.rst>`_

Optional units—choose the topics that are relevant to you:

- `Unit 5: Web application authentication and authorisation <5-web-app-authnz.rst>`_
- `Unit 6: Service certificates <6-cert-management.rst>`_
- `Unit 7: Replica installation <7-replica-install.rst>`_
- `Unit 8: Sudo rule management <8-sudorule.rst>`_
- `Unit 9: SELinux User Maps <9-selinux-user-map.rst>`_
- `Unit 10: SSH user and host key management <10-ssh-key-management.rst>`_


Editing files on VMs
--------------------

Parts of the workshop involve editing files on virtual
machines.  The ``vi`` and GNU ``nano`` editors are available on the
VMs.  If you are not familiar with ``vi`` or you are unsure of what to use, you
should choose ``nano``.


Example commands
----------------

This guide contains many examples of commands.  Some of the commands
should be executed on your host, others on a particular guest VM.
For clarity, commands are annotated with the host on which they are
meant to be executed, as in these examples::

  $ echo "Run it on virtualisation host (no annotation)"

  [server]$ echo "Run it on FreeIPA server"

  [client]$ echo "Run it on IPA-enrolled client"

  ...


Preparation
===========

Some preparation is needed prior to the workshop.  The workshop is
designed to be carried out in a Vagrant_ environment that configures
three networked virtual machines (VMs) with all software needed for
the workshop.  **The goal of this preparation** is to ``vagrant up``
the VMs.  After this preparation is completed you are ready to begin
the workshop.

.. _Vagrant: https://www.vagrantup.com/


Requirements
------------

For the FreeIPA workshop you will need to:

- Install **Vagrant** and **VirtualBox**. (On Fedora, you can use **libvirt**
  instead of VirtualBox).

- Use Git to clone the repository containing the ``Vagrantfile``

- Fetch the Vagrant *box* for the workshop

- Add entries for the guest VMs to your hosts file (so you can
  access them by their hostname)

Please set up these items **prior to the workshop**.  More detailed
instructions follow.


Install Vagrant and VirtualBox
------------------------------

Fedora
^^^^^^

If you intend to use the ``libvirt`` provider (recommended), install
``vagrant-libvirt`` and ``vagrant-libvirt-doc``::

  $ sudo dnf install -y vagrant-libvirt vagrant-libvirt-doc

Also ensure you have the latest versions of ``selinux-policy`` and
``selinux-policy-targeted``.

Allow your regular user ID to start and stop Vagrant boxes using ``libvirt``.
Add your user to ``libvirt`` group so you don't need to enter your administrator
password everytime::

  $ sudo gpasswd -a ${USER} libvirt
  $ newgrp libvirt

On **Fedoda 28** you need to enable ``virtlogd``::

  $ systemctl enable virtlogd.socket
  $ systemctl start virtlogd.socket

Finally restart the services::

  $ systemctl restart libvirtd
  $ systemctl restart polkit

Otherwise, you will use VirtualBox and the ``virtualbox`` provider.
VirtualBox needs to build kernel modules, and that means that you must
first install kernel headers and Dynamic Kernel Module Support::

  $ sudo dnf install -y vagrant kernel-devel dkms

Next, install VirtualBox from the official VirtualBox package repository.
Before using the repo, check that its contents match what appears
in the transcript below (to make sure it wasn't tampered with)::

  $ sudo curl -o /etc/yum.repos.d/virtualbox.repo \
    http://download.virtualbox.org/virtualbox/rpm/fedora/virtualbox.repo

  $ cat /etc/yum.repos.d/virtualbox.repo
  [virtualbox]
  name=Fedora $releasever - $basearch - VirtualBox
  baseurl=http://download.virtualbox.org/virtualbox/rpm/fedora/$releasever/$basearch
  enabled=1
  gpgcheck=1
  repo_gpgcheck=1
  gpgkey=https://www.virtualbox.org/download/oracle_vbox.asc

  $ sudo dnf install -y VirtualBox-5.2

Finally, load the kernel modules (you may need to restart your system for this to work)::

  $ sudo modprobe vboxdrv vboxnetadp


Mac OS X
^^^^^^^^

Install Vagrant for Mac OS X from
https://www.vagrantup.com/downloads.html.

Install VirtualBox 5.2 for **OS X hosts** from
https://www.virtualbox.org/wiki/Downloads.

Install Git from https://git-scm.com/download/mac or via your
preferred package manager.


Debian / Ubuntu
^^^^^^^^^^^^^^^

Install Vagrant and Git::

  $ sudo apt-get install -y vagrant git

**Virtualbox 5.2** may be available from the system package manager,
depending your your release.  Find out which version of VirtualBox is
available::

  $ apt list virtualbox
  Listing... done
  virtualbox/bionic 5.2.10-dfsg-6 amd64

If version 5.2 is available, install it via ``apt-get``::

  $ sudo apt-get install -y virtualbox

If VirtualBox 5.2 was not available in the official packages for
your release, follow the instructions at
https://www.virtualbox.org/wiki/Linux_Downloads to install it.


Windows
^^^^^^^

Install Vagrant via the ``.msi`` available from
https://www.vagrantup.com/downloads.html.

Install VirtualBox 5.2 for **Windows hosts** from
https://www.virtualbox.org/wiki/Downloads.

You will also need to install an SSH client, and Git.  Git for
Windows also comes with an SSH client so just install Git from
https://git-scm.com/download/win.


Clone this repository
---------------------

This repository contains the ``Vagrantfile`` that is used for the
workshop, which you will need locally.

::

  $ git clone https://github.com/freeipa/freeipa-workshop.git


Fetch Vagrant box
-----------------

Please fetch the Vagrant box prior to the workshop.  It is > 600MB
so it may not be feasible to download it during the workshop.

::

  $ vagrant box add netoarmando/freeipa-workshop


Add hosts file entries
----------------------

*This step is optional.  All units can be completed using the CLI
only.  But if you want to access the FreeIPA Web UI or other web
servers on the VMs from your browser, follow these instructions.*

Add the following entries to your hosts file::

  192.168.33.10   server.ipademo.local
  192.168.33.11   replica.ipademo.local
  192.168.33.20   client.ipademo.local

On Unix systems (including Mac OS X), the hosts file is ``/etc/hosts``
(you need elevated permissions to edit it.)

On Windows, edit ``C:\Windows\System32\system\drivers\etc\hosts`` as
*Administrator*.


Next step
---------

You are ready to begin the workshop.  Continue to
`Unit 1: Installing the FreeIPA server <1-server-install.rst>`_.


After the workshop
------------------

Here are some contact details and resources that may help you after
the workshop is over:

- IRC: ``#freeipa`` and ``#sssd`` (Freenode)

- ``freeipa-users@lists.fedorahosted.org`` `mailing list
  <https://lists.fedoraproject.org/archives/list/freeipa-users@lists.fedorahosted.org/>`_

- `How To guides <https://www.freeipa.org/page/HowTos>`_: large
  index of articles about specialised tasks and integrations

- `Troubleshooting guide
  <https://www.freeipa.org/page/Troubleshooting>`_: how to debug
  common problems; how to report bugs

- `Bug tracker <https://pagure.io/freeipa>`_

- Information about the `FreeIPA public demo
  <https://www.freeipa.org/page/Demo>`_ instance

- `Deployment Recommendations
  <https://www.freeipa.org/page/Deployment_Recommendations>`_:
  things to consider when going into production

- `Documentation index
  <https://www.freeipa.org/page/Documentation>`_

- `FreeIPA Planet <http://planet.freeipa.org/>`_: aggregate of
  several FreeIPA and identity-management related blogs

- `GitHub organisation <https://github.com/freeipa>`_.  In addition
  to the `main repository <https://github.com/freeipa/freeipa>`_
  there are various tools, CI-related projects and documentation.

<<<<<<< HEAD
After ``ipa-replica-install`` finishes, the replica is operational.


Module 8: Managing public ssh-keys for users and hosts
======================================================

In this module you explore, how to use FreeIPA as a backend provider
for ssh-keys. Instead of using distributed ``authorized_keys`` and
``known_hosts`` files, ssh-keys are uploaded to their corresponding
user and host entries in FreeIPA.

Using FreeIPA as a backend store for ssh user-keys
--------------------------------------------------

OpenSSH uses ``public-private key pairs`` to authenticate users.
A user attempts to access some network resource and presents its key
pair. The machine then stores the user's public key in an
``authorized_keys`` file. Any time that the user attempts to access
the resource again, the machine simply checks its ``authorized_keys``
file and then grants access automatically to approved users. If the
target system does not share a common home directory, the user must
copy the public part of his SSH key to the target system he intends
to log in to. The public portion of the SSH key must be copied to
each target system the user intends to log in to.

In FreeIPA, the System Security Services Daemon (SSSD) can be
configured to cache and retrieve user SSH keys so that applications
and services only have to look in one location for user keys.
Because SSSD can use FreeIPA  as one of its identity information
providers, FreeIPA provides a universal and centralized repository
of keys. Administrators do not need to worry about distributing,
updating, or verifying user SSH keys.

Generate a user key on the client system::

  [client]$ sudo su - alice
  [alice@client]$
  [alice@client]$ ssh-keygen -C alice@ipademo.local
  Generating public/private rsa key pair.
  Enter file in which to save the key (/home/alice/.ssh/id_rsa):
  Created directory '/home/alice/.ssh'.
  Enter passphrase (empty for no passphrase):
  Enter same passphrase again:
  Your identification has been saved in /home/alice/.ssh/id_rsa.
  Your public key has been saved in /home/alice/.ssh/id_rsa.pub.
  The key fingerprint is:
  SHA256:rgVSyPM/yn/b5bsZQIsAXWF+16zkP59VS9GS+k+bbOg alice@ipademo.local

Copy the base64-encoded public key and upload it to Alice
user entry in the FreeIPA system::

  [alice@client]$ kinit alice
  Password for alice@IPADEMO.LOCAL:

  [alice@client]$ cat /home/alice/.ssh/id_rsa.pub
  ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAA[...]alice@ipademo.local

  [alice@client]$ ipa user-mod alice --sshpubkey="ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAA[...]alice@ipademo.local"

During the configuration of the systems, the Security
Services Daemon (SSSD) has already been configured to use
FreeIPA as one of its identity domains and OpenSSH has been
setup to use SSSD for managing user keys.

If you have disabled the ``allow_all`` HBAC rule, add a new rule
that will **allow ``alice`` to access the ``sshd`` service on any
host**.

Logging in to the server using the previously created ``ssh pubkey``
should now work out of the box::

  [alice@client]$ ssh -o GSSAPIAuthentication=no server.ipademo.local
  Last login: Tue Feb  2 15:10:13 2016
  [alice@server]$

To verify the ssh pubkey has been used for authentication, you can
check the sshd service journal on the server, which should have a
similar entry like this one::

  server.ipademo.local sshd[19729]: Accepted publickey for alice from 192.168.33.20 port 37244 \
    ssh2: RSA SHA256:rgVSyPM/yn/b5bsZQIsAXWF+16zkP59VS9GS+k+bbOg


Using FreeIPA as a backend store for ssh host-keys
--------------------------------------------------

OpenSSH uses public keys to authenticate hosts. One machine attempts to
access another machine and presents its key pair. The first time the host
authenticates, the administrator on the target machine has to approve the
request manually. The machine then stores the host's public key in a
``known_hosts`` file. Any time that the remote machine attempts to access the
target machine again, the target machine simply checks its ``known_hosts``
file and then grants access automatically to approved hosts.

Based on the last exercise, try to figure out how you can upload ssh
host-keys to the FreeIPA server. **Note: OpenSSH has already been
configured to lookup keys on the FreeIPA server, so no manual configuration
is required for this service.**

