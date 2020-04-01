# lsf-slurm-wrappers

These are a set of wrapper scripts to common Slurm commands that execute LSF commands in the background.  The scripts are intended as a migration aid for customers migrating from Slurm to LSF and not as a replacement for the LSF commands.

## Abstract

* Slurm Command Wrapper
* Product/Component Release: 10.1
* First Public Release: 01 April 2020

## Contents

1. Abbreviations
2. About Slurm command wrapper
3. Supported operating systems
4. Contributing Changes
5. Wrapper command usage notes
6. Installation and configuration
7. Copyright
8. License

## Abbreviations

Slurm: Simple Linux Utility for Resource Management


## About Slurm command wrapper

The Slurm Command Wrapper provides a way for Slurm users, to submit and manage jobs in LSF cluster environment, with Slurm command syntax and options.

Commands included: 

* srun 
* sbatch 
* squeue 
* scontrol 
* sinfo
* scancel


Not all Slurm command options are supported.  If you don't find an option that you like, see the second Contributing Changes below.

## Supported operating systems

* RHEL 7 x86_64 and later
* Generally any operating system supporting LSF and Python 2.7+

## Contributing Changes

Changes from the broader Linux community are welcomed.  If you would like to make a contribution to these scripts, simply open a pull request and in the body of the request, follow the DCO process documented in this and other IBMSpectrumComputing repositories.  You essentially must "SignOff" using your real email address.

## Wrapper command usage notes

### srun - Run a parallel job.

Supported options:

```shell
Usage: srun [-N nnodes] [-n ntasks] [-i in] [-o out] [-e err] [-p partition]
            [--hold] [-t minutes] [-J jobname] [--open-mode=a|A|t|T] [-m dist]
            [--cpu_bind=...] [--mem_bind=...] [--ntasks-per-socket=n]
            [--ntasks-per-core=n] [--sockets-per-node=S] [--cores-per-socket=C]
            [--threads-per-core=T] [--ntasks-per-node=n] [--mail-user=user]
            executable [args...]
``` 


### sbatch - Submit a batch script.

Supported options:

```shell
Usage: sbatch [-N nnodes] [-n ntasks] [-p partition] [--hold]
              [-t minutes] [-D path] [--workdir=directory]
              [-J jobname] [--mail-user=user] [--ntasks-per-node=n]
              [--export[=names]] executable [args...]
```


### squeue - Submit a batch script.

Supported options:

```shell
Usage: srun [-N nnodes] [-n ntasks] [-i in] [-o out] [-e err] [-p partition]
            [--hold] [-t minutes] [-J jobname] [--open-mode=a|A|t|T] [-m dist]
            [--cpu_bind=...] [--mem_bind=...] [--ntasks-per-socket=n]
            [--ntasks-per-core=n] [--sockets-per-node=S] [--cores-per-socket=C]
            [--threads-per-core=T] [--ntasks-per-node=n] [--mail-user=user]
            executable [args...]
``` 


### scontrol - Run a parallel job.

Supported options:


```shell
scontrol  [<OPTION>] [<COMMAND&>]
    Valid <OPTION> values are:
     -h or --help: equivalent to "help" command

    Valid <COMMAND> values are:
     resume <jobid_list>        resume previously suspended job (see suspend)
     suspend <jobid_list>       susend specified job (see resume)
     requeue <jobid_list>       re-queue a batch job
     release <jobid_list>       permit specified job to start (see hold)
     hold <jobid_list>          prevent specified job from starting.
                                <jobid_list> argument is
                                a comma separated list of job IDs.
```


### sinfo - Report resource state.

Supported options:

```shell
Usage: sinfo [-alN] [-p partition]
```


### scancel - Cancel the pending or running job.

Supported options:

```shell
Usage: scancel [-n job_name] [-p partition] [--usage]
               [-u user_name] [-w hosts...] [job_id]
```


### Examples of command wrapper on LSF cluster

1. Job submission: sbatch, srun
 
sbatch example:

```shell
[hosta] # sbatch -n4 --ntasks-per-node=2 ./job.sh
Submitted batch job 695
[hosta] #
``` 

The job 695 requires 4 tasks in total and 2 tasks per node, to LSF cluster.

```shell
srun example:
[hosta] # srun -n4 --ntasks-per-node=2 ./job.sh
Thu Sep 1 01:12:51 EDT 2016: 0
Thu Sep 1 01:12:51 EDT 2016: 0
hosta
hosta
Thu Sep 1 01:12:53 EDT 2016: 0
Thu Sep 1 01:12:53 EDT 2016: 0
hostb
hostb
Thu Sep 1 01:12:53 EDT 2016: 1
Thu Sep 1 01:12:53 EDT 2016: 1
hosta
hosta
Thu Sep 1 01:12:55 EDT 2016: 1
Thu Sep 1 01:12:55 EDT 2016: 1
hostb
hostb
...
```

