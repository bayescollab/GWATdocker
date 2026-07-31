# GWAT Development with Docker

Developing in GWAT is fairly straightforward, and we have been working towards modernizing the process as much as we can. Some differences from the initial Docker image of GWAT (circa 2024) are:

- **Lighter images**: The Debian-based image `gw.base.deps` has been switched to the lighter Ubuntu base. APT packages are downloaded without their recommended installs and the cached lists are cleared after their installation.

- **Updated software**: With Ubuntu 24.04 (noble), PIP is a bit more restrained now in order to avoid conflicts with software installed via APT. A Python environment is now set up and added to `PATH` to handle 1) PIP packages and 2) BayesShip packages.

- **Multi-stage builds**: Installing the code comes in two main stages: the building of the bse image, here referred to as `gwatubuntu`, and the installation of BayesShip and GWAT on top of this image. These two codes are handled in separate stages. See their installation below.

## Installation

The first container to build is `gwatubuntu`, which is meant to take the place of `gw.base.deps`. In a directory with structure

```bash
.
├── gw_analysis_tools/
├── BayesShip/
├── Dockerfile.gwatubuntu
└── Dockerfile.gwat.bayesship
```

run the command

```sh
docker build -f Dockerfile.gwatubuntu -t gwatubuntu .
```

This should build the base image. Now, to install *all* of GWAT, simply run

```sh
docker build -f Dockerfile.gwat.bayesship -t gwatbuild .
```

This will simply use your local GWAT code and install it within the image, and then remove the codes used to build it. You should now have a not-so-heavy image of GWAT. If your plan is to develop GWAT code, i.e. modify it, then follow the instructions below.

## Developing

To modify the code and then make sure it runs within the code, we can run an interactive image and do so. In general, we don't need to modify BayesShip, and so we allow that code to install by running

```sh
docker build -f Dockerfile.gwat.bayesship -t gwatbuild --target bayes-build .
```

This only builds the Dockerfile up to and including the `bayes-build` stage. To then modify the contents of `gw_analysis_tools/`, we run an image,

```sh
docker run -it --rm \
    --mount type=bind,src="$(pwd)"/gw_analysis_tools,dst=/opt/gw_analysis_tools \
    gwatbuild
```

This mounts the `gw_analysis_tools/` directory and allows changes made within the image to be stored locally. The `-it` option also places you within an interactive shell. To install the GWAT code within the image, run within that shell the following,

```sh
# Make the build directory if it does not exist
mkdir build
cd build
# Make sure it is clean
rm -rf *
# Set up the installation
cmake -DCMAKE_INSTALL_PREFIX=/usr/local ../
# Compile across 4 cores. It is a large code
make -j4
# Install
make install
```

"Developing" implies that you make changes to the code, and re-run the installation. If modifying existing files, starting only from the `make` line should suffice; if modifying it substantially, or creating new files, start from the `cmake` line.

If you wish to use VS Code to make the modifications and test the software, one can start a detached image with the options `-di`. With the `Dev Containers` extension installed in VSC, you can attach to this image and play with it like any other VSC project.