FROM ubuntu:26.04 AS gwatubuntu

# Path to Python environment to be created
ARG PYENV=/usr/gwatpyenv

## C++ and wget INSTALL ##
    # build-essential - libc, gcc, g++, make, dpkg-dev
    # wget - Download from URL [REQUIRED FOR LALSUITE METAIO]
# Clear /var/lib/apt/lists/ to reduce cache.
# Note that this will require apt-get update before running apt-get install.

RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install -yq \
        build-essential \
        wget \
        cmake \
    && rm -rf /var/lib/apt/lists/*

## PACKAGE INSTALL GUIDE ##
    ## DEVELOPMENT PACKAGE installs ##
        # autoconf - Produce configure scripts for building [REQUIRED FOR ADOL-C]
        # automake - Atomate parts of the compilation process
        # git - Revision control system
        # libtool - General library support script
        # swig - Generate scripting interfaces to C/C++ code

    ## PYTHON PACKAGE installs ##   
        # python3
        # python3-pip - Python package installer
        # python3-dev - Header files and static library for Python [REQUIRED FOR: LALSUITE]
        # python3-venv - Virtual environment manager to avoid conflicts with APT

    ## SCIENTIFIC AND DATA PACKAGE installs ##
        # libeigen3-dev - C++ library for linear algebra
        # libfftw3-dev - Fast Fourier Transform library
        # libgsl-dev - GNU scientific library
        # libhdf5f-dev - Heirarchical Data Format 5 (HDF5) development files

RUN apt-get update \
&& DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends -yq \
    autoconf \
    automake \
    git \
    libtool \
    swig \
    python3 \
    python3-dev \
    python3-pip \
    python3-venv \
    libeigen3-dev \
    libfftw3-dev \
    libgsl-dev \
    libhdf5-dev \
    && rm -rf /var/lib/apt/lists/*
    

## ADOL-C install ##
WORKDIR /opt/adolc
# Download old release, since we need ADOL-C to compile with OpenMP.
# New release breaks when using AutoTools, and currently the CMake installation doesn't allow for OpenMP.
# Once ADOL-C is modernized with CMake and allows this option, we should upgrade to it.
RUN wget https://github.com/coin-or/ADOL-C/archive/refs/tags/releases/2.7.2.tar.gz \
    && tar -zxf 2.7.2.tar.gz && rm 2.7.2.tar.gz
WORKDIR ADOL-C-releases-2.7.2/build
# Configure the installation using autoconf to compile with OpenMP
RUN autoreconf -fi ../ \
   && ../configure --prefix=/usr/local --with-openmp-flag=-fopenmp
RUN make && make install
WORKDIR /
RUN rm -rf /opt/adolc
