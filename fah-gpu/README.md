# Folding@home GPU Container

Folding@home is a distributed computing project for simulating protein
dynamics, including the process of protein folding and the movements of
proteins implicated in a variety of diseases. It brings together citizen
scientists who volunteer to run simulations of protein dynamics on their
computers. Insights from this data are helping scientists to better
understand biology, and providing new opportunities for developing
therapeutics.

## Overview

Running the Folding@home container is straightforward, however special care
must be taken to manage and return Work Units on time.

Familiarity with Linux and containers is assumed. Due to the prerequisites
and setup complexity this does not make an ideal "hello-world" container -
the standard Folding@home Linux clients work great and have _slightly_ less
overhead.

The Folding@home container is similar to a database container needing
persistent storage mounted into /fah and careful lifecycle management to
avoid losing or wasting work. The config.xml also contains client state, so
must be managed that way.

CUDA 9.2 is used as a base for greater compatibility - for the details, see:
[CUDA Compatibility](https://docs.nvidia.com/deploy/cuda-compatibility/index.html)

This document uses Docker as the example runtime but others are also
supported. Read the [Other Runtimes](#other-runtimes) section for Singularity
and other runtimes.

### Operating the Folding@home Container

Each of these is explained in more detail below, but they are included here
for clarity. RFC 2119 meanings.

* MUST mount read-writable persistent storage to `/fah` of the running
  container. Running containers MUST NOT share the same mounted directory,
  but directories SHOULD be reused to avoid lost Work Units.
* MUST create and preload a tuned `config.xml` in each persistent
  storage directory before running the container for the first time.
* MUST run the container as a uid:gid, specified with with `--user` or
  equivalent, so that the running container has read-write permissions to
  the persistent storage mounted in `/fah`.
* SHOULD NOT run containers as root.
* SHOULD NOT expose ports to internet without firewall rules, encryption, and
  strong passwords.

### Folding@home Websites

* Folding@home: https://foldingathome.org/
* Folding@home Support Forum: https://foldingforum.org/
* Folding@home Containers GitHub: https://github.com/foldingathome/containers/
* Folding@home Docker Hub: https://hub.docker.com/u/foldingathome

### Feedback and Issues

Read the README and CONTRIBUTING at
https://github.com/foldingathome/containers/ for design goals,
architecture, guidelines for contributing, and other information.

Please raise any bugs or issues with the containers on GitHub:
https://github.com/foldingathome/containers/issues

## Prerequisites

### Setup User Configuration

* Pick your Username -
  [FAQ](https://foldingathome.org/support/faq/stats-teams-usernames/).
* Setup your
  [Passkey](https://foldingathome.org/support/faq/points/passkey/).
  This will give bonus points after completing 10 Work Units on time.
* Join a team or create your own -
  [FAQ](https://foldingathome.org/support/faq/stats-teams-usernames/).
* If running inside of a company, make sure management has signed off on both
  your participation with company resources, and the team/user names used.

These values will be used in your config.xml later.

### Technical Requirements

* Docker 19.03 or later (for single node).
* Persistent storage for each running container.

#### For NVIDIA GPUs
* Updated NVIDIA GPU Driver - v396 or later. 440+ recommended, avoid .run files.
* NVIDIA Container Runtime -
  https://github.com/NVIDIA/nvidia-container-runtime

## Running on Single machines

Before scaling up containers on a cluster or cloud, it's important to be
familiar with the `/fah` storage requirements, life cycle, and
usage that will complete Work Units and help the research on Folding@home.
That starts with one machine.

### Single Machine Setup

Once the [prerequisites](#prerequisites) are met, it's time to run the
container.

See [example config files](#example-config-files) and be sure to
set your user/passkey/team.

```bash
# Make a directory for persistent storage
mkdir $HOME/fah

# Edit config.xml based on an example config below, use vi or other editor.
vi $HOME/fah/config.xml
```

Over time config.xml will also have client state, and will be rewritten by
the client.

### Start Folding on a Single Machine

```bash
# Run container with GPUs, name it "fah0", map user and /fah volume
docker run --gpus all --name fah0 -d --user "$(id -u):$(id -g)" \
  --volume $HOME/fah:/fah fah-gpu:VERSION
```

### Monitoring Logs on a Single Machine

```bash
# Dump output so far this run
docker logs fah0

# Tail the log
docker logs -f fah0
```

### Stopping Container on a Single Machine

```bash
# Stop container once Work Units finish (preferred), may take hours
docker exec fah0 FAHClient --send-command finish

# Stop container after checkpoint, usually under 30 seconds.
# Be sure to start it again to let it finish before the Work Units expire.
docker exec fah0 FAHClient --send-command shutdown
# The container can also just be killed, but that's not as nice.
```

## Running on Clusters

There are a lot of container orchestrators, so the requirements are as
simple as possible:

* The container orchestration needs to be able to allocate and manage GPUs.
* Run one container per machine/VM - each client can manage many GPUs and
  CPU cores, and should have a config.xml tuned for the host/VM size.
* Each running container must have it's own seperate persistent storage
  directory mounted into the `/fah` directory of the container. They should
  be reused, but two containers should never be using the same directory.

### Cluster Storage Setup

Create a root folder on the cluster storage, e.g. `.../root-dir/` and create
subdirectories based on one of these methods:

Method 1: For smaller clusters, having one directory per host is simple. When
run the containers can mount `.../root-dir/$hostname/` to `/fah` for the job
running on `hostname`.

Method 2: For larger clusters, having a pool of directories that can be reused
based on how many clients are run. Running them takes more careful management
but mounting `.../root-dir/$jobname/` to the `/fah` folder of jobs named
`fold00` ... `fold99` is the general idea.

_**Before running any clients make sure to copy your customized  `config.xml`
to all the subfolders**_.

Other methods are valid, as long as they meet the requirements above.

### Start Folding on a Cluster

Based on the storage setup, run one container per subfolder, mounting it
into `/fah`.

### Monitoring Logs on a Cluster

Your container orchestrator should have commands equivalent to
`docker logs ...` and `docker exec ... ` to perform the same functions.

```bash
# See how many Work Untis have been returned by all clients
grep points .../root-dir/*/log.txt .../root-dir/*/logs/*.txt
```

### Stopping Container on a Cluster

How containers are stopped on the cluster will effect how many Work Units are
late or lost.

```bash
# prefered shutdown
command exec container-id FAHClient --send-command finish

# Stop container after checkpoint, usually under 30 seconds.
command exec container-id FAHClient --send-command shutdown
# The container can also just be killed, but that's not as nice.
```

The goal is to **avoid accumulating a lot of subdirectories with unfinished
Work Units**.

Running the Folding@home container with low priority on a cluster where it
gets preempted and resumed will work fine. The `max-units` configuration
option may also be useful in combination with low priority to use idle
capacity where preemption is not available.

## Example Config Files

The config options used for running the client in containers are slightly
different than the ones used in a standalone install.
These are the interesting ones:

* user, passkey, team - user and team. Set them to your values.
* exit-when-done - have container exit once a finish is sent to it.
* power - run 100% of the time but idle priority.
* web-enable, disable-viz, gui-enabled - disable unnecessary things.
* slots ... - SMP and GPU slots. Each GPU slot also takes 1 CPU core.
  Each SMP slot can be set to use N cores. The "cpus" tag can be left out on
  1-cpu low core count machines and it will autoconfig.

Client help on all the options is available with:

```bash
docker run --rm fah-gpu:VERSION --help
```

#### 1-GPU, 1-CPU, 16 thread Example Config

```xml
<config>
  <!-- Set with your user, passkey, team-->
  <user value="Anonymous"/>
  <passkey value=""/>
  <team value="0"/>

  <power value="full"/>
  <exit-when-done v='true'/>

  <web-enable v='false'/>
  <disable-viz v='true'/>
  <gui-enabled v='false'/>

  <!-- 1 slots for GPUs -->
  <slot id='0' type='GPU'> </slot>

  <!-- 16-1 = 15 = 3*5 for decomposition -->
  <slot id='1' type='SMP'> <cpus v='15'/> </slot>

</config>
```

#### 8-GPU, 2-CPU, 80-thread Example Config

```xml
<config>
  <!-- Set with your user, passkey, team-->
  <user value="Anonymous"/>
  <passkey value=""/>
  <team value="0"/>

  <power value="full"/>
  <exit-when-done v='true'/>

  <web-enable v='false'/>
  <disable-viz v='true'/>
  <gui-enabled v='false'/>

  <!-- 8 slots for GPUs -->
  <slot id='0' type='GPU'> </slot>
  <slot id='1' type='GPU'> </slot>
  <slot id='2' type='GPU'> </slot>
  <slot id='3' type='GPU'> </slot>
  <slot id='4' type='GPU'> </slot>
  <slot id='5' type='GPU'> </slot>
  <slot id='6' type='GPU'> </slot>
  <slot id='7' type='GPU'> </slot>

  <!-- 80-8= 72/4 = 18 = 2*3*3 to allow for NUMA domains and decomposition -->
  <slot id='8' type='SMP'> <cpus v='18'/> </slot>
  <slot id='9' type='SMP'> <cpus v='18'/> </slot>
  <slot id='10' type='SMP'> <cpus v='18'/> </slot>
  <slot id='11' type='SMP'> <cpus v='18'/> </slot>

</config>
```

## Other Runtimes

While this README focused on Docker, it is not the only container runtime.

### Singularity

A full Singularity HOWTO is beyond this document currently. These commands
should help someone familiar with Singularity get started on a single machine:

```bash
mkdir fah && cd fah
# Create/Copy config.xml
singularity build fah.sif docker://CONTAINER-PATH/fah-gpu:VERSION
singularity instance start --nv -B$(pwd):/fah fah.sif fah_instance
singularity exec instance://fah_instance /bin/bash -c "coproc /usr/bin/FAHClient"
tail -f log.txt
```
