# ABINIT Docker Image - Ubuntu Official Package
# Base: Ubuntu 22.04 LTS for maximum HPC compatibility
# Target: Intel Xeon E5-2698 v4 (AVX2 support)

FROM ubuntu:22.04

# Metadata
LABEL maintainer="HPC User"
LABEL description="ABINIT computational materials science package (Ubuntu official)"
LABEL version="ubuntu-official"

# Prevent interactive prompts during package installation
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=UTC

# Set locale to avoid warnings
ENV LANG=C.UTF-8
ENV LC_ALL=C.UTF-8

# Update package lists and install essential packages
RUN apt-get update && apt-get install -y \
    # Essential system packages
    build-essential \
    wget \
    curl \
    git \
    vim \
    nano \
    # Scientific computing essentials
    gfortran \
    gcc \
    g++ \
    # MPI implementation (OpenMPI)
    libopenmpi-dev \
    openmpi-bin \
    openmpi-common \
    # Mathematical libraries
    liblapack-dev \
    libblas-dev \
    libfftw3-dev \
    libfftw3-mpi-dev \
    # Data I/O libraries
    libhdf5-dev \
    libhdf5-mpi-dev \
    libnetcdf-dev \
    libnetcdff-dev \
    # Python for analysis (optional but useful)
    python3 \
    python3-pip \
    python3-numpy \
    python3-scipy \
    python3-matplotlib \
    && rm -rf /var/lib/apt/lists/*

# Install ABINIT from Ubuntu repository
# Note: This may not be the latest version but is well-tested and stable
RUN apt-get update && apt-get install -y \
    abinit \
    && rm -rf /var/lib/apt/lists/*

# Verify ABINIT installation
RUN abinit --version && \
    echo "ABINIT successfully installed from Ubuntu repository"

# Create working directory for calculations
RUN mkdir -p /workspace
WORKDIR /workspace

# Set environment variables for optimal performance
# Adjust OMP_NUM_THREADS based on your job requirements
ENV OMP_NUM_THREADS=1
ENV OMPI_MCA_btl_vader_single_copy_mechanism=none

# Add useful aliases and functions to bashrc
RUN echo 'alias ll="ls -la"' >> /root/.bashrc && \
    echo 'alias abinit-version="abinit --version"' >> /root/.bashrc && \
    echo 'export PATH=$PATH:/usr/bin' >> /root/.bashrc

# Performance optimization notes (commented out for compatibility):
# Uncomment the following if you want CPU-specific optimizations
# Note: This requires rebuilding ABINIT from source
# ENV CFLAGS="-O3 -march=native -mtune=native"
# ENV CXXFLAGS="-O3 -march=native -mtune=native" 
# ENV FCFLAGS="-O3 -march=native -mtune=native"

# For Intel Xeon E5-2698 v4 specifically (AVX2 support):
# ENV CFLAGS="-O3 -march=haswell -mtune=haswell -mavx2"
# ENV CXXFLAGS="-O3 -march=haswell -mtune=haswell -mavx2"
# ENV FCFLAGS="-O3 -march=haswell -mtune=haswell -mavx2"

# Default command - start bash shell
CMD ["/bin/bash"]

# Usage examples:
# Build: docker build -t abinit-ubuntu .
# Run: docker run -it -v $(pwd):/workspace abinit-ubuntu
# MPI Run: docker run -it -v $(pwd):/workspace abinit-ubuntu mpirun -np 4 abinit < input.files
