# ABINIT Docker Image - Source Build 10.4.5
# Base: Ubuntu 22.04 LTS for maximum HPC compatibility
# Target: Intel Xeon E5-2698 v4 (AVX2 support)
# Source: https://forge.abinit.org/abinit-10.4.5.tar.gz

FROM ubuntu:22.04

# Metadata
LABEL maintainer="HPC User"
LABEL description="ABINIT 10.4.5 computational materials science package (source build)"
LABEL version="10.4.5"
LABEL abinit.source="https://forge.abinit.org/abinit-10.4.5.tar.gz"

# Prevent interactive prompts during package installation
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=UTC

# Set locale to avoid warnings
ENV LANG=C.UTF-8
ENV LC_ALL=C.UTF-8

# Update package lists and install build dependencies
RUN apt-get update && apt-get install -y \
    # Essential build tools
    build-essential \
    autoconf \
    automake \
    libtool \
    pkg-config \
    cmake \
    make \
    wget \
    curl \
    git \
    vim \
    nano \
    # Compilers and development tools
    gfortran \
    gcc \
    g++ \
    # MPI implementation (OpenMPI)
    libopenmpi-dev \
    openmpi-bin \
    openmpi-common \
    # Mathematical libraries (optimized versions)
    liblapack-dev \
    libblas-dev \
    libopenblas-dev \
    # FFTW3 (essential for DFT calculations)
    libfftw3-dev \
    libfftw3-mpi-dev \
    # HDF5 for efficient I/O
    libhdf5-dev \
    libhdf5-mpi-dev \
    # NetCDF for scientific data formats
    libnetcdf-dev \
    libnetcdff-dev \
    # LibXC for exchange-correlation functionals
    libxc-dev \
    # Additional scientific libraries
    libgsl-dev \
    # Python ecosystem for analysis
    python3 \
    python3-pip \
    python3-dev \
    python3-numpy \
    python3-scipy \
    python3-matplotlib \
    python3-h5py \
    # Compression utilities
    bzip2 \
    xz-utils \
    && rm -rf /var/lib/apt/lists/*

# Set build environment variables for optimization
# Intel Xeon E5-2698 v4 supports AVX2 (Haswell architecture)
ENV CC=gcc
ENV CXX=g++
ENV FC=gfortran
ENV F77=gfortran

# IMPORTANT: Generic optimization flags for cross-platform building
# These work regardless of your build machine's CPU architecture
ENV CFLAGS="-O3 -fPIC"
ENV CXXFLAGS="-O3 -fPIC"
ENV FCFLAGS="-O3 -fPIC"
ENV F77FLAGS="-O3 -fPIC"

# TARGET CLUSTER OPTIMIZATION (Intel Xeon E5-2698 v4 Haswell/AVX2)
# Uncomment these lines ONLY if building directly on the target cluster:
# ENV CFLAGS="-O3 -march=haswell -mtune=haswell -mavx2 -fPIC"
# ENV CXXFLAGS="-O3 -march=haswell -mtune=haswell -mavx2 -fPIC"
# ENV FCFLAGS="-O3 -march=haswell -mtune=haswell -mavx2 -fPIC"
# ENV F77FLAGS="-O3 -march=haswell -mtune=haswell -mavx2 -fPIC"

# GENERIC OPTIMIZATION (for maximum compatibility):
# The current settings above use -O3 optimization without CPU-specific flags
# This ensures the binary runs on the target Intel Xeon E5-2698 v4 cluster
# Performance loss is minimal (~5-10%) compared to CPU-specific optimization

# Create build directory
WORKDIR /tmp/build

# Download and extract ABINIT 10.4.5 source code
RUN wget https://forge.abinit.org/abinit-10.4.5.tar.gz && \
    tar -xzf abinit-10.4.5.tar.gz && \
    cd abinit-10.4.5 && \
    ls -la

# Configure ABINIT build
WORKDIR /tmp/build/abinit-10.4.5
RUN ./configure \
    --prefix=/usr/local \
    --enable-mpi \
    --enable-openmp \
    --enable-optim \
    --with-mpi-prefix=/usr \
    --with-fft-flavor="fftw3" \
    --with-fft-libs="-lfftw3 -lfftw3f -lfftw3_mpi -lfftw3f_mpi" \
    --with-linalg-flavor="openblas" \
    --with-linalg-libs="-lopenblas" \
    --with-hdf5="/usr" \
    --with-netcdf="/usr" \
    --with-netcdf-fortran="/usr" \
    --with-libxc="/usr" \
    --enable-gw-dpc \
    --enable-libtetra \
    2>&1 | tee configure.log

# Check configure output for any issues
RUN echo "=== Configure Summary ===" && \
    tail -50 configure.log

# Build ABINIT (use multiple cores for faster compilation)
RUN make -j$(nproc) 2>&1 | tee build.log

# Install ABINIT
RUN make install

# Verify installation
RUN /usr/local/bin/abinit --version && \
    echo "ABINIT 10.4.5 successfully built and installed"

# Clean up build files to reduce image size
WORKDIR /
RUN rm -rf /tmp/build

# Create working directory for calculations
RUN mkdir -p /workspace
WORKDIR /workspace

# Set runtime environment variables
ENV PATH="/usr/local/bin:$PATH"
ENV OMP_NUM_THREADS=1
ENV OMPI_MCA_btl_vader_single_copy_mechanism=none

# Add useful aliases and environment setup to bashrc
RUN echo 'alias ll="ls -la"' >> /root/.bashrc && \
    echo 'alias abinit-version="/usr/local/bin/abinit --version"' >> /root/.bashrc && \
    echo 'alias abinit-help="/usr/local/bin/abinit --help"' >> /root/.bashrc && \
    echo 'export PATH="/usr/local/bin:$PATH"' >> /root/.bashrc && \
    echo 'echo "ABINIT 10.4.5 Docker Container"' >> /root/.bashrc && \
    echo 'echo "Available executables: abinit, anaddb, aim, cut3d, conducti, etc."' >> /root/.bashrc && \
    echo 'echo "Working directory: /workspace"' >> /root/.bashrc

# Install optional Python packages for post-processing
RUN pip3 install --no-cache-dir \
    abipy \
    pymatgen \
    ase \
    jupyter \
    netcdf4

# GPU Support (commented out - uncomment if CUDA is available)
# For NVIDIA GPU support, uncomment the following:
# RUN apt-get update && apt-get install -y \
#     nvidia-cuda-toolkit \
#     && rm -rf /var/lib/apt/lists/*
# 
# # Rebuild with GPU support (requires modifying configure step)
# # Add to configure: --enable-gpu --with-gpu-flavor="cuda-double"

# Default command - start bash shell with welcome message
CMD ["/bin/bash"]

# Usage examples:
# Build: docker build -t abinit-source:10.4.5 .
# Run: docker run -it -v $(pwd):/workspace abinit-source:10.4.5
# MPI Run: docker run -it -v $(pwd):/workspace abinit-source:10.4.5 mpirun -np 4 abinit < input.files
# Jupyter: docker run -it -p 8888:8888 -v $(pwd):/workspace abinit-source:10.4.5 jupyter notebook --ip=0.0.0.0 --allow-root
