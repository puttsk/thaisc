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
31 Oct 2018          **Implementation**: Implementation document for SLURM
                     
                     **Testing**: Plan and testing script for SLURM. 
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
P Ar+, Arm        Best, Oat, Eee
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
    * ``nhc`` Node health check. Installed from `nhc <https://github.com/mej/nhc>`_
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

=========================  =======================  ==========
Config                     Value                    Detail
=========================  =======================  ==========
**AuthType**               *munge*
**CryptoType**             *munge* 
**PriorityType**           *priority/multifactor*   See. `Multifactor plugin <https://slurm.schedmd.com/priority_multifactor.html>`_
**SchedType**              *backfill*
**SelectType**             *select/cons_res*        See. `Consumable Resources in Slurm <https://slurm.schedmd.com/cons_res.html>`_ 
**SelectTypeParameters**  
**PreemptMode**      
**TopologyPlugin**                                  Should consider using plugin for tree topology
**HealthCheckProgram**     ``/usr/sbin/nhc``        For ``nhc`` See. `[1] <https://wiki.fysik.dtu.dk/niflheim/Slurm_configuration#node-health-check>`_ and `[2] <https://slurm.schedmd.com/SUG14/node_health_check.pdf>`_
**HealthCheckInterval**    *3600*                   
**HealthCheckNodeState**   *ANY*                    
=========================  =======================  ==========

Node Configuration (Testing System)
===================================

============  =============  =============================  ===========
Node Class    NodeName       Host Name                      Notes
============  =============  =============================  ===========
freeipa       \-             freeipa.hpc.nstda.or.th        VM
slurmctrl     slurmctrl      slurmctld.hpc.nstda.or.th      VM
slurmdbd      slurmdbd       slurmdbd.hpc.nstda.or.th       VM
mysql         \-             mysql.hpc.nstda.or.th          VM, MySQL or MariaDB ? 
frontend      \-             tara.nstda.or.th
compute       compute[1-8]   compute[1-8].hpc.nstda.or.th 
memory        memory[1-2]    memory[1-2].hpc.nstda.or.th    FAT nodes
dgx           dgx[1-2]       dgx[1-2].hpc.nstda.or.th       dgx1 is reserved. 
============  =============  =============================  ===========

.. warning:: Changes in node configuration (e.g. adding nodes, changing their processor count, etc.) require restarting both the ``slurmctld`` daemon and the ``slurmd`` daemons.


| **NodeName**: The name used by all Slurm tools when referring to the node
| **NodeAddr**: The name or IP address Slurm uses to communicate with the node
| **NodeHostname**: The name returned by the command ``/bin/hostname -s``
|
| **TmpDisk**: Total size of temporary disk storage in **TmpFS** in megabytes (e.g. "16384"). *TmpFS* (for "Temporary File System") identifies the location which jobs should use for temporary storage. Note this does not indicate the amount of free space available to the user on the node, only the total file system size. *The system administration should ensure this file system is purged as needed so that user jobs have access to most of this space.* The Prolog and/or Epilog programs (specified in the configuration file) might be used to ensure the file system is kept clean. 

``slurm.conf``
---------------

.. code:: bash

    NodeName=compute[1-8] CPUs=2 RealMemory=2048 Sockets=1 CoresPerSocket=2 ThreadsPerCore=1 State=UNKNOWN 
    NodeName=memory[1-2] CPUs=4 RealMemory=4096 Sockets=1 CoresPerSocket=4 ThreadsPerCore=1 State=UNKNOWN 
    NodeName=dgx[1-2] CPUs=2 RealMemory=2048 Sockets=1 CoresPerSocket=2 ThreadsPerCore=1 Gres=gpu:volta:8 State=UNKNOWN 


Partitions (Testing System)
===========================
===============  =============  ==========  =====  ===========
Partition        AllocNodes     MaxTime     State  Additional Parameters
===============  =============  ==========  =====  ===========
debug (default)  compute[1-2]    02:00:00   UP     DefaultTime=00:30:00
standby          compute[1-8]   120:00:00   UP
memory           memory[1-2]    120:00:00   UP
dgx              dgx2           120:00:00   UP     OverSubscribe=EXCLUSIVE
biobank          dgx1           UNLIMITED   UP     AllowGroups=biobank OverSubscribe=EXCLUSIVE
===============  =============  ==========  =====  ===========

| **AllowAccounts**: Comma separated list of accounts which may execute jobs in the partition. The default value is "ALL" 
| **AllowGroups**: Comma separated list of group names which may execute jobs in the partition. If at least one group associated with the user attempting to execute the job is in AllowGroups, he will be permitted to use this partition. Jobs executed as user root can use any partition without regard to the value of AllowGroups.
| **AllowQos**: Comma separated list of Qos which may execute jobs in the partition. Jobs executed as user root can use any partition without regard to the value of AllowQos.
| **OverSubscribe**: Controls the ability of the partition to execute more than one job at a time on each resource. Jobs that run in partitions with ``OverSubscribe=EXCLUSIVE`` will have exclusive access to all allocated nodes.

``slurm.conf``
---------------

.. code:: bash

    PartitionName=debug Nodes=compute[1-2] Default=YES MaxTime=02:00:00 DefaultTime=00:30:00 State=UP
    PartitionName=standby Nodes=compute[1-8] MaxTime=120:00:00 State=UP
    PartitionName=memory Nodes=memory[1-2] MaxTime=120:00:00 State=UP
    PartitionName=dgx Nodes=dgx2 MaxTime=120:00:00 State=UP OverSubscribe=EXCLUSIVE
    PartitionName=biobank Nodes=dgx1 MaxTime=120:00:00 State=UP AllowGroups=biobank OverSubscribe=EXCLUSIVE

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

