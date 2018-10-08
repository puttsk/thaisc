=================
Installation
=================

.. only:: html

    .. contents::

Install MUNGE
==============

Install prerequisites

.. code:: bash

    yum install rpm-build gcc gcc-c++ bzip2-devel openssl-devel zlib-devel

Download MUNGE from its `repository <https://github.com/dun/munge>` 

.. code:: bash
    
    wget https://github.com/dun/munge/releases/download/munge-0.5.13/munge-0.5.13.tar.xz

Build rpm files and install 

.. code:: bash
    
    rpmbuild -tb --clean munge-0.5.13.tar.xz 
    rpm -ivh munge-0.5.13-1.el7.x86_64.rpm munge-libs-0.5.13-1.el7.x86_64.rpm munge-devel-0.5.13-1.el7.x86_64.rpm 

Create MUNGE credential in ``/etc/munge/munge.key``

.. code:: bash

    dd if=/dev/urandom bs=1 count=1024 > /etc/munge/munge.key 

This file permission should be ``400`` and owned by the user that the munged daemon will run as. 
Securely propagate this file (e.g., via ssh) to all other hosts within the same security realm.

Start MUNGE daemon

.. code:: bash

    systemctl start munge
    systemctl status munge

Testing installation 

.. code:: bash

    munge -n
    munge -n | unmunge
    munge -n | ssh <host> unmunge
    remunge

Install SLURM
==============