```shell
job.sh:
#!/bin/bash

((i=0))
while ((i<10));
do
        sleep 2
        echo `date`: $i
        hostname
        ((i=i+1))
done
```


2. Job management: squeue, scancel, scontrol

squeue example:

```shell
[hosta] # squeue
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
               696    normal ./job.sh  user1  R       5(s)      2 hosta,hostb 
[hosta] #
``` 

Compare this output to the LSF bjobs command output:

```shell
[hosta] # bjobs
JOBID   USER    STAT  QUEUE      FROM_HOST   EXEC_HOST   JOB_NAME   SUBMIT_TIME
696     user1 RUN   normal     hosta         hosta       ./job.sh   Sep  1 01:03
                                             hosta
                                             hostb
                                             hostb
[hosta] # 
```

By default, job output is named slurm-696.out, and saved in the current directory.

scancel example:

```shell
[hosta] # squeue
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
               697 interactive    sleep  user1  R      11(s)      1 hostb
[hosta] # scancel 697
[hosta] # squeue
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
[hosta] # 
```


scontrol example:

Suspend a running job, and then resume it:

```shell
[hosta] # bjobs
JOBID   USER    STAT  QUEUE      FROM_HOST   EXEC_HOST   JOB_NAME   SUBMIT_TIME
1421    llbld   RUN   normal     hosta       hosta       sleep 999  Oct 19 22:20
[hosta] # scontrol suspend 1421
[hosta] # bjobs
JOBID   USER    STAT  QUEUE      FROM_HOST   EXEC_HOST   JOB_NAME   SUBMIT_TIME
1421    llbld   USUSP normal     hosta       hosta       sleep 999  Oct 19 22:20
[hosta] # scontrol resume 1421
[hosta] # bjobs
JOBID   USER    STAT  QUEUE      FROM_HOST   EXEC_HOST   JOB_NAME   SUBMIT_TIME
1421    llbld   RUN   normal     hosta       hosta       sleep 999  Oct 19 22:20
```

- Resource status: sinfo

Example:

```shell
[hosta] # sinfo
PARTITION    AVAIL  TIMELIMIT    NODES      STATE NODELIST
priority        up   infinite        2       idle all
admin           up   infinite        2       idle all
interactive*    up   infinite        2       idle all
normal*         up   infinite        2       idle all
night           up        5.0        2       idle all
test1           up       20.0        2       idle hosta,hostb
idle            up   infinite        2       idle all
test2           up       25.0        1       idle hosta
short           up   infinite        2       idle all
owners          up   infinite        2       idle all
test3           up       30.0        2       idle hosta,hostb
[hosta] #
``` 

Compare this output to the LSF bqueues command output:

```shell
[hosta] # bqueues
QUEUE_NAME      PRIO STATUS          MAX JL/U JL/P JL/H NJOBS  PEND   RUN  SUSP
admin            50  Open:Active       -    -    -    -     0     0     0     0
owners           43  Open:Active       -    -    -    -     0     0     0     0
priority         43  Open:Active       -    -    -    -     0     0     0     0
night            40  Open:Active       -    -    -    -     0     0     0     0
short            35  Open:Active       -    -    -    -     0     0     0     0
test1            35  Open:Active       -    -    -    -     0     0     0     0
test2            35  Open:Active       -    -    -    -     0     0     0     0
test3            35  Open:Active       -    -    -    -     0     0     0     0
normal           30  Open:Active       -    -    -    -     0     0     0     0
interactive      30  Open:Active       -    -    -    -     0     0     0     0
idle             20  Open:Active       -    -    -    -     0     0     0     0
[hosta] # 	
```

## Installation and Configuration

1. Before installation
	(LSF_TOP=Full path to the top-level installation directory of LSF.)

	1) Log on to the LSF master host as root
	2) Set your environment:

	- For csh or tcsh: % source LSF_TOP/conf/cshrc.lsf
	- For sh, ksh, or bash: $ . LSF_TOP/conf/profile.lsf

2. Installation steps

	1) Download the command wrappers and make them executable, for example:
	
	```cmd
	chmod 755 srun
	```

	2) Copy the command wrappers to the binary directory $LSF_BINDIR/

3. Ensure LSF is installed and working properly.

4. You are now able to use the srun, sbatch, squeue, scontrol, sinfo, and scancel commands.


## Copyright

© Copyright IBM Corporation 2016

## License

Apache License

Version 2.0, January 2004

http://www.apache.org/licenses/ 

TERMS AND CONDITIONS FOR USE, REPRODUCTION, AND DISTRIBUTION.  For details check the LICENSE file in the root of this repository.
