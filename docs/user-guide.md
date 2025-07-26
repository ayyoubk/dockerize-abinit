# 👨‍💻 ABINIT User Guide

> **Complete guide for researchers and scientists using ABINIT Docker containers**

## 🎯 Overview

This guide provides ABINIT users with comprehensive instructions for running calculations, setting up workflows, and analyzing results using the containerized ABINIT environment. Whether you're new to ABINIT or an experienced user, this guide will help you leverage the power of containerized computational materials science.

## 🚀 Quick Start

### Your First ABINIT Calculation

```bash
# 1. Create a working directory
mkdir my_abinit_calculation
cd my_abinit_calculation

# 2. Run ABINIT interactively
docker run -it -v $(pwd):/workspace ayyoubk/abinit-ubuntu:latest

# 3. Inside the container, check ABINIT version
abinit --version
```

### Running a Basic Calculation

```bash
# Run ABINIT with input files from your host directory
docker run --rm -v $(pwd):/workspace ayyoubk/abinit-ubuntu:latest abinit < input.files
```

## 📋 Container Overview

### What's Included

| Component | Description | Version |
|-----------|-------------|---------|
| **ABINIT** | Main DFT calculation engine | Latest stable |
| **OpenMPI** | Parallel processing support | 4.1+ |
| **Python 3** | Scientific computing environment | 3.10+ |
| **AbiPy** | ABINIT analysis toolkit | Latest |
| **PyMatGen** | Materials analysis framework | Latest |
| **Jupyter** | Interactive notebooks | Latest |
| **Scientific Libraries** | NumPy, SciPy, Matplotlib | Latest |

### Available Executables

```bash
# Core ABINIT executables
abinit          # Main DFT program
anaddb          # Analysis of derivative databases  
cut3d           # 3D data visualization
aim             # Atoms in molecules analysis
optic           # Optical properties calculation
conducti        # Transport properties
atdep           # Lattice dynamics
```

## 🔧 Basic Usage Patterns

### Pattern 1: Interactive Session

```bash
# Start interactive container
docker run -it \
  -v $(pwd):/workspace \
  -v ~/abinit-data:/data \
  --name my-abinit-session \
  ayyoubk/abinit-ubuntu:latest

# Inside container:
cd /workspace
abinit < input.files > output.log &
tail -f output.log
```

### Pattern 2: Single Calculation

```bash
# Run calculation and exit
docker run --rm \
  -v $(pwd):/workspace \
  ayyoubk/abinit-ubuntu:latest \
  bash -c "cd /workspace && abinit < input.files > output.log 2>&1"
```

### Pattern 3: Parallel Calculation

```bash
# Run with MPI (4 processes)
docker run --rm \
  -v $(pwd):/workspace \
  ayyoubk/abinit-ubuntu:latest \
  mpirun -np 4 abinit < input.files
```

### Pattern 4: Batch Processing

```bash
# Process multiple calculations
for dir in calc_*/; do
  echo "Running calculation in $dir"
  docker run --rm \
    -v $(pwd)/$dir:/workspace \
    ayyoubk/abinit-ubuntu:latest \
    abinit < input.files > output.log 2>&1
done
```

## 📁 Working with Files

### Directory Structure Recommendations

```
my_project/
├── input.files          # ABINIT input file list
├── system.in            # Main input file
├── pseudopotentials/    # Pseudopotential files
│   ├── Si.psp8
│   └── O.psp8
├── results/            # Output files
└── analysis/           # Post-processing scripts
```

### Essential Input Files

#### input.files
```
system.in        # Main input file
system.out       # Main output file  
systemi          # Input wavefunction file
systemo          # Output wavefunction file
systemt          # Temporary file
../pseudopotentials/  # Path to pseudopotentials
```

### Volume Mounting Best Practices

```bash
# Recommended volume mounts
docker run -it \
  -v $(pwd)/input:/workspace \              # Input files
  -v $(pwd)/results:/results \              # Output files  
  -v $(pwd)/pseudopotentials:/pp:ro \       # Pseudopotentials (read-only)
  -v $(pwd)/analysis:/analysis \            # Analysis scripts
  ayyoubk/abinit-ubuntu:latest
```

## 🧮 Example Calculations

### Example 1: Silicon Crystal Structure Optimization

Create the input files:

**input.files**
```
si_opt.in
si_opt.out
si_i
si_o
si_t
../pseudopotentials/
```

**si_opt.in**
```
# Silicon crystal - structure optimization
ndtset 2

# Dataset 1: Self-consistent calculation
iscf1    7
nband1   8
nstep1   20
toldfe1  1.0d-8

# Dataset 2: Structure optimization  
iscf2    7
ionmov2  3
ntime2   20
tolmxf2  1.0d-6
nband2   8

# Common variables
acell 3*10.26
rprim 0.0  0.5  0.5
      0.5  0.0  0.5  
      0.5  0.5  0.0
natom 2
typat 1 1
xred  0.0  0.0  0.0
      0.25 0.25 0.25
ntypat 1
znucl 14

# k-points
ngkpt 4 4 4
nshiftk 1
shiftk 0.5 0.5 0.5

# Plane wave basis
ecut 15.0
ecutsm 0.5

# Pseudopotential
pp_dirpath "../pseudopotentials/"
pseudos "Si.psp8"
```

**Run the calculation:**

```bash
# Create directory structure
mkdir -p si_optimization/{pseudopotentials,results}
cd si_optimization

# Copy input files (create the files above)
# Download Si pseudopotential to pseudopotentials/ directory

# Run calculation
docker run --rm \
  -v $(pwd):/workspace \
  ayyoubk/abinit-ubuntu:latest \
  abinit < input.files
```

### Example 2: Band Structure Calculation

**bands.in**
```
# Silicon band structure calculation
ndtset 3

# Dataset 1: Ground state calculation
nband1   8
toldfe1  1.0d-8

# Dataset 2: Non-self-consistent calculation  
getden2   1
iscf2    -2
nband2   12
tolwfr2  1.0d-16

# Dataset 3: Band structure along high-symmetry path
getden3   1
iscf3    -2  
nband3   12
tolwfr3  1.0d-16

# High-symmetry k-point path for band structure
kptopt3  -7
ndivsm3  10
kptbounds3
  0.5  0.5  0.5  # L point
  0.0  0.0  0.0  # Gamma point  
  0.5  0.0  0.5  # X point
  0.625 0.25 0.625 # U point
  0.5  0.25  0.75  # K point
  0.0  0.0  0.0   # Gamma point

# Common parameters (same as previous example)
# ... (copy from si_opt.in)
```

### Example 3: Phonon Calculation

```bash
# Multi-step phonon calculation workflow
docker run --rm -v $(pwd):/workspace ayyoubk/abinit-ubuntu:latest bash -c "
  # Step 1: Ground state
  abinit < gs.files
  
  # Step 2: Response function  
  abinit < rf.files
  
  # Step 3: Analysis with anaddb
  anaddb < anaddb.files
"
```

## 🐍 Python Analysis with AbiPy

### Interactive Analysis Session

```bash
# Start Jupyter notebook server
docker run -it -p 8888:8888 \
  -v $(pwd):/workspace \
  ayyoubk/abinit-ubuntu:latest \
  jupyter notebook --ip=0.0.0.0 --port=8888 --allow-root --no-browser
```

### Python Script Example

Create `analyze_results.py`:

```python
#!/usr/bin/env python3
import abipy.abilab as abilab
import matplotlib.pyplot as plt

# Load ABINIT output files
gsr_file = "system_GSR.nc"  # Ground state results
gsr = abilab.abiopen(gsr_file)

# Plot band structure
gsr.ebands.plot()
plt.savefig("bandstructure.png")
plt.show()

# Print basic information
print(f"Number of electrons: {gsr.nelect}")
print(f"Total energy: {gsr.energy}")
print(f"Forces: {gsr.cart_forces}")

# Close file
gsr.close()
```

Run analysis:
```bash
docker run --rm \
  -v $(pwd):/workspace \
  ayyoubk/abinit-ubuntu:latest \
  python3 analyze_results.py
```

## 🔄 Workflow Examples

### Workflow 1: Complete DFT Study

