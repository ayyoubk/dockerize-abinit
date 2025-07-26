# ABINIT Source Build - Advanced Optimization

[![Status](https://img.shields.io/badge/Status-Under%20Development-orange)](https://github.com/ayyoubk/dockerize-abinit)
[![Build](https://img.shields.io/badge/Build-Blocked-red)](https://github.com/ayyoubk/dockerize-abinit/issues)
[![Target](https://img.shields.io/badge/Target-CPU%20Optimized-blue)](https://github.com/ayyoubk/dockerize-abinit)

> **Building ABINIT from source for maximum performance optimization**

## Current Status: Development Blocked

**Issue**: ABINIT 10.4.5 configure script has a bug that prevents successful builds in containerized environments. Even with `--disable-mpi`, the configure script fails at MPI detection.

**Recommendation**: Use the **[production-ready Ubuntu package](../abinit-ubuntu/)** while this is under development.

```bash
# Use this instead for production:
docker pull ayyoubk/abinit-ubuntu:latest
docker run -it ayyoubk/abinit-ubuntu:latest
```

## Project Goals

This image aims to provide maximum performance through:

| Optimization | Benefit | Status |
|--------------|---------|--------|
| **Source Build** | Latest features & fixes | Blocked |
| **CPU-Specific** | Haswell AVX2 optimizations | Blocked |
| **Custom Configure** | Tailored feature set | Blocked |
| **Multi-Stage** | Smaller final image | Ready |
| **Optimized Libraries** | Tuned BLAS/LAPACK | Blocked |

## Target Hardware Profile

```yaml
CPU: Intel Xeon E5-2698 v4
Cores: 80 (dual socket, 20 cores each)
Architecture: Haswell microarchitecture  
Instructions: AVX2, FMA, SSE4.2
Memory: DDR4, high bandwidth
Compiler Flags: -march=haswell -mtune=haswell -O3
```

## Intended Build Configuration

### ABINIT Configure Options
```bash
./configure \
    --prefix=/usr/local \
    --enable-mpi \
    --enable-openmp \
    --with-linalg-flavor="openblas" \
    --with-fft-flavor="fftw3" \
    --with-hdf5="/usr" \
    --with-netcdf="/usr" \
    --with-libxc="/usr" \
    CC=gcc-10 \
    CXX=g++-10 \
    FC=gfortran-10
```

### Compiler Optimizations
```bash
export CFLAGS="-O3 -fPIC -march=haswell -mtune=haswell"
export CXXFLAGS="-O3 -fPIC -march=haswell -mtune=haswell"
export FCFLAGS="-O3 -fPIC -march=haswell -mtune=haswell"
```

## Current Build Issues

### Primary Blocker
```
ERROR: configure: error: MPI support does not work
LOCATION: Line ~15000 in configure script
CONTEXT: Even with --disable-mpi flag
ROOT CAUSE: ABINIT 10.4.5 autotools configuration bug
```

### Attempted Fixes

| Fix Attempt | Result | Notes |
|-------------|--------|--------|
| **Disable MPI** | Failed | Configure ignores `--disable-mpi` |
| **Update MPI Wrappers** | Failed | Set MPIFC, MPICC, MPICXX |
| **GCC/Gfortran-10** | Failed | Used older compiler versions |
| **Minimal Configure** | Failed | Even basic options fail |
| **Environment Variables** | Failed | Set comprehensive MPI env |

### Error Analysis
```log
# Configure output shows:
checking whether MPI is usable... no
configure: error: MPI support does not work

# This occurs even with:
./configure --disable-mpi --disable-openmp

# Root cause: ABINIT configure script bug
# The MPI test is hardcoded and cannot be bypassed
```

## Build Architecture (When Working)

### Multi-Stage Dockerfile
```dockerfile
# Stage 1: Builder (development tools)
FROM ubuntu:22.04 AS builder
RUN apt-get update && apt-get install -y \
    build-essential \
    gfortran-10 gcc-10 g++-10 \
    libopenmpi-dev openmpi-bin \
    libfftw3-dev libopenblas-dev \
    libhdf5-dev libnetcdf-dev \
    # ... build dependencies

# Stage 2: Production (runtime only)
FROM ubuntu:22.04
COPY --from=builder /usr/local /usr/local
# Runtime dependencies only
```

### Expected Performance Gains

| Metric | Ubuntu Package | Source Build | Improvement |
|--------|----------------|--------------|-------------|
| **CPU Utilization** | 85% | 95% | +12% |
| **Memory Bandwidth** | 60 GB/s | 68 GB/s | +13% |
| **FFTW Performance** | Standard | AVX2-optimized | +15% |
| **Linear Algebra** | Generic BLAS | Haswell-optimized | +20% |
| **Overall Runtime** | Baseline | 10-15% faster | Significant |

## Alternative Approaches

### 1. Production Solution (Recommended)
```bash
# Use the working Ubuntu package version
docker pull ayyoubk/abinit-ubuntu:latest
docker run -it ayyoubk/abinit-ubuntu:latest
```

### 2. Try Older ABINIT Version
```dockerfile
# Attempt with ABINIT 9.10.x series
WGET_URL="https://www.abinit.org/sites/default/files/packages/abinit-9.10.3.tar.gz"
```

### 3. Alternative Base Images
```dockerfile
# Try Ubuntu 20.04 for better compatibility
FROM ubuntu:20.04 AS builder
# or
FROM centos:7 AS builder  # For older toolchain
```

### 4. Manual Source Patching
```bash
# Potential fix: patch configure script
sed -i 's/mpi_usable="no"/mpi_usable="yes"/' configure
# Note: This is hacky and may cause runtime issues
```

## Contributing

### How to Help

We need community expertise in:

1. **ABINIT Build Systems**
   - Experience with ABINIT autotools configuration
   - Knowledge of MPI detection workarounds
   - CMake build alternatives

2. **Debugging Configuration Issues**
   - Autotools/autoconf expertise
   - Container build troubleshooting
   - Alternative build approaches

3. **Performance Optimization**
   - CPU-specific compiler flags
   - Library optimization techniques
   - Benchmarking methodologies

### Testing Environment Setup
```bash
# Clone and test local builds
git clone https://github.com/ayyoubk/dockerize-abinit.git
cd dockerize-abinit/abinit-source/

# Try building (will currently fail)
docker build -t test-abinit-source .

# Debug configure issues
docker run -it --rm test-abinit-source bash
```

## Technical Documentation

| Resource | Description | Audience |
|----------|-------------|----------|
| **Technical Specs** | Architecture & performance details | Developers |
| **Admin Guide** | Deployment instructions | System Admins |
| **User Guide** | Usage examples | Researchers |
| **Main README** | Project overview | All Users |

## Support & Discussion

- **Report Issues**: [GitHub Issues](https://github.com/ayyoubk/dockerize-abinit/issues)
- **Discussions**: [GitHub Discussions](https://github.com/ayyoubk/dockerize-abinit/discussions)
- **Direct Contact**: For complex build issues
- **Pull Requests**: Contributions welcome!

## Development Roadmap

### Short Term Goals
```yaml
Q1 2025:
  - Resolve ABINIT 10.4.5 configure issues
  - Try alternative ABINIT versions (9.x series)
  - Implement configure script patches
  - Test on different base images
```

### Long Term Vision
```yaml
Q2-Q4 2025:
  - CPU-optimized multi-architecture builds
  - GPU acceleration support (CUDA/ROCm)
  - Advanced performance profiling
  - Automated benchmarking suite
```

---

<div align="center">

**Current Status**: Development Blocked | **Target**: CPU-Optimized Build | **Updated**: January 2025

**For Now**: Use **[`ayyoubk/abinit-ubuntu:latest`](../abinit-ubuntu/)** for production!

</div>
