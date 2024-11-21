---
title: "Using resources effectively"
teaching: 15
exercises: 10
questions:
- "How do we monitor our jobs?"
- "How can I get my jobs scheduled more easily?" 
objectives:
- "Understand how to look up job statistics and profile code."
- "Understand job size implications."
keypoints:
- "The smaller your job, the faster it will schedule."
---

We now know virtually everything we need to know about getting stuff on a cluster. We can log on,
submit different types of jobs, use pre-installed software, and install and use software of our own.
What we need to do now is use the systems effectively.

## Estimating required resources using the scheduler

Although we covered requesting resources from the scheduler earlier, how do we know what type of
resources the software will need in the first place, and the extent of its demand for each?

Unless the developers or prior users have provided some idea, we don't. Not until we've tried it
ourselves at least once. We'll need to benchmark our job and experiment with it before we know how
how great its demand for system resources.

> ## Read the documentation
>
> Most HPC facilities maintain documentation as a wiki, website, or a document sent along when
> you register for an account. Take a look at these resources, and search for the software of
> interest: somebody might have written up guidance for getting the most out of it.
{: .discussion}

The most effective way of figuring out the resources required for a job to run successfully needs is
to submit a test job, and then ask the scheduler about its impact using `sacct`.

```
{{ site.remote.prompt }} sacct
```
{: .bash}

This shows all the jobs we ran recently (note that there are multiple entries per job). To get
info about a specific job, we change command slightly.

```
{{ site.remote.prompt }} sacct -j 727107
```
{: .bash}

This shows a lot of information, even more if you use the long display option,
`-l`. Plato also has a convenience command, `seff`, that gives a terser output.

```
{{ site.remote.prompt }} seff 727107
```
{: .bash}

```
Job ID: 727107
Cluster: plato
User/Group: nsid/nsid
State: COMPLETED (exit code 0)
Nodes: 1
Cores per node: 4
CPU Utilized: 00:00:04
CPU Efficiency: 25.00% of 00:00:16 core-walltime
Job Wall-clock time: 00:00:04
Memory Utilized: 1.12 MB
Memory Efficiency: 0.05% of 2.00 GB
```
{: .output}

Some interesting fields include the following:

* **Hostname**: Where did your job run?
* **MaxRSS**: What was the maximum amount of memory used?
* **Elapsed**: How long did the job take?
* **State**: What is the job currently doing/what happened to it?
* **MaxDiskRead**: Amount of data read from disk.
* **MaxDiskWrite**: Amount of data written to disk.

You can use this knowledge to set up the next job with a close estimate of its load on the system. A
good general rule is to ask the scheduler for 20% to 30% more time and memory than you expect the
job to need. This ensures that minor fluctuations in run time or memory use will not result in your
job being cancelled by the scheduler. Keep in mind that if you ask for too much, your job may not
run even though enough resources are available, because the scheduler will be waiting to match what
you asked for.

## Measuring the statistics of currently running tasks

### Connecting to Nodes

Typically, clusters allow users to connect to compute nodes where they have
running jobs. This is useful to check on a running job and see how it's doing.
To know which node to connect to, use `{{ site.sched.status }}`. Then, run `ssh
nodename`. Once you are on the node of interest, use programs such as `top` or
`ps`, as described below.

### Monitor system processes with `top`

The most reliable way to check current system stats is with `top`. Some sample output might look
like the following (type `q` to exit `top`):

```
{{ site.remote.prompt }} top
```
{: .bash}

{% include {{ site.snippets }}/resources/monitor-processes-top.snip %}

Overview of the most important fields:

* `PID`: What is the numerical id of each process?
* `USER`: Who started the process?
* `RES`: What is the amount of memory currently being used by a process (in bytes)?
* `%CPU`: How much of a CPU is each process using? Values higher than 100 percent indicate that a
  process is running in parallel.
* `%MEM`: What percent of system memory is a process using?
* `TIME+`: How much CPU time has a process used so far? Processes using 2 CPUs accumulate time at
  twice the normal rate.
* `COMMAND`: What command was used to launch a process?

`htop` provides a [curses]()-based overlay for `top`, producing a better-organized and "prettier"
dashboard in your terminal.

### `ps `

To show all processes from your current session, type `ps`.

```
{{ site.remote.prompt }} ps
```
{: .bash}

```
  PID TTY          TIME CMD
15113 pts/5    00:00:00 bash
15218 pts/5    00:00:00 ps
```
{: .output}

Note that this will only show processes from our current session. To show all processes you own
(regardless of whether they are part of your current session or not), you can use `ps ux`.

```
{{ site.remote.prompt }} ps ux
```
{: .bash}

```
    USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
{{ site.remote.user }}  67780  0.0  0.0 149140  1724 pts/81   R+   13:51   0:00 ps ux
{{ site.remote.user }}  73083  0.0  0.0 142392  2136 ?        S    12:50   0:00 sshd: {{ site.remote.user }}@pts/81
{{ site.remote.user }}  73087  0.0  0.0 114636  3312 pts/81   Ss   12:50   0:00 -bash
```
{: .output}


This is useful for identifying which processes are doing what.
