# 📋 System Administrator Guide

> **Comprehensive deployment and management guide for HPC administrators**

## 🎯 Overview

This guide provides system administrators with everything needed to deploy, manage, and maintain ABINIT Docker containers on HPC clusters. The guide covers installation, configuration, security considerations, and operational best practices.

## 📋 Prerequisites

### System Requirements
- **Docker Engine**: Version 20.10+ recommended
- **Operating System**: Linux-based (Ubuntu 20.04+ preferred)
- **Architecture**: x86_64
- **Memory**: Minimum 4GB RAM, 16GB+ recommended for large calculations
- **Storage**: 10GB+ free space for images and workspaces
- **Network**: Internet access for initial image pull

### User Permissions
```bash
# Add users to docker group (optional, for non-root access)
sudo usermod -aG docker $USER

# Verify Docker installation
docker --version
docker info
```

## 🚀 Installation & Deployment

### Step 1: Pull the ABINIT Image

```bash
# Pull the latest production-ready image
docker pull ayyoubk/abinit-ubuntu:latest

# Verify the image
docker images | grep abinit-ubuntu
```

**Expected Output:**
```
REPOSITORY              TAG       IMAGE ID       CREATED        SIZE
ayyoubk/abinit-ubuntu  latest    e87dfb718704   2 days ago     1.48GB
```

### Step 2: Test Installation

```bash
# Test ABINIT version
docker run --rm ayyoubk/abinit-ubuntu:latest abinit --version

# Test MPI functionality
docker run --rm ayyoubk/abinit-ubuntu:latest mpirun --version

# Test Python scientific stack
docker run --rm ayyoubk/abinit-ubuntu:latest python3 -c "import numpy, scipy, matplotlib; print('Scientific Python stack working')"
```

### Step 3: Create Shared Directories

```bash
# Create shared directories for user data
sudo mkdir -p /shared/abinit/{workspaces,data,results}
sudo chmod 755 /shared/abinit/
sudo chown root:users /shared/abinit/{workspaces,data,results}
sudo chmod 775 /shared/abinit/{workspaces,data,results}
```

## 🔧 Container Configuration

### Basic Container Run Command

```bash
docker run -it \
  --name abinit-session \
  -v /shared/abinit/workspaces:/workspace \
  -v /shared/abinit/data:/data \
  --cpus="4" \
  --memory="8g" \
  ayyoubk/abinit-ubuntu:latest
```

### Advanced Configuration Options

#### Resource Limits
```bash
# Set CPU and memory limits
docker run -it \
  --cpus="8" \                    # Limit to 8 CPU cores
  --memory="16g" \                # Limit to 16GB RAM
  --memory-swap="20g" \           # Set swap limit
  --oom-kill-disable \            # Don't kill on OOM
  ayyoubk/abinit-ubuntu:latest
```

#### Volume Mounts
```bash
# Mount multiple directories
docker run -it \
  -v /shared/abinit/workspaces:/workspace \      # User workspaces
  -v /shared/abinit/data:/data:ro \              # Read-only data
  -v /shared/abinit/results:/results \           # Results output
  -v /etc/passwd:/etc/passwd:ro \                # User information
  -v /etc/group:/etc/group:ro \                  # Group information
  ayyoubk/abinit-ubuntu:latest
```

#### Network Configuration
```bash
# For cluster networking
docker run -it \
  --network=host \                # Use host networking
  --hostname=abinit-node \        # Set hostname
  ayyoubk/abinit-ubuntu:latest
```

## 🎛️ HPC Integration

### SLURM Integration

Create a SLURM job script template:

```bash
#!/bin/bash
#SBATCH --job-name=abinit-job
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --time=24:00:00
#SBATCH --partition=compute
#SBATCH --output=abinit_%j.out
#SBATCH --error=abinit_%j.err

# Load Docker module (if using modules)
module load docker

# Set working directory
cd $SLURM_SUBMIT_DIR

# Run ABINIT calculation
docker run --rm \
  --cpus="$SLURM_NTASKS_PER_NODE" \
  --memory="32g" \
  -v $(pwd):/workspace \
  -v /shared/data:/data:ro \
  ayyoubk/abinit-ubuntu:latest \
  mpirun -np $SLURM_NTASKS_PER_NODE abinit < input.files > output.log 2>&1
```

### PBS/Torque Integration

