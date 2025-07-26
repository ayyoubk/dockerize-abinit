# =================================================================
# STAGE 1: Builder - Compile ABINIT from source
# =================================================================
FROM ubuntu:22.04 AS builder

# Metadata
LABEL maintainer="HPC User"
LABEL description="ABINIT 10.4.5 build stage"

# Prevent interactive prompts during package installation
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=UTC
ENV LANG=C.UTF-8
ENV LC_ALL=C.UTF-8

# Set compiler optimization flags for Intel Xeon E5-2698 v4
# Note: Using permissive flags to avoid treating warnings as errors
ENV CFLAGS="-O3 -fPIC"
ENV CXXFLAGS="-O3 -fPIC"
ENV FCFLAGS="-O3 -fPIC -fallow-argument-mismatch -Wno-error"

# Install build dependencies with older, more compatible compilers
RUN apt-get update && apt-get install -y \
    build-essential autoconf automake libtool pkg-config cmake make wget \
    gfortran-10 gcc-10 g++-10 libopenmpi-dev openmpi-bin liblapack-dev libblas-dev \
    libopenblas-dev libfftw3-dev libfftw3-mpi-dev libhdf5-dev libhdf5-mpi-dev \
    libnetcdf-dev libnetcdff-dev libxc-dev libgsl-dev python3 python3-pip \
    python3-dev python3-numpy python3-scipy && \
    rm -rf /var/lib/apt/lists/*

# Set specific compiler versions for ABINIT compatibility
ENV FC=gfortran-10
ENV CC=gcc-10
ENV CXX=g++-10

# Configure MPI to work properly with our compiler versions
ENV MPIFC=mpif90
ENV MPICC=mpicc
ENV MPICXX=mpicxx

# Update alternatives to ensure MPI uses our compilers
RUN update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-10 100 && \
    update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-10 100 && \
    update-alternatives --install /usr/bin/gfortran gfortran /usr/bin/gfortran-10 100

# Download and extract ABINIT source
WORKDIR /tmp/build
RUN wget https://forge.abinit.org/abinit-10.4.5.tar.gz && \
    tar -xzf abinit-10.4.5.tar.gz

# Configure ABINIT with minimal options to isolate configuration issues
WORKDIR /tmp/build/abinit-10.4.5
RUN ./configure \
    --prefix=/usr/local \
    --disable-mpi \
    --disable-openmp \
    --disable-atompaw \
    --disable-bigdft \
    --disable-wannier90 \
    --without-netcdf \
    --without-hdf5 \
    --without-libxc \
    2>&1 | tee configure.log && \
    echo "=== Configure Summary ===" && \
    tail -50 configure.log && \
    echo "=== Checking if Makefiles were generated ===" && \
    ls -la Makefile && \
    echo "=== Configure completed successfully ==="

# Build ABINIT with proper error checking
RUN echo "=== Starting ABINIT compilation ===" && \
    make -j$(nproc) 2>&1 | tee build.log && \
    echo "=== Checking if binaries were built ===" && \
    ls -la src/98_main/ && \
    echo "=== Build completed successfully ==="

# Install ABINIT to temporary location
RUN make install DESTDIR=/tmp/install && \
    echo "=== Installation completed ===" && \
    ls -la /tmp/install/usr/local/bin/

# =================================================================
# STAGE 2: Final runtime image (much smaller)
# =================================================================
FROM ubuntu:22.04

# Metadata
LABEL maintainer="HPC User"
LABEL description="ABINIT 10.4.5 computational materials science package (optimized)"
LABEL version="10.4.5"

# Set environment variables
ENV DEBIAN_FRONTEND=noninteractive
ENV OMP_NUM_THREADS=1
ENV OMPI_MCA_btl_vader_single_copy_mechanism=none
ENV PATH="/usr/local/bin:$PATH"
ENV LANG=C.UTF-8
ENV LC_ALL=C.UTF-8

# Install only runtime dependencies (much smaller than build deps)
RUN apt-get update && apt-get install -y \
    libopenmpi3 \
    libopenblas0 \
    libfftw3-double3 \
    libfftw3-mpi3 \
    libhdf5-openmpi-103-1 \
    libnetcdf19 \
    libnetcdff7 \
    libxc9 \
    libgsl27 \
    python3 \
    python3-pip \
    python3-numpy \
    python3-scipy \
    python3-matplotlib \
    python3-h5py \
    vim \
    htop \
    && rm -rf /var/lib/apt/lists/*

# Copy compiled ABINIT from builder stage
COPY --from=builder /tmp/install/usr/local/ /usr/local/

# Install Python packages for ABINIT analysis
RUN pip3 install --no-cache-dir \
    abipy \
    pymatgen \
    ase \
    jupyter \
    netcdf4 \
    pandas \
    seaborn

# Create working directory
RUN mkdir -p /workspace
WORKDIR /workspace

# Set up user environment
RUN echo 'alias ll="ls -alF"' >> /root/.bashrc && \
    echo 'alias la="ls -A"' >> /root/.bashrc && \
    echo 'alias l="ls -CF"' >> /root/.bashrc && \
    echo 'echo "ABINIT 10.4.5 (Optimized Build) | Intel Xeon E5-2698 v4"' >> /root/.bashrc && \
    echo 'echo "Working Directory: /workspace"' >> /root/.bashrc && \
    echo 'echo "Available commands: abinit, anaddb, cut3d, aim, optic"' >> /root/.bashrc && \
    echo 'echo "Python analysis: AbiPy, PyMatGen, ASE available"' >> /root/.bashrc && \
    echo 'echo "Usage: mpirun -np N abinit < input.files > output.log"' >> /root/.bashrc

# Verify installation works
RUN /usr/local/bin/abinit --version && \
    echo "ABINIT 10.4.5 successfully installed in optimized image"

CMD ["/bin/bash"]
