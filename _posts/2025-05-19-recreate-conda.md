---
title: 'A swiss army knife for conda issues - re-create your environment'
date: 2025-05-19
permalink: /posts/2025/02/recreate-conda/
tags:
  - mamba 
  - micromamba
  - conda
  - pixi
  - maintenance
---

## Delete your conda environments often

Here, I'll be making the case that deleting and re-creating your conda environments is a best practice to do every once in a while and also something to frequently reach for when things look off. This doesn't solve every problem, but it works more often than you might expect.

It's also commonly the first step in producing a high-quality issue and, for "ghost in the machine" types of problems, can save potentially hours of human time. The trade-offs of a few minutes' investment are pretty good.

## No, really, they're meant to be deleted

The point of virtual environments (a category of tools that includes conda environments) is to isolate library and application versions used in a specific project from everything else on your system. Python practitioners do this so often we may take for granted how cool this is - we can safely test out different versions of software in an isolated corner of a harddrive without committing to changing anything outside of it.

A feature of virtual environments is that they're easy to create. If it took as long to try out the newest Python release as it did to install a fresh operating system or start a new virtual machine, there wouldn't be much of a point to it. The flipside of this is that virtual environments are also eminently deletable. **Easy to create, easy delete.**

Another implementation detail: the environments are stored separately from the virtual environment manager itself and all of its configs, cache, etc. So deleting an environment **does not** delete your conda/mamba/micromamba/etc installation.

## How to delete your conda environments

It's as simple as finding where your conda environment is stored - probably at `$CONDA_PREFIX` - and blow it away. Just remove it.

For example, I was most recently working on something in an environment named `interchange-examples-env` which is tracked by a file in the path `devtools/conda-envs/examples.yaml`. I use `micromamba` which is installed on this machine at `/Users/mattthompson/micromamba/` - your setup might differ slightly but `$CONDA_PREFIX` should point to your current environment. Like so:

```console
$ echo $CONDA_PREFIX
/Users/mattthompson/micromamba/envs/interchange-examples-env
$ rm $CONDA_PREFIX
$ micromamba create --file devtools/conda-envs/examples_env.yaml --quiet
Confirm changes: [Y/n] y
warning  libmamba [xorg-libx11-1.8.10-h6a5fb8c_1] The following files were already present in the environment:
    - include/X11/Xlib.h
    - include/X11/Xutil.h
    - include/X11/cursorfont.h
warning  libmamba [openff-interchange-base-0.4.1-pyhd8ed1ab_0] The following files were already present in the environment:
    - lib/python3.12/site-packages/docs/conf.py
warning  libmamba [openff-nagl-base-0.5.1-pyhd8ed1ab_0] The following files were already present in the environment:
    - lib/python3.12/site-packages/docs/conf.py
$ micromamba activate interchange-examples-env
$ micromamba list  # or whatever command to verify the result
...
```

I timed this and it took about 100 seconds. This environment is particularly bulky with several large packages; other environments should take much less time to re-create. For smaller environments, think along the lines of 5 or so seconds.

## More reasons to delete your conda environments

### Virtual environments can carry historical cruft

It's hard to be more specific because of the range of things that can happen when updating conda environments, switching Python versions, or blindly copying installation instructions (no shame, I do this myself). I also don't fully understand the ways that constraints and various configs can persist in environments (and some of these interactions are probably bugs).

### Re-creating your environment syncs your development with the environment file

It's easy for the contents of your environment to diverge from what's encoded in your environment. It can start with an innocuous-seeming change to one package's version but over time grow to include changes to several packages' major versions. As these changes accumulate, you start to lose track of important details such as why they were made and whether or not some packages are incompatible with each other. Environment files easily capture this on disk on disk in a standardized format (YAML is more standardized than a string of shell commands) which can be tracked with version control. You can, and probably should, pepper this file with comments explaining, for example, why particular version constraints are included or when a particular pin might be removable ("let OpenMM 8 be installed when X plugin is updated ...")

This can be done with `mamba env update ...` but going all the way to re-creating a an environment better represents the contents of the environment, i.e. it "forgets" the other modifications that you may have made. (If you do want to make changes, you should make them to the environment file directly, since that's the best record of what you intend to be there.)

If all of this seems to you like a nuisance - you're not alone! Keeping a project environment and its environment specificaiton _always_ and _automatically_ in sync is the basic pitch of [Pixi](https://pixi.sh/latest/). I think `uv` and some `pip`-adjacent tools have similar features, but haven't used them myself.

### It's what's going to happen when deploying your product

No matter where your script or package ends up after you're done with it, an environment will be re-created from scratch later on.

* If you're collaborating with somebody on a shared project, their first step will probably be making a conda environment.
* If you're deploying an app to the cloud or firing off huge production runs to HPC, it's the same story.
* If you're developing a software package, bots in the form of CI runners will do this frequently.
* f you're a solo researcher working on a project that can be started and completed on your workstation, somebody trying to reproduce your findings will start by (you guessed it!) re-creating the environment.

### It's the first step in submitting a bug report, anyway

Remember that devising a minimally reproducible example is central to making a high-quality bug report - if the behavior can't be reproduced on a new machine, **it effectively doesn't exist**. The first thing a maintainer (or bot!) will do in trying to diagnose your possibly-buggy finding is reproduce the behavior of interest. In order to do this, essentially the zeroth step, they will need to install versions of all relevant software on their machine, a.k.a. re-creating your environment. In submitting a bug report, you might as well do this yourself to verify that the maintainer will see what you predict they'll see.

### It shouldn't take long to re-create the environment

I can hear this concern because it was my internal dialogue _for ages_ -  "it's too risky for me to re-create this environment, the current version constraints are going to make impossible to solve!" These days, with rare exceptions, that's no longer the case. It's true that Conda environments were often slow - even unusably slow - to solve many years ago. (In packaging jargon, "solving" or finding a "solution" is the process by which the package manager finds versions of every package that satsfies the requirements, probably communicated in an environment YAML file.) These days, environments take a few seconds to solve - gone are the days of watching `conda` spin its wheels for 20 minutes and get nowhere. These tools also make use of caching under the hood so that many steps (i.e. downloading tarballs) are often skipped when creating an environment the second time.

### An aside - Mamba/micromamba are great, but conda is fine

Mamba came into existence in large part out of a desire to re-write Conda's solver from the ground up in C++ while maintaining a similar user experience. It has some rough edges 5+ years ago but now (in mid 2025) the community has accepted it for general use. Its solver, `libmamba`, was even [integrated into `conda` itself](https://www.anaconda.com/blog/a-faster-conda-for-a-growing-community) a few years ago and later [the default](https://conda.org/blog/2023-11-06-conda-23-10-0-release/). I still recommend using [Mamba](https://mamba.readthedocs.io/en/latest/) or its slimmed-down drop-in replacement [Micromamba](https://mamba.readthedocs.io/en/latest/user_guide/micromamba.html), but if you see teammates using `conda`, don't automatically assume that they're using the same tool you were frustrated by in 2019. It's much-improved across the board thanks to a monumental and sustained volume of effort from volunteer contributors in the conda orbit. Some companies have also made significant contributions: Anaconda, Inc., Quansight, QuantStack, and prefix.dev immediately come to mind, but I'm sure I'm forgetting some.
