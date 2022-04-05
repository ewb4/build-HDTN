# build-HDTN

## Overview

Building the NASA HDTN software from https://github.com/nasa/HDTN

## System configuration

The HDTN build instructions specify a few different platforms.  For this Vagrant build, the Ubuntu 20.04 LTS distribution is selected.  The Vagrant boxes available for Ubuntu 20.04 LTS come from two different providers.  Both the Bento box from the Chef build and the ubuntu/focal64 box are available in the "machines" array.

```ruby
machines = [
  { selector: 'focal64', hostname: 'hdtn-ubuntu2004-lts',   ram: 1536, family: 'debian', box: 'ubuntu/focal64',     box_version: '20220302.0.0', cpus: 1 },
  { selector: 'bento',   hostname: 'hdtn-bentoubuntu-2004', ram: 1536, family: 'debian', box: 'bento/ubuntu-20.04', box_version: '202112.19.0',  cpus: 1 }
]
```

## Vagrant added options

Since the Vagrantfile is really just a bit of Ruby code, command line options can be added to help select options to pass into the build.


## FAQ

Q: Why not just follow the instructions from NASA on a Linux system like their build documents say?  (After all, aren't they rocket scientists?)
A: Here is a list of some of the reasons to do this exercise.  Writing code to follow the directions ensures that the directions are followed completely each time.  The [HashiCorp Vagrant](https://www.vagrantup.com/docs/vagrantfile) methodology helps to avoid the "works on my system" problem when debugging a problem.

Q: The latest code from the NASA repo does not build any more.  Why?
A: In this repo there is a "proof-of-concept" build of a specific commit hash.  Work was done to add options so other commit hashes can be built, but there may be additional requirements as the NASA HDTN software is developed.


