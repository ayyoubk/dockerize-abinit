# ABINIT Docker Images for HPC Clusters

This repository contains two Docker solutions for running ABINIT on HPC clusters:

1. **`abinit-ubuntu/`** - Uses Ubuntu's official ABINIT package (quick setup)
2. **`abinit-source/`** - Builds ABINIT 10.4.5 from source (latest features, optimized)

## System Requirements
- Target cluster: Ubuntu-based HPC systems
- CPU: Intel Xeon E5-2698 v4 (AVX2 support)
- Architecture: x86_64

## Quick Start
Choose the appropriate directory and follow the instructions in each folder's README.

For most users, start with `abinit-ubuntu/` for simplicity, then move to `abinit-source/` if you need the latest features or custom optimizations.