```bash
#!/bin/bash
# complete_study.sh - Full DFT workflow

PROJECT_DIR=$(pwd)
IMAGE="ayyoubk/abinit-ubuntu:latest"

echo "Starting complete DFT study..."

# Step 1: Structure optimization
echo "1. Structure optimization..."
docker run --rm -v ${PROJECT_DIR}:/workspace ${IMAGE} \
  bash -c "cd /workspace && abinit < opt.files > opt.log 2>&1"

# Step 2: Electronic structure  
echo "2. Electronic structure..."
docker run --rm -v ${PROJECT_DIR}:/workspace ${IMAGE} \
  bash -c "cd /workspace && abinit < electronic.files > electronic.log 2>&1"

# Step 3: Optical properties
echo "3. Optical properties..."  
docker run --rm -v ${PROJECT_DIR}:/workspace ${IMAGE} \
  bash -c "cd /workspace && abinit < optic.files > optic.log 2>&1"

# Step 4: Analysis
echo "4. Analysis with Python..."
docker run --rm -v ${PROJECT_DIR}:/workspace ${IMAGE} \
  python3 analysis_script.py

echo "Study completed!"
```

### Workflow 2: High-Throughput Screening

```bash
#!/bin/bash
# screening.sh - Screen multiple materials

MATERIALS_LIST="materials_list.txt"
IMAGE="ayyoubk/abinit-ubuntu:latest"

while read material; do
  echo "Processing $material..."
  
  # Create working directory
  mkdir -p results/$material
  cp templates/* results/$material/
  
  # Customize input for this material
  sed -i "s/MATERIAL_NAME/$material/g" results/$material/input.in
  
  # Run calculation
  docker run --rm \
    -v $(pwd)/results/$material:/workspace \
    ${IMAGE} \
    abinit < input.files > output.log 2>&1
    
  # Extract key results
  docker run --rm \
    -v $(pwd)/results/$material:/workspace \
    ${IMAGE} \
    python3 extract_results.py >> summary.csv
    
done < $MATERIALS_LIST
```

## 📊 Post-Processing & Visualization

### Using Built-in Tools

```bash
# Generate 3D plots with cut3d
docker run --rm -v $(pwd):/workspace ayyoubk/abinit-ubuntu:latest \
  cut3d < cut3d.input

# Analyze DOS and band structure
docker run --rm -v $(pwd):/workspace ayyoubk/abinit-ubuntu:latest \
  python3 -c "
import abipy.abilab as abilab
gsr = abilab.abiopen('system_GSR.nc')
gsr.ebands.plot_dos()
gsr.close()
"
```

### Custom Analysis Scripts

Create `custom_analysis.py`:

```python
#!/usr/bin/env python3
"""Custom analysis script for ABINIT results"""

import numpy as np
import matplotlib.pyplot as plt
from abipy import abilab
import argparse

def analyze_convergence(log_file):
    """Analyze SCF convergence from log file"""
    energies = []
    with open(log_file, 'r') as f:
        for line in f:
            if 'Total energy' in line:
                energy = float(line.split()[-2])
                energies.append(energy)
    
    plt.figure(figsize=(10, 6))
    plt.plot(energies, 'b.-')
    plt.xlabel('SCF Iteration')
    plt.ylabel('Total Energy (Ha)')
    plt.title('SCF Convergence')
    plt.grid(True)
    plt.savefig('convergence.png', dpi=300)
    return energies

def analyze_structure(gsr_file):
    """Analyze structural properties"""
    with abilab.abiopen(gsr_file) as gsr:
        print(f"Final lattice parameters: {gsr.structure.lattice.abc}")
        print(f"Volume: {gsr.structure.volume:.3f} Angstrom^3")
        print(f"Pressure: {gsr.pressure:.3f} GPa")
        return gsr.structure

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description='Analyze ABINIT results')
    parser.add_argument('--log', help='Log file path')
    parser.add_argument('--gsr', help='GSR file path')
    
    args = parser.parse_args()
    
    if args.log:
        energies = analyze_convergence(args.log)
        print(f"Final energy: {energies[-1]:.6f} Ha")
    
    if args.gsr:
        structure = analyze_structure(args.gsr)
```

## 🚨 Troubleshooting

### Common Issues and Solutions

#### Issue: "Permission denied" when accessing files

**Solution:**
```bash
# Fix file permissions
chmod -R 755 your_project_directory/

# Or run with specific user ID
docker run --rm \
  --user $(id -u):$(id -g) \
  -v $(pwd):/workspace \
  ayyoubk/abinit-ubuntu:latest \
  abinit < input.files
```

#### Issue: MPI not working properly

