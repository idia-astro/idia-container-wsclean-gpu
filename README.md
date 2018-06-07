This container contains a build of [wsclean](https://sourceforge.net/projects/wsclean/)
with [ASTRON IDG](https://gitlab.com/astron-idg/idg) integrated to enable the container
to utilize GPUs for acceleration.

Note that this container assume that CUDA has been installed in
`/usr/local/cuda` and that this path has been bound into the container as this binds
in a more complete set of libraries.

The build process for this container is slight different from the typical singularity
build process as the CUDA libraries are not bound into the container at the type of
the initial build.  Thus first a writeable sandbox must be built, a `first-run.sh`
script executed to build the application, and then the sandbox converted into a
standard image.

The build can be done using a sequence of commands like the following:

```
SAND=${RANDOM}.sandbox
sudo singularity build --sandbox $SAND wsclean.def
sudo singularity exec --nv --writable $SAND /first-run.sh
sudo singularity exec $SAND rm /first-run.sh
sudo singularity build wsclean-gpu.simg $SAND
sudo rm -r $SAND
```
