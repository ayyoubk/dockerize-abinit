# ABINIT Ubuntu Package Docker Image

This Docker image provides ABINIT using Ubuntu's official package repository. This is the **quickest setup** but may not have the latest ABINIT version.

## Prerequisites

1. **Docker Desktop** installed on your local machine
   - Download from: https://www.docker.com/products/docker-desktop
   - Ensure Docker is running before proceeding

2. **Docker Hub Account** (optional, for sharing images)
   - Create account at: https://hub.docker.com

## Quick Start

### 1. Build the Docker Image

```bash
# Pull and run - it's that simple!
docker pull ayyoubk/abinit-ubuntu:latest
docker run -it ayyoubk/abinit-ubuntu:latest

# Run a calculation with your files
docker run --rm -v $(pwd):/workspace ayyoubk/abinit-ubuntu:latest abinit < input.files
```

## 📦 What's Included

| Component | Version | Purpose |
|-----------|---------|----------|
| **🧮 ABINIT** | Latest stable | Core DFT calculations |
| **🚀 OpenMPI** | 4.1+ | Parallel processing |
| **🐍 Python 3** | 3.10+ | Scientific computing |
| **📊 AbiPy** | Latest | ABINIT analysis toolkit |
| **⚛️ PyMatGen** | Latest | Materials analysis |
| **📈 Jupyter** | Latest | Interactive notebooks |
| **🔢 NumPy/SciPy** | Latest | Scientific libraries |

### 🏗️ System Libraries
- **FFTW3**: Fast Fourier transforms
- **BLAS/LAPACK**: Linear algebra routines  
- **HDF5/NetCDF**: High-performance I/O
- **OpenBLAS**: Optimized math operations

## 🎯 Target Environment

| Specification | Requirement |
|---------------|-------------|
| **OS** | Ubuntu 24.04.1 LTS (adaptable) |
| **CPU** | Intel Xeon E5-2698 v4 (80 cores) |
| **Memory** | 4GB min, 16GB+ recommended |
| **Storage** | 10GB+ for images/workspaces |
| **Network** | Docker networking support |

## 💡 Usage Examples

### 🖥️ Interactive Session
```bash
# Start interactive container with workspace mount
docker run -it -v $(pwd):/workspace ayyoubk/abinit-ubuntu:latest

# Inside container - check everything works
abinit --version
mpirun --version
python3 -c "import abipy; print('AbiPy ready!')"
```

### ⚡ Parallel Calculation
```bash
# Run with 8 MPI processes
docker run --rm \
  -v $(pwd):/workspace \
  ayyoubk/abinit-ubuntu:latest \
  mpirun -np 8 abinit < input.files > output.log
```

### 📊 Python Analysis Session
```bash
# Start Jupyter for post-processing
docker run -p 8888:8888 \
  -v $(pwd):/workspace \
  ayyoubk/abinit-ubuntu:latest \
  jupyter notebook --ip=0.0.0.0 --allow-root --no-browser
```

### 🔄 Batch Processing
```bash
# Process multiple calculations
for calc in calc_*/; do
  echo "Processing $calc"
  docker run --rm -v $(pwd)/$calc:/workspace \
    ayyoubk/abinit-ubuntu:latest \
    abinit < input.files > output.log 2>&1
done
```

## 🛠️ Build Information

### 🏗️ Docker Build
```bash
# Build locally (optional - image available on Docker Hub)
cd abinit-ubuntu/
docker build -t ayyoubk/abinit-ubuntu:latest .
```

### 📋 Build Details
- **Base Image**: Ubuntu 22.04 LTS
- **Build Method**: Ubuntu package manager (apt)
- **Build Time**: ~5 minutes
- **Final Size**: ~1.5GB
- **Optimization**: Multi-layer caching, cleanup

## 🔍 Container Internals

### 📁 Directory Structure
```
/usr/bin/abinit           # ABINIT executables
/usr/lib/python3.10/      # Python packages
/workspace/               # User workspace (mount point)
/data/                    # Data directory (optional mount)
/results/                 # Results directory (optional mount)
```

