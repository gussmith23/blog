---
layout: post
title:  "Building gem5: A Supplement"
date:   2018-02-06 00:00:00 +0000
categories: software-engineering tech gem5
---

*This is not a tutorial on building gem5; for that, please read gem5's documentation. This article supplements the existing documentation with some specific workarounds needed to build gem5 on my system.*

*I didn't figure out all of these workarounds on my own; thanks to my labmates at Penn State's Microsystems Design Lab for their help.*

# Who Should Read This Article?
This article is meant for those who have tried building gem5 using [gem5's documentation](http://learning.gem5.org/book/part1/building.html) and run into issues. Specifically, this article is written for those trying to build gem5 on outdated systems and/or systems on which you don’t have root access. It will cover workarounds for the following issues:

* Incorrect GCC version.
* Incorrect Python version.

I will assume throughout this article that the reader has read gem5's build instructions. This means that I will not explain the major steps in building gem5.  I will also assume throughout this article that you have downloaded the required software listed in gem5's documentation.

# Using an Alternate GCC Version

At the time of writing, gem5 requires GCC 4.8+. My system’s default GCC version is much older; without root access, I couldn’t install a new version nor change the default. Working around this involved two main steps:

1. Our system administrator installed GCC 4.8.3 for us. If your sysadmin is unwilling or unable to do this for you, you can [build GCC yourself](https://gcc.gnu.org/install/index.html). Whichever path you choose, be sure to remember where GCC is installed. For me, this path is `/home/software/gcc/gcc-4.8.3`, but this path will look completely different on different systems.
2. An older version of GCC was still the system's default, so I forced scons to use our newer version of GCC through some `PATH` manipulation.

Achieving step 2 was straightforward on a Linux machine. First, I created a new directory (in my home directory) which contains four symbolic links to the four main GCC binaries:

```
cd
mkdir gcc-symlinks
cd gcc-symlinks
ln -s /home/software/gcc/gcc-4.8.3/bin/gcc483 gcc
ln -s /home/software/gcc/gcc-4.8.3/bin/g++483 g++
ln -s /home/software/gcc/gcc-4.8.3/bin/cpp483 cpp
ln -s /home/software/gcc/gcc-4.8.3/bin/c++483 c++
```

Second, when we build gem5 with scons, we set the `PATH` variable at the start of the command:
```
PATH=~/gcc-symlinks/:$PATH [rest of scons command]
```

Or, you can export the new `PATH` variable for your whole session:

```
export PATH=~/gcc-symlinks/:$PATH
```

With this second method, you will have to re-export the `PATH` every time you open up a new shell session.

# Using an Alternate Python Version
gem5 requires a specific version of Python, too.  At the time of writing, this is Python 2.7+; again, my system's default is older, and so I needed a workaround. gem5 provides documentation on using an alternate Python version, but these steps alone did not work for me.

For the most part, we use the same two steps as above, with one more step added.

First, ask your sysadmin to install a newer version of Python, or build it yourself. Remember to log the directory in which it is installed; for me, this is `/home/software/python/Anaconda2.4-Python2.7`.

Second, we will manipulate environment variables so that the correct version of Python is found. For GCC, we only had to manipulate the `PATH`; for Python, we must also update `PYTHONHOME`, `LIBRARY_PATH`, and `LD_LIBRARY_PATH`:

```
export PATH=/home/software/python/Anaconda2.4-Python2.7/bin:$PATH
export PYTHONHOME=/home/software/python/Anaconda2.4-Python2.7
export LIBRARY_PATH=/home/software/python/Anaconda2.4-Python2.7/lib:$LIBRARY_PATH
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/software/python/Anaconda2.4-Python2.7/lib
```
Adapt these settings to your own system using your own Python path.

Author's note: I'm not sure if all of these are required, nor am I sure whether prepending vs. appending is required in specific cases. For example: we prepend to the `PATH` variable so that our alternate Python gets found first, but why do we append to `LD_LIBRARY_PATH`?

Third, we run scons using our specific Python version explicitly. Even though the `PATH` variable has been set correctly, running scons directly was still causing issues on my system. Specifically, standard Python modules (such as `os`) could not be found. Running scons explicitly with our Python version fixed the issue. To do this, run scons as follows:

```
/home/software/python/Python-2.7.10/bin/python-2.7.10 `which scons` build/X86/gem5.debug
```

# Putting It All Together
We can put all of our workarounds into a script:

```
export PATH=~/gcc-symlinks:$PATH
export PATH=/home/software/python/Anaconda2.4-Python2.7/bin:$PATH
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/software/python/Anaconda2.4-Python2.7/lib
export LIBRARY_PATH=/home/software/python/Anaconda2.4-Python2.7/lib:$LIBRARY_PATH
export PYTHONHOME=/home/software/python/Anaconda2.4-Python2.7
```

Simply source this script before building gem5. Then, build gem5 with:

```
/home/software/python/Python-2.7.10/bin/python-2.7.10 `which scons` build/X86/gem5.debug
```

Again, remember to change the paths to the locations Python/GCC on your own system.