**Solution:**
```bash
# Set MPI environment variables
docker run --rm \
  -e OMPI_MCA_btl_vader_single_copy_mechanism=none \
  -v $(pwd):/workspace \
  ayyoubk/abinit-ubuntu:latest \
  mpirun -np 4 abinit < input.files
```

#### Issue: Out of memory errors

**Solution:**
```bash
# Increase memory limit and reduce k-point sampling
docker run --rm \
  --memory="8g" \
  -v $(pwd):/workspace \
  ayyoubk/abinit-ubuntu:latest \
  abinit < input.files

# Or modify input file to use less memory:
# - Reduce ecut (plane wave cutoff)
# - Reduce k-point sampling  
# - Use smaller supercells
```

#### Issue: Calculation too slow

**Solutions:**
```bash
# Use more CPU cores
docker run --rm \
  --cpus="8" \
  -v $(pwd):/workspace \
  ayyoubk/abinit-ubuntu:latest \
  mpirun -np 8 abinit < input.files

# Optimize ABINIT parameters:
# - Use paral_kgb for k-point parallelization
# - Optimize nband and nbdblock
# - Use appropriate iscf algorithm
```

### Debugging Techniques

```bash
# Check container resources
docker stats --no-stream

# Monitor calculation progress
docker run --rm -v $(pwd):/workspace ayyoubk/abinit-ubuntu:latest \
  bash -c "abinit < input.files > output.log 2>&1 & tail -f output.log"

# Interactive debugging
docker run -it -v $(pwd):/workspace ayyoubk/abinit-ubuntu:latest bash
# Then inside container:
gdb abinit
```

## 📚 Learning Resources

### ABINIT Documentation
- **Official Tutorial**: https://docs.abinit.org/tutorial/
- **User Guide**: https://docs.abinit.org/guide/
- **Input Variables**: https://docs.abinit.org/variables/

### AbiPy Documentation  
- **AbiPy Tutorials**: https://github.com/abinit/abipy
- **Gallery of Examples**: https://abipy.readthedocs.io/

### Materials Science Background
- **DFT Textbooks**: Recommended reading list
- **Crystallography**: Structure databases and tools
- **Electronic Structure**: Theory and computational methods

## 🎯 Best Practices

### Input File Organization
```bash
# Use descriptive naming
silicon_scf.in          # Self-consistent calculation
silicon_bands.in        # Band structure  
silicon_phonons.in      # Phonon calculation

# Keep backups
cp important_calc.in important_calc.in.backup
```

### Resource Management
```bash
# Start with small calculations for testing
# Use convergence studies for critical parameters
# Monitor disk space and memory usage
# Clean up temporary files regularly
```

### Reproducibility
```bash
# Version control your input files
git init
git add *.in *.files
git commit -m "Initial calculation setup"

# Document your workflow
echo "ABINIT calculation for silicon" > README.txt
echo "Date: $(date)" >> README.txt
echo "Container: ayyoubk/abinit-ubuntu:latest" >> README.txt
```

## 💡 Advanced Tips

### Performance Optimization
```bash
# Use tmpfs for temporary files (faster I/O)
docker run --rm \
  --tmpfs /tmp:rw,noexec,nosuid,size=2g \
  -v $(pwd):/workspace \
  ayyoubk/abinit-ubuntu:latest \
  abinit < input.files
```

### Custom Container Extensions
```dockerfile
# Extend the base image with your tools
FROM ayyoubk/abinit-ubuntu:latest

# Install additional packages
RUN apt-get update && apt-get install -y \
    vim \
    htop \
    custom-analysis-tool

# Add your scripts
COPY my_analysis_scripts/ /opt/analysis/
```

## 📞 Getting Help

### Community Resources
- **ABINIT Forum**: https://forum.abinit.org/
- **GitHub Issues**: [Repository Issues](https://github.com/ayyoubk/dockerize-abinit/issues)
- **Stack Overflow**: Tag your questions with `abinit` and `docker`

### Reporting Issues
When reporting issues, please include:
- Input files (if possible)
- Complete error messages  
- Container version: `docker image ls ayyoubk/abinit-ubuntu`
- System information: `docker system info`

---

*Last Updated: July 2025*  
*Author: Ayyoub Al keyyam, Software Engineer*

**Happy Computing! 🧪⚛️**