### 🌍 Environment Variables
```bash
PATH=/usr/local/bin:/usr/bin:$PATH
OMP_NUM_THREADS=1
OMPI_MCA_btl_vader_single_copy_mechanism=none
PYTHONPATH=/usr/local/lib/python3.10/site-packages:$PYTHONPATH
```

## 🔧 Advanced Configuration

### 🎛️ Resource Limits
```bash
# Set CPU and memory limits
docker run --cpus="8" --memory="16g" \
  -v $(pwd):/workspace \
  ayyoubk/abinit-ubuntu:latest \
  mpirun -np 8 abinit < input.files
```

### 🔒 Security Configuration
```bash
# Run as specific user
docker run --user $(id -u):$(id -g) \
  -v $(pwd):/workspace \
  ayyoubk/abinit-ubuntu:latest \
  abinit < input.files
```

### 🌐 Network Configuration
```bash
# For HPC clusters - use host networking
docker run --network=host \
  -v $(pwd):/workspace \
  ayyoubk/abinit-ubuntu:latest

**Deployment Commands:**

Pull the image:
```
docker pull YOUR_USERNAME/abinit-ubuntu:latest
```

Run interactive session:
```
docker run -it -v /path/to/user/data:/workspace YOUR_USERNAME/abinit-ubuntu:latest
```

Run ABINIT calculation:
```
docker run -it -v /path/to/calculation:/workspace YOUR_USERNAME/abinit-ubuntu:latest abinit < input.files
```

Run MPI parallel calculation:
```
docker run -it -v /path/to/calculation:/workspace YOUR_USERNAME/abinit-ubuntu:latest mpirun -np [NUM_CORES] abinit < input.files
# Example: mpirun -np 16 abinit < input.files
```

**Benefits:**
- Eliminates dependency conflicts with existing cluster software
- Consistent environment across different nodes
- Easy to replicate and maintain
- Contains all necessary scientific libraries (OpenMPI, FFTW3, HDF5, NetCDF)

**Resource Requirements:**
- Disk space: ~1GB per image
- RAM: Standard ABINIT requirements
- CPU: Compatible with Intel Xeon E5-2698 v4 (no special requirements)

Please let me know if you need any additional information or modifications to the deployment procedure.

Best regards,
[Your Name]
```

## Advanced Usage

### Custom Compiler Optimizations

If you want to optimize for your specific Intel Xeon E5-2698 v4 processors, you can rebuild the image with custom flags by editing the Dockerfile:

```dockerfile
# Uncomment these lines in the Dockerfile for CPU-specific optimization:
ENV CFLAGS="-O3 -march=haswell -mtune=haswell -mavx2"
ENV CXXFLAGS="-O3 -march=haswell -mtune=haswell -mavx2" 
ENV FCFLAGS="-O3 -march=haswell -mtune=haswell -mavx2"
```

Note: This requires rebuilding ABINIT from source (see the `abinit-source` directory).

## Troubleshooting

### Common Issues

1. **Permission Errors**
   ```bash
   # Ensure your data directory has correct permissions
   chmod -R 755 /path/to/your/calculations
   ```

2. **MPI Warnings**
   ```bash
   # If you see MPI warnings, they're usually harmless
   # You can suppress them with:
   export OMPI_MCA_btl_vader_single_copy_mechanism=none
   ```

3. **Out of Memory**
   ```bash
   # For large calculations, increase Docker's memory limit
   # In Docker Desktop: Settings -> Resources -> Memory
   ```

## What's Included

- **ABINIT**: Official Ubuntu package version
- **OpenMPI**: For parallel calculations
- **Scientific libraries**: FFTW3, LAPACK, BLAS, HDF5, NetCDF
- **Python tools**: NumPy, SciPy, Matplotlib for analysis
- **Text editors**: vim, nano for file editing

## Performance Notes

- This image uses Ubuntu's pre-compiled ABINIT package
- For maximum performance, consider the source-built version in `../abinit-source/`
- The source version allows CPU-specific optimizations and includes the latest features
