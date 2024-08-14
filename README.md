This container contains a build of [wsclean](https://sourceforge.net/projects/wsclean/)
with [ASTRON IDG](https://gitlab.com/astron-idg/idg) integrated to enable the container
to utilize GPUs for acceleration.

Launching the container
=======================

The process for launching the wsclean + IDG container is a bit more involved than
that for launching the other containers due to the need to ensure that the kernel
modules are loaded and libraries linked in.

```
# Triggers the loading of all the needed NVIDIA kernel modules
# (should be replaced by use of `nvidia-modprobe` at some point but IIRC
#  `nvidia-modprobe` doesn't load all drivers with default params)
/usr/local/cuda/extras/demo_suite/bandwidthTest &> /dev/null

# Launch the container itself
singularity shell wsclean-gpu.simg
```

> **Note**: that this container assume that CUDA has been installed in
`/usr/local/cuda` and that this path has been bound into the container as this binds
in a more complete set of libraries than adding the `--nv` parameter adds

> **Note**: a wiki page describe the Image-Domain Gridder is available
[here](https://sourceforge.net/p/wsclean/wiki/ImageDomainGridder/).

When running the Image-Domain Gridder, the `CUDA_DEVICE` environment variable will
control which GPUs are used.  e.g. if you `export CUDA_DEVICE=0,1` this will result
in the first two GPUs being used.

The gridder uses OpenMP for parallelism on the CPU in hybrid and CPU mode, so if not
on a dedicated machine you can set the `OMP_NUM_THREADS` environment variable to limit
the number of CPU threads used. 

Build instructions
==================

First install CUDA using the packages available [here](https://developer.nvidia.com/cuda-downloads)
and then add the following line to `/etc/singularity/singularity.conf`:

```
bind path = /usr/local/cuda
```

The build process for this container is slight different from the typical singularity
build process as the CUDA libraries are not bound into the container at the time of
the initial build.  Thus first a writeable sandbox must be built, a `first-run.sh`
script executed to build the application, and then the sandbox converted into a
standard image.

The build can be done using a sequence of commands like the following:

```
SAND=${RANDOM}.sandbox
sudo singularity build --sandbox $SAND wsclean.def
sudo singularity exec --nv --writable $SAND /opt/build/first-run.sh
sudo singularity exec $SAND rm /opt/build/first-run.sh
sudo singularity build wsclean-gpu.simg $SAND
sudo rm -r $SAND
```

