# 🧪 ABINIT Docker Solutions for HPC Clusters

[![Docker Hub](https://img.shields.io/badge/Docker%20Hub-ayyoubk%2Fabinit--ubuntu-blue?logo=docker)](https://hub.docker.com/r/ayyoubk/abinit-ubuntu)
[![GitHub](https://img.shields.io/badge/GitHub-dockerize--abinit-black?logo=github)](https://github.com/ayyoubk/dockerize-abinit)
[![License](https://img.shields.io/badge/License-Open%20Source-green)](LICENSE)
[![Status](https://img.shields.io/badge/Status-Production%20Ready-brightgreen)](https://github.com/ayyoubk/dockerize-abinit)

> **Professional Docker containerization solutions for ABINIT computational materials science on HPC clusters**

## 🎯 Overview

**ABINIT** is a leading computational materials science package that implements density functional theory (DFT) for studying electronic, structural, and optical properties of materials. This project provides **production-ready Docker solutions** to simplify ABINIT deployment and usage on High-Performance Computing (HPC) systems.

### ✨ Key Benefits

- 🚀 **Rapid Deployment**: Get ABINIT running in minutes, not hours
- 🔒 **Reproducible Environment**: Consistent results across different systems
- 📦 **No Dependencies**: Everything bundled in a container
- 🎛️ **HPC Optimized**: Designed for cluster environments
- 📚 **Complete Documentation**: Comprehensive guides for admins and users

## 🎯 Target Environment

| Component | Specification |
|-----------|---------------|
| **OS** | Ubuntu 24.04.1 LTS (adaptable) |
| **CPU** | Intel Xeon E5-2698 v4 (80 cores, AVX2) |
| **Architecture** | x86_64 Haswell microarchitecture |
| **Memory** | Optimized for large-scale calculations |
| **Deployment** | Docker containerization |

## 📦 Available Solutions

### 🟢 Production Ready: ABINIT Ubuntu Package

[![Docker Pulls](https://img.shields.io/docker/pulls/ayyoubk/abinit-ubuntu)](https://hub.docker.com/r/ayyoubk/abinit-ubuntu)
[![Image Size](https://img.shields.io/docker/image-size/ayyoubk/abinit-ubuntu/latest)](https://hub.docker.com/r/ayyoubk/abinit-ubuntu)

```bash
# Quick start - pull and run
docker pull ayyoubk/abinit-ubuntu:latest
docker run -it ayyoubk/abinit-ubuntu:latest
```

| Feature | Details |
|---------|----------|
| **Image** | `ayyoubk/abinit-ubuntu:latest` |
| **Base** | Ubuntu 22.04 LTS |
| **Build Time** | ~5 minutes |
| **Image Size** | ~1.5GB |
| **Status** | ✅ Production Ready |

**Includes:**
- 🧮 Pre-compiled ABINIT with standard optimizations
- 🚀 OpenMPI support for parallel calculations  
- 📊 Scientific libraries (FFTW3, BLAS, LAPACK)
- 🐍 Python environment with analysis tools
- 📈 AbiPy, PyMatGen for post-processing

### 🟡 In Development: ABINIT Source Build

```bash
# Source build (under development)
# Custom optimizations for Intel Xeon E5-2698 v4
```

| Feature | Details |
|---------|----------|
| **Target** | CPU-specific optimizations |
| **ABINIT Version** | 10.4.5 from source |
| **Status** | 🚧 In Development |
| **ETA** | Future release |

## 🚀 Quick Start Guide

### For System Administrators

```bash
# 1. Pull the ABINIT image
docker pull ayyoubk/abinit-ubuntu:latest

# 2. Test the installation
docker run --rm ayyoubk/abinit-ubuntu:latest abinit --version

# 3. Run interactive session
docker run -it ayyoubk/abinit-ubuntu:latest
```

### For ABINIT Users

```bash
# Run ABINIT calculation with mounted workspace
docker run -v $(pwd):/workspace ayyoubk/abinit-ubuntu:latest abinit < input.files

# Parallel calculation with MPI
docker run -v $(pwd):/workspace ayyoubk/abinit-ubuntu:latest mpirun -np 4 abinit < input.files
```

## 📚 Documentation

| Guide | Description | Audience |
|--------|-------------|----------|
| **[📋 System Admin Guide](docs/admin-guide.md)** | Installation, deployment, cluster management | HPC Administrators |
| **[👨‍💻 ABINIT User Guide](docs/user-guide.md)** | Running calculations, examples, workflows | Researchers & Scientists |
| **[🔧 Technical Specifications](docs/technical-specs.md)** | Architecture, optimization details | Technical Users |

## 📁 Repository Structure

```
dockerize-abinit/
├── 📄 README.md                    # Project overview (this file)
├── 📁 docs/                        # Comprehensive documentation
│   ├── 📋 admin-guide.md           # System administrator guide  
│   ├── 👨‍💻 user-guide.md             # ABINIT user guide
│   └── 🔧 technical-specs.md       # Technical specifications
├── 📁 abinit-ubuntu/               # ✅ Production ready solution
│   ├── 🐳 Dockerfile               # Ubuntu package build
│   └── 📄 README.md               # Ubuntu-specific docs
└── 📁 abinit-source/               # 🚧 Development solution
    ├── 🐳 Dockerfile               # Source build (in progress)
    └── 📄 README.md               # Source build docs
```

## 🌟 Features & Capabilities

### ✅ Ready to Use
- **Immediate deployment** on any Docker-enabled system
- **Pre-configured environment** with all dependencies
- **MPI support** for parallel calculations
- **Scientific Python stack** for analysis

### 🔬 Scientific Computing
- **Density Functional Theory (DFT)** calculations
- **Electronic structure** analysis
- **Materials properties** prediction
- **Structural optimization** workflows

### 🖥️ HPC Integration
- **Cluster-ready** containerization
- **Resource management** compatibility
- **Scalable** parallel execution
- **Batch job** integration

## 📞 Support & Community

| Resource | Link | Description |
|----------|------|-------------|
| **Issues** | [GitHub Issues](https://github.com/ayyoubk/dockerize-abinit/issues) | Bug reports & feature requests |
| **Discussions** | [GitHub Discussions](https://github.com/ayyoubk/dockerize-abinit/discussions) | Questions & community support |
| **Docker Hub** | [ayyoubk/abinit-ubuntu](https://hub.docker.com/r/ayyoubk/abinit-ubuntu) | Container repository |
| **Contributions** | [Contributing Guide](CONTRIBUTING.md) | How to contribute |

## 📜 License & Acknowledgments

- **Project License**: Open Source
- **ABINIT License**: GNU General Public License
- **Author**: Ayyoub Al keyyam, Software Engineer
- **Acknowledgments**: ABINIT development team, Ubuntu/Docker communities, HPC community

---

<div align="center">

**🎯 Status**: ✅ Production Ready | **📅 Last Updated**: July 2025 | **🐳 Docker Hub**: [ayyoubk/abinit-ubuntu](https://hub.docker.com/r/ayyoubk/abinit-ubuntu)

**Made with ❤️ for the computational materials science community**

</div>
