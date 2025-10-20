---
title: "Accessing software"
teaching: 30
exercises: 15
questions:
- "How do we load and unload software packages?"
objectives:
- "Understand how to load and use a software package."
keypoints:
- "Load software with `module load softwareName`"
- "Unload software with `module purge`"
- "The module system handles software versioning and package conflicts for you automatically."
---

On a high-performance computing system, it is seldom the case that the software we want to use is
available when we log in. It is installed, but we will need to "load" it before it can run.

Before we start using individual software packages, however, we should understand the reasoning
behind this approach. The three biggest factors are:

- software incompatibilities
- versioning
- dependencies

Software incompatibility is a major headache for programmers. Sometimes the presence (or absence) of
a software package will break others that depend on it. Two of the most famous examples are Python 2
and 3 and C compiler versions. Python 3 famously provides a `python` command that conflicts with
that provided by Python 2. Software compiled against a newer version of the C libraries and then
used when they are not present will result in a nasty `'GLIBCXX_3.4.20' not found` error, for
instance.

Software versioning is another common issue. A team might depend on a certain package version for
their research project - if the software version was to change (for instance, if a package was
updated), it might affect their results. Having access to multiple software versions allow a set of
researchers to prevent software versioning issues from affecting their results.

A dependency exists when a particular software package (or even a particular version)
depends on having access to another software package (or even a particular version of
another software package). For example, the VASP materials science software may 
depend on having a particular version of the FFTW (Fastest Fourier Transform in the West)
software library available for it to work.

## Environment modules

Environment modules are the solution to these problems.
A *module* is a self-contained description of a software package - 
it contains the settings required to run a software package 
and, usually, encodes required dependencies on other software packages.

There are a number of different environment module implementations commonly
used on HPC systems: the two most common are *TCL modules* and *Lmod*. Both of
these use similar syntax and the concepts are the same so learning to use one will
allow you to use whichever is installed on the system you are using. In both 
implementations the `module` command is used to interact with environment modules. An
additional subcommand is usually added to the command to specify what you want to do. For a list
of subcommands you can use `module -h` or `module help`. As for all commands, you can 
access the full help on the *man* pages with `man module`.

On login you may start out with a default set of modules loaded or you may start out
with an empty environment; this depends on the setup of the system you are using.

### Listing available modules

To see available software modules, use `module avail`

```
{{ site.remote.prompt }} module avail
```
{: .bash}

{% include {{ site.snippets }}/modules/available-modules.snip %}

### Listing currently loaded modules

You can use the `module list` command to see which modules you currently have loaded
in your environment. If you have no modules loaded, you will see a message telling you
so

```
{{ site.remote.prompt }} module list
```
{: .bash}

```
Currently Loaded Modules:
  1) CCconfig            4) gcc/12.3    (t)   7) libfabric/1.18.0  10) openmpi/4.1.5   (m)     13) StdEnv/2023 (S)
  2) gentoo/2023   (S)   5) hwloc/2.9.1       8) pmix/4.2.4        11) flexiblas/3.3.1         14) mii/1.1.2
  3) gcccore/.12.3 (H)   6) ucx/1.14.1        9) ucc/1.2.0         12) imkl/2023.2.0   (math)

[Some output removed for brevity]

```
{: .output}

## Loading and unloading software

To load a software module, use `module load`.
In this example we will use R, a software environment for statistics.

Initially, R is not loaded. 
We can test this by using the `which` command.
`which` looks for programs the same way that Bash does,
so we can use it to tell us where a particular piece of software is stored.

```
{{ site.remote.prompt }} which R
```
{: .bash}

{% include {{ site.snippets }}/modules/missing-python.snip %}

We can load the `R` command with `module load` (be careful of the case of the
letters, `r` vs `R`):

{% include {{ site.snippets }}/modules/module-load-python.snip %}

{% include {{ site.snippets }}/modules/python-executable-dir.snip %}

So, what just happened?

To understand the output, first we need to understand the nature of the `$PATH` environment
variable. `$PATH` is a special environment variable that controls where a UNIX system looks for
software. Specifically `$PATH` is a list of directories (separated by `:`) that the OS searches
through for a command before giving up and telling us it can't find it. As with all environment
variables we can print it out using `echo`.

```
{{ site.remote.prompt }} echo $PATH
```
{: .bash}

{% include {{ site.snippets }}/modules/python-module-path.snip %}

You'll notice a similarity to the output of the `which` command. In this case, there's only one
difference: the different directory at the beginning. When we ran the `module load` command,
it added a directory to the beginning of our `$PATH`. Let's examine what's there:

{% include {{ site.snippets }}/modules/python-ls-dir-command.snip %}

{% include {{ site.snippets }}/modules/python-ls-dir-output.snip %}

Taking this to its conclusion, `module load` will add software to your `$PATH`. It "loads"
software. The `module load` command will also load required software dependencies.

{% include {{ site.snippets }}/modules/software-dependencies.snip %}

## Software versioning

So far, we've learned how to load and unload software packages. This is very useful. However, we
have not yet addressed the issue of software versioning. At some point or other, you will run into
issues where only one particular version of some software will be suitable. Perhaps a key bugfix
only happened in a certain version, or version X broke compatibility with a file format you use. In
either of these example cases, it helps to be more specific about what version of a software is loaded.

In the previous example, running `module load r` loaded version 4.4.0 (r/4.4.0). Since we did not 
specify a version, a default version was loaded. The default versions may change periodically as 
software and standard environments change, so we should learn to find and load specific versions of 
software. Let's examine the output of `module spider gcc`.

```
{{ site.remote.prompt }} module spider gcc
```
{: .bash}

```
    Description:
      The GNU Compiler Collection includes front ends for C, C++, Objective-C, Fortran, Java, and Ada, as well as libraries for these languages (libstdc++,
      libgcj,...).

     Versions:
        gcc/8.4.0
        gcc/9.3.0
        gcc/10.2.0
        gcc/10.3.0
        gcc/11.3.0
        gcc/12.3
        gcc/13.3
     Other possible modules matches:

[Some output removed for clarity]
```
{: .output}

{% include {{ site.snippets }}/modules/wrong-gcc-version.snip %}

> ## Using software modules in scripts
>
> Create a job that is able to run `R --version`. Remember, no software is loaded by default!
> Running a job is just like logging on to the system (you should not assume a module loaded on the
> login node is loaded on a compute node).
>
> > ## Solution
> >
> > ```
> > {{ site.remote.prompt }} nano job-with-module.sh
> > {{ site.remote.prompt }} cat job-with-module.sh
> > ```
> > {: .bash}
> >
> > ```
> > #!/bin/bash
> > 
> > module load r/4.3.1
> > 
> > R --version
> > ```
> > {: .output}
> > 
> > ```
> > {{ site.remote.prompt }} {{ site.sched.submit.name }} {{ site.sched.submit.options }} job-with-module.sh
> > ```
> > {: .bash}
> {: .solution}
{: .challenge}

