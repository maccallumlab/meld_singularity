# Building MELD singularity containers

[Singularity](https://sylabs.io) gives us a way to package up an
entire software stack in a way that is portable across different
machines.  The idea is to build an image that should be:

1. Reproducible
2. Sharable
3. Portable across machines

## Step 1: Install VirtualBox and Vagrant

We use a virtual box that has everything needed to build the singularity image
pre-installed. This works on linux, mac, and windows. On linux, you can avoid
this step and use singularity directly, either by installing the appropriate
package or by [building from source](https://sylabs.io/guides/3.5/admin-guide/installation.html).

1. Install [VirtualBox](https://www.virtualbox.org)
2. Install [Vagrant](https://www.virtualbox.org)
3. `vagrant plugin install vagrant-disksize`

Run `vagrant up` to boot up the virutal machine. 

Run `vagrant ssh` to connect to the virtual machine. Press `Ctrl-D` to exit.

Run `vagrant halt` to stop the virtual machine. If you want to reclaim the space,
run `vagrant destroy` to remove the image from your system. (The vagrant images
can be quite large.)

## Step 2: Edit the singularity definition

Edit `meld.def` as needed. The main section that might need to updated is `%post`,
which is the series of steps that are used to install meld and all dependencies
into the image.

The main lines that might need to be changed are the ones that download specific
versions of openmm and meld from github. The format of the github urls is as follows.

- Tagged version
    - `https://github.com/openmm/openmm/archive/7.6.0.tar.gz`
        - This will download the version tagged with `7.6.0`
        - The resulting directory will be `openmm-7.6.0`
- Specific branch
    - `https://github.com/openmm/openmm/archive/master.tar.gz`
        - This will download the branch `master`
        - The resulting directory will be `openmm-master`
- Specific hash
    - `https://github.com/openmm/openmm/archive/ed9df87.tar.gz`
        - This will download the specific commite with has `ed9df87`
        - The resulting directory will be `openmm-ed9df87`

Note that we use a two-stage build process. In the `build` stage, we
install the full `cuda-devel` stack which is very large. We then build
ambertools, openmm, and meld. Next, we setup a second stage that only
includes the `cuda-runtime` stack. We then copy over all of the build
artifacts for ambertools, openmm, and meld to this image. Overall, this
process reduces the size of the image by about half.

## Step 2: Build the singularity image

Connect to the virtual machine:
1. `vagrant up`
2. `vagrant ssh`
3. `cd /vagrant`

The `/vagrant` directory is a pass-through directory. Files in this directory
are accessible in both the virtual machine and the host system.

4. `sudo singularity build image_name.sif meld.def`

#### Naming Convention

It's not helpful to have dozens of images all named `meld.sif`. The following
naming scheme is **strongly** recommmended:

```
meld_MELDVER_omm_OMMVER
```

where `OMMVER` is the openmm version and can either be a tagged release, like
`7.6.0`, or a speficic hash, like `ed9df87`. `MELDVER` is the same thing, but
for meld. **Do not** use a branch name, like `master`, as this will change as
the source tree is updated -- use a specific hash instead.

## Step 3: Clean up
1. `vagrant halt`
2. (Optional) `vagrant destroy`

## Step 4: Use the image

Copy the image to the system you intend to use (e.g. cedar or glados).
It is recommended to store the images in a standard location, like in your
home directory or project space.

To use interactively:

```
singularity shell BIND_COMMANDS IMAGE_PATH
```

To use in a job script:

```
srun singularity BIND_COMMANDS IMAGE_PATH COMMAND_TO_RUN
```

Bind commands allow for the host file system to show up
inside of the singularity image:

- glados: bind commands can be left blank, as `/home` is mounted by default
- cedar: -B /home -B /project -B /scratch

For COMMAND_TO_RUN, note that the images use the `python3` command, rather
than `python`.

## Example job script for cedar

## Example job script for glados