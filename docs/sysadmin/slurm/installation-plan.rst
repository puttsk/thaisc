==================
Installation Plan
==================

SLURM installation plan for Tara cluster. 

Timeline
==================

Wednesday meeting: Set weekly goals and progress update? 

===================  =============== 
Important Date       Plan            
===================  =============== 
16 Oct 2018          Initial meeting. 
24 Oct 2018          **Implementation**: Prepare the machines to the :ref:`prerequisites` state and **document** for initializing the machines. 
                     
                     **Testing**: Prepare the plan and, if possible, automate script to test initail services in :ref:`prerequisites`.
31 Oct 2018          **Implementation**: 
                     
                     **Testing**: 
07 Nov 2018          **Implementation**: 
                     
                     **Testing**:                      
14 Nov 2018          **Implementation**: 
                     
                     **Testing**:                      
21 Nov 2018          **Implementation**: 
                     
                     **Testing**:                      
28 Nov 2018          **Implementation**: 
                     
                     **Testing**:                      
===================  =============== 

Teams
===========

================  =========
Implementation    Testing 
================  =========
\-                \-
================  =========


General Information
=====================

| **SLURM Version**: 18.08.1 (Oct 8, 2018)
| **MUNGE Version**: 0.5.13 (Sep 27, 2017)

| **SLURM user**: ``slurm``
| **SLURM user environment variable**: ``SLURMUSER``
| **SLURM UID**: ``982``
| **SLURM group**: ``slurm``
| **SLURM GID**: ``982``

| **MUNGE user**: ``munge``
| **MUNGE user environment variable**: ``MUNGEUSER``
| **MUNGE UID**: ``981``
| **MUNGE group**: ``munge``
| **MUNGE GID**: ``981``

.. _prerequisites:

Prerequisites
=====================

* A set of virtual machines with following components installed 

    * *Clean* installation of CentOS 7. (If possible, the initial state of Tara cluster machines.)
    * ``freeIPA``.
    * ``NTP`` (Should be installed as a part of ``freeIPA``)
    * An NFS storage mounted to all machines. (Can we create a virtual HDD and mount it to all machine?)
    * Name resolution mechanism either through DNS or ``/etc/hosts`` files. 

* A Unix user **munge** for use by ``munge`` on all nodes of the cluster. 

* A Unix user **slurm** for use by ``slurmctld`` on all nodes of the cluster. 

TODO
===================

* Install and config MUNGE on all nodes
* Build SLURM RPMs and install on each node as shown in :ref:`slurm-services`. To enable required plugins see. :ref:`slurm-plugins` for the list of additional libraries. 

    * Set ``SlurmUser`` in the ``slurm.conf`` configuration file.     
    * Files and directories used by ``slurmctld`` will need to be readable or writable by the user SlurmUser (the Slurm configuration files must be readable; the log file directory and state save directory must be writable).
    * The parent directories for Slurm's log files, process ID files, state save directories, etc. **are not created by Slurm**. They must be created and made writable by SlurmUser as needed prior to starting Slurm daemons.

* Configure SLURM PAM module to limit access to allocated compute nodes. 

    * On job termination, any processes initiated by the user outside of Slurm's control may be killed using an ``Epilog`` script configured in ``slurm.conf``.

.. _slurm-services:

SLURM Services
=====================

================  ==========
Node Class        Services
================  ==========
Controller (VM)   ``slurm``, ``slurm-perlapi``, ``slurm-slurmctld``
Compute           ``slurm``, ``slurm-perlapi``, ``slurm-slurmd``
Frontend          ``slurm``, ``slurm-perlapi``
SlurmDBD (VM)     ``slurm``, ``slurm-dbd``
================  ==========

.. _slurm-plugins:

Plugins Dependencies 
======================

| List of plugins and their dependencies to be installed when building SLURM RPM packages. 
| *Need to check that the package contains these plugins after installing*

============================  =====================
Plugins                       Dependencies        
============================  =====================
**MUNGE**                     ``munge-devel``     
**PAM Support**               ``pam-devel``       
**cgroup Task Affinity**      ``hwloc-devel``     
**cgroup NUMA Affinity**      ???                 
**IPMI Engergy Consumption**  ``freeimpi-devel``  
**InfiniBand Accounting**     ``libibmad-devel``, ``libibumad-devel`` 
**Lua Support**               ``lua-devel``       
**My SQL Support**            ``mysql-devel``     
============================  =====================

Configuration
==================

===================  =======================  ==========
Config               Value                    Detail
===================  =======================  ==========
**AuthType**         *munge*
**CryptoType**       *munge* 
**PriorityType**     *priority/multifactor*    See. `Multifactor plugin <https://slurm.schedmd.com/priority_multifactor.html>`_
**SchedType**        *backfill*
**SelectType**                                 Might require ``cons_res`` plugin See. `Consumable Resources in Slurm <https://slurm.schedmd.com/cons_res.html>`_ 
===================  =======================  ==========

Node Information 
=================

============  =============  ==============================  ===========
Node Class    NodeName       Host Name                       Notes
============  =============  ==============================  ===========
freeipa       \-             freeipa.tara.nstda.or.th
slurmctrl     slurmctrl      slurmctld.tara.nstda.or.th
slurmdbd      slurmdbd       slurmdbd.hpc.nstda.or.th 
mysql         \-             mysql.hpc.nstda.or.th           MySQL or MariaDB ? 
frontend      \-             tara.nstda.or.th
compute       compute[1-8]   compute[1-8].tara.nstda.or.th  
memory        memory[1-2]    memory[1-2].tara.nstda.or.th    FAT nodes
============  =============  ==============================  ===========

| **NodeName**: The name used by all Slurm tools when referring to the node
| **NodeAddr**: The name or IP address Slurm uses to communicate with the node
| **NodeHostname**: The name returned by the command ``/bin/hostname -s``

``slurm.conf``
---------------

.. code:: bash

    NodeName=compute[1-8] CPUs=2 RealMemory=2G Sockets=1 CoresPerSocket=2 ThreadsPerCore=1 State=UNKNOWN 
    NodeName=memory[1-2] CPUs=4 RealMemory=4G Sockets=1 CoresPerSocket=4 ThreadsPerCore=1 State=UNKNOWN 

.. warning:: Changes in node configuration (e.g. adding nodes, changing their processor count, etc.) require restarting both the ``slurmctld`` daemon and the ``slurmd`` daemons.

Partitions
============


Multifactor Priority Settings
==============================



MPI Support
==============

We will support only MPI libraries and versions that support ``PMIx`` APIs as follow

* OpenMPI
* MPICH (version 3) (Do we need MPICH2 ?)
* IntelMPI

Notes
===================

