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
# Navigate to this directory
cd abinit-ubuntu/

# Build the image (this will take 5-10 minutes)
docker build -t abinit-ubuntu .

# Verify the build succeeded
docker images | grep abinit-ubuntu
```

### 2. Test the Installation

```bash
# Run a quick test to verify ABINIT works
docker run --rm abinit-ubuntu abinit --version
```

### 3. Run Interactive Session

```bash
# Start an interactive container with your current directory mounted
docker run -it -v $(pwd):/workspace abinit-ubuntu

# Inside the container, you can now run ABINIT commands:
# abinit --version
# abinit input.files
```

## Usage Examples

### Basic ABINIT Calculation

```bash
# Prepare your input files on the host system
# Then run the container with volume mounting:
docker run -it -v /path/to/your/calculations:/workspace abinit-ubuntu

# Inside container:
abinit < input.files
```

### MPI Parallel Calculations

```bash
# Run ABINIT with multiple processes
docker run -it -v $(pwd):/workspace abinit-ubuntu mpirun -np 4 abinit < input.files

# For larger calculations, adjust -np based on your needs
docker run -it -v $(pwd):/workspace abinit-ubuntu mpirun -np 8 abinit < input.files
```

## Sharing Your Image

### 1. Push to Docker Hub

```bash
# Tag your image with your Docker Hub username
docker tag abinit-ubuntu YOUR_USERNAME/abinit-ubuntu:latest

# Login to Docker Hub
docker login

# Push the image
docker push YOUR_USERNAME/abinit-ubuntu:latest
```

### 2. Template Email for System Administrator

```
Subject: Docker Image for ABINIT - Ready for HPC Deployment

Dear [System Administrator Name],

I have prepared a Docker image containing ABINIT for use on our HPC cluster. The image is ready for deployment and has been tested for compatibility.

**Docker Image Details:**
- Image: YOUR_USERNAME/abinit-ubuntu:latest
- Base: Ubuntu 22.04 LTS
- ABINIT: Official Ubuntu package version
- Size: ~500MB (estimated)

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
