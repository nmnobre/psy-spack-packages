# PSyclone Spack Software Stack

This repository contains the Spack packages necessary to build some of the
applications that have PSyclone as part of their build system. As such, these
packages do not install the final applications nor PSyclone itself. Instead
they provide the necessary dependencies and environment variables (FC,
LD_LIBRARY_PATH, ...) to build each application. You need your own version
of PSyclone. Currently, the purpose of this repository is to provide two bundle
packages:

- lfric-build-environment
- nemo-build-environment

> **_NOTE:_**  Spack bundle packages were chosen over Spack environments because
they have more flexibility to choose compiler/variant/dependencies when installing
them and they allow the set up of environment variables when loading the package.

## Initial set up

Clone Spack from GitHub and add it to your environment:
```bash
git clone https://github.com/spack/spack.git
source "$PWD/spack/share/spack/setup-env.sh"
```

In addition to Spack's builtin package repository, the bundles need the
Met Office package repository and this repository. Add both directly from
GitHub, in this order:
```bash
spack repo add --name metoffice https://github.com/MetOffice/mo-spack-packages.git
spack repo add --name psyclone https://github.com/nmnobre/psy-spack-packages.git
spack repo list
```

Spack prepends each repository, so this gives the PSyclone packages precedence
over the Met Office and builtin packages. Spack manages the Git checkouts; see
the [Spack repository documentation](https://spack.readthedocs.io/en/latest/repositories.html)
for how to pin repositories to commits or choose checkout locations.

Then configure any Spack settings. Config files can be added to either:
  - ${PWD}/spack/etc/spack/ to affect only this Spack instance but for all users.
  - ~/.spack/ to affect all Spack instances but only for this user.

First use `spack external find`, to use system libraries where available, then
make the necessary changes to the generated `packages.yaml`, for example adding:
```
    all:
        variants: cuda_arch=70
```
Choose the right "cuda_arch" by referring to: https://developer.nvidia.com/cuda-gpus

V100 -> 70
A100 -> 80
H100 -> 90
3060ti -> 86
4050 -> 89

If your system needs specific installations e.g. cuda for WSL or MPI with
specific system configurations, also add them here:
```
    # Modify spack/var/spack/repos/builtin/packages/cuda/package.py:
    # find_libraries("libcudart" -> find_libraries("libcuda"
    cuda:
        buildable: false
        externals:
        - spec: cuda@12.5
          prefix: /usr/lib/wsl/
```

To install the compilers use:
```
spack install gcc +nvptx
spack install intel-oneapi-compilers
spack install nvhpc
spack install llvm-amdgpu
```
Alternatively use `spack compiler find` to have Spack auto-detect system
compilers (and edit the `packages.yaml` as needed).
Once done create a `toolchains.yaml` with each toolchain, e.g:

```
toolchains:
  gnu14_openmpi:
  - spec: "%c=gcc@14"
    when: "%c"
  - spec: "%cxx=gcc@14"
    when: "%cxx"
  - spec: "%fortran=gcc@14"
    when: "%fortran"
  - spec: "%mpi=openmpi@5"
    when: "%mpi"
```

## Spack Usage

Applications/libraries available on Spack (`spack list`) can be installed with:
```bash
spack install <application>
```

Before installing, use the commands:
- `spack info <application>` to see the available versions and variants
- `spack spec <application>` to see which dependencies will be installed

It is sometimes useful to use the `-U` (`--fresh`) option to concretise the
dependencies without preferring the already installed packages:

Currently, for lfric we need to be specific about which mpi version to use
otherwise the toolchain will use a mix of them:
```bash
spack install lfric-build-environment%nvhpc ^openmpi@4.1+cuda%nvhpc
```

Once installed, a package can be, found, loaded and its files located with:
```bash
spack find -x
spack load lfric-build-environment%nvhpc
spack location -i lfric-build-environment
```
