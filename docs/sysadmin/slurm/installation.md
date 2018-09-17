# Install MUNGE

Install `rpmbuild`

```bash
yum install rpm-build
```

Download MUNGE
```
wget https://github.com/dun/munge/releases/download/munge-0.5.13/munge-0.5.13.tar.xz
```

Install dependencies for MUNGE

```bash
yum install gcc gcc-c++ bzip2-devel openssl-devel zlib-devel
```

Build rpm files and install 

```bash
rpmbuild -tb --clean munge-0.5.13.tar.xz 
rpm -ivh munge-0.5.13-1.el7.x86_64.rpm munge-libs-0.5.13-1.el7.x86_64.rpm munge-devel-0.5.13-1.el7.x86_64.rpm 
```

Create MUNGE credential in `/etc/munge/munge.key` 

```bash
dd if=/dev/urandom bs=1 count=1024 > /etc/munge/munge.key 
```

This file should be permissioned `400` and owned by the user that the munged daemon will run as. Securely propagate this file (e.g., via ssh) to all other hosts within the same security realm.

Start MUNGE daemon

```bash
systemctl start munge
systemctl status munge
```

Testing installation 
```
munge -n
munge -n | unmunge
munge -n | ssh <host> unmunge
remunge
```
# Install SLURM

Download SLURM and untar

```bash
wget https://download.schedmd.com/slurm/slurm-17.11.8.tar.bz2
tar --bzip -x -f slurm-17.11.8.tar.bz2
```

Install dependencies

```bash
yum install readline-devel perl-ExtUtils-MakeMaker 
yum install pam-devel                              # PAM support 
yum install hwloc hwloc-devel                      # cgroup Task Affinity
yum install mysql++-devel                          # MySQL accounting
yum install lua-devel                              # Lua support
yum install man2html                               # Documentation           
```

Build and install SLURM rpm package

```bash
rpmbuild -ta slurm-17.11.8.tar.bz2 
```

**Head Node**: Installation

```bash
rpm --install slurm-17.11.8-1.el7.x86_64.rpm slurm-perlapi-17.11.8-1.el7.x86_64.rpm slurm-slurmctld-17.11.8-1.el7.x86_64.rpm 

systemctl enable slurmctld
systemctl start slurmctld
systemctl status slurmctld
```

**Compute Node**: Installation

```bash
rpm --install slurm-17.11.8-1.el7.x86_64.rpm slurm-perlapi-17.11.8-1.el7.x86_64.rpm slurm-slurmd-17.11.8-1.el7.x86_64.rpm slurm-pam_slurm-17.11.8-1.el7.x86_64.rpm 

systemctl enable slurmd
systemctl start slurmd
systemctl status slurmd
```

**SlurmDBD Node**: Installation

```bash
rpm --install slurm-17.11.8-1.el7.x86_64.rpm slurm-slurmdbd-17.11.8-1.el7.x86_64.rpm 

systemctl enable slurmdbd
systemctl start slurmdbd
systemctl status slurmdbd
```

Add SLURM port to firewall

```bash
firewall-cmd --add-port 6817/tcp --permanent
firewall-cmd --add-port 6818/tcp --permanent
firewall-cmd --add-port 60001-63000/tcp --permanent # If set SrunPortRange=60001-63000
```

# Setup PAM

In compute node, add `pam_slurm.so` to `/etc/pam.d/sshd`. For example:

```
#%PAM-1.0
auth       required     pam_sepermit.so
auth       substack     password-auth
#auth       include      postlogin
# Used with polkit to reauthorize users in remote sessions
#-auth      optional     pam_reauthorize.so prepare
account    required     pam_slurm.so
account    required     pam_nologin.so
account    include      password-auth
password   include      password-auth
# pam_selinux.so close should be the first session rule
session    required     pam_selinux.so close
session    required     pam_loginuid.so
...
```

# Troubleshooting

* Log location: `/var/log/slurmctld.log`

* Possible missing files/directories in head node
  * `/var/lib/slurm`
  * `/var/run/slurmctld.pid`
  * `/var/spool/slurm/ctld`

* Possible missing files/directories in compute nodes
  * `/var/spool/slurm/d`

  Change ownership using

  ```bash
  chown -R slurm:slurm <file>
  ```

* Installing example configuration (`slurm.conf`)

  ```
  rpm --install slurm-example-configs-17.11.8-1.el7.x86_64.rpm
  ```

# Example `slurm.conf`

```
ClusterName=linux
ControlMachine=nstdahpc03.nstda.or.th
#ControlAddr=
#BackupController=
#BackupAddr=
#
SlurmUser=slurm
#SlurmdUser=root
SlurmctldPort=6817
SlurmdPort=6818
SrunPortRange=60001-63000
AuthType=auth/munge
#JobCredentialPrivateKey=
#JobCredentialPublicCertificate=
StateSaveLocation=/var/spool/slurm/ctld
SlurmdSpoolDir=/var/spool/slurm/d
SwitchType=switch/none
MpiDefault=none
SlurmctldPidFile=/var/run/slurmctld.pid
SlurmdPidFile=/var/run/slurmd.pid
ProctrackType=proctrack/pgid
#PluginDir=
#FirstJobId=
ReturnToService=0
#MaxJobCount=
#PlugStackConfig=
#PropagatePrioProcess=
#PropagateResourceLimits=
#PropagateResourceLimitsExcept=
#Prolog=
#Epilog=
#SrunProlog=
#SrunEpilog=
#TaskProlog=
#TaskEpilog=
#TaskPlugin=
#TrackWCKey=no
#TreeWidth=50
#TmpFS=
UsePAM=1
#
# TIMERS
SlurmctldTimeout=300
SlurmdTimeout=300
InactiveLimit=0
MinJobAge=300
KillWait=30
Waittime=0
#
# SCHEDULING
SchedulerType=sched/backfill
#SchedulerAuth=
#SelectType=select/linear
FastSchedule=1
#PriorityType=priority/multifactor
#PriorityDecayHalfLife=14-0
#PriorityUsageResetPeriod=14-0
#PriorityWeightFairshare=100000
#PriorityWeightAge=1000
#PriorityWeightPartition=10000
#PriorityWeightJobSize=1000
#PriorityMaxAge=1-0
#
# LOGGING
SlurmctldDebug=3
SlurmctldLogFile=/var/log/slurmctld.log
SlurmdDebug=3
SlurmdLogFile=/var/log/slurmd.log
JobCompType=jobcomp/none
#JobCompLoc=
#
# ACCOUNTING
#JobAcctGatherType=jobacct_gather/linux
#JobAcctGatherFrequency=30
#
#AccountingStorageType=accounting_storage/slurmdbd
#AccountingStorageHost=
#AccountingStorageLoc=
#AccountingStoragePass=
#AccountingStorageUser=
#
# COMPUTE NODES
NodeName=nstdahpc04 NodeAddr=nstdahpc04.nstda.or.th Procs=2 State=UNKNOWN
PartitionName=debug Nodes=ALL Default=YES MaxTime=INFINITE State=UP
```