```bash
#!/bin/bash
#PBS -N abinit-job
#PBS -l nodes=1:ppn=8
#PBS -l walltime=24:00:00
#PBS -q batch
#PBS -o abinit.out
#PBS -e abinit.err

cd $PBS_O_WORKDIR

docker run --rm \
  --cpus="8" \
  --memory="32g" \
  -v $(pwd):/workspace \
  ayyoubk/abinit-ubuntu:latest \
  mpirun -np 8 abinit < input.files > output.log 2>&1
```

## 🔒 Security Considerations

### User Isolation

```bash
# Run as specific user (recommended)
docker run -it \
  --user $(id -u):$(id -g) \      # Run as current user
  -v /etc/passwd:/etc/passwd:ro \ # Mount user database
  -v /etc/group:/etc/group:ro \   # Mount group database
  ayyoubk/abinit-ubuntu:latest
```

### Resource Security

```bash
# Limit resources and capabilities
docker run -it \
  --security-opt=no-new-privileges \     # Prevent privilege escalation
  --cap-drop=ALL \                       # Drop all capabilities
  --cap-add=DAC_OVERRIDE \              # Allow file access
  --read-only \                          # Read-only filesystem
  --tmpfs /tmp:rw,noexec,nosuid,size=1g \ # Temporary filesystem
  ayyoubk/abinit-ubuntu:latest
```

### Network Security

```bash
# Restrict network access
docker run -it \
  --network=none \                # No network access
  # OR
  --network=internal_cluster \    # Custom internal network
  ayyoubk/abinit-ubuntu:latest
```

## 📊 Monitoring & Maintenance

### Container Health Checks

```bash
# Check running containers
docker ps

# Monitor container resource usage
docker stats

# View container logs
docker logs <container_id>

# Inspect container configuration
docker inspect <container_id>
```

### System Monitoring Script

```bash
#!/bin/bash
# monitor_abinit.sh - Monitor ABINIT containers

echo "=== ABINIT Container Status ==="
docker ps --filter ancestor=ayyoubk/abinit-ubuntu:latest

echo -e "\n=== Resource Usage ==="
docker stats --no-stream --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.MemPerc}}"

echo -e "\n=== Disk Usage ==="
docker system df

echo -e "\n=== Image Information ==="
docker images ayyoubk/abinit-ubuntu:latest
```

### Log Management

```bash
# Configure Docker logging
cat > /etc/docker/daemon.json << EOF
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
EOF

# Restart Docker service
sudo systemctl restart docker
```

## 🔄 Updates & Maintenance

### Image Updates

```bash
# Pull latest image
docker pull ayyoubk/abinit-ubuntu:latest

# Check for updates
docker image ls ayyoubk/abinit-ubuntu:latest

# Remove old images (optional)
docker image prune -f
```

### Scheduled Maintenance Script

```bash
#!/bin/bash
# weekly_maintenance.sh

echo "Starting weekly ABINIT maintenance..."

# Pull latest image
docker pull ayyoubk/abinit-ubuntu:latest

# Clean up unused containers and images
docker container prune -f
docker image prune -f

# Clean up volumes (be careful!)
docker volume prune -f

# Update system packages
apt-get update && apt-get upgrade -y

echo "Maintenance completed."
```

## 🚨 Troubleshooting

### Common Issues

#### Issue: Image Pull Fails
```bash
# Solution: Check network connectivity
ping hub.docker.com
docker login  # If using private registry
```

#### Issue: Permission Denied
```bash
# Solution: Fix volume permissions
sudo chown -R $USER:$USER /shared/abinit/
chmod -R 755 /shared/abinit/
```

#### Issue: Out of Memory
```bash
# Solution: Increase memory limits
docker run --memory="16g" --memory-swap="20g" ...
```

#### Issue: MPI Not Working
```bash
# Solution: Check container networking
docker run --network=host ...
# OR use proper MPI configuration
export OMPI_MCA_btl_vader_single_copy_mechanism=none
```

### Diagnostic Commands

```bash
# System diagnostics
docker system info
docker system df
docker system events

# Container diagnostics
docker exec -it <container> /bin/bash
docker logs --tail 50 <container>
docker inspect <container>
```



## 📚 Additional Resources

- **ABINIT Official Documentation**: https://docs.abinit.org/
- **Docker Documentation**: https://docs.docker.com/
- **Container Security Best Practices**: https://docs.docker.com/engine/security/
- **HPC Container Guidelines**: [Your organization's container policy]

---

## 📞 Support

For technical support and questions:
- **GitHub Issues**: [Repository Issues](https://github.com/ayyoubk/dockerize-abinit/issues)
- **Documentation**: Complete guides available in repository docs/

---

*Last Updated: July 2025*  
*Author: Ayyoub Al keyyam, Software Engineer*
