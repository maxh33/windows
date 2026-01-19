# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This project creates a Docker container that runs Windows virtual machines using QEMU/KVM. It provides automatic Windows installation, web-based VNC access, and RDP connectivity. The container supports multiple Windows versions from Windows XP to Windows 11 and Server editions.

## Common Commands

### Docker Operations
```bash
# Build the Docker image
docker build -t dockurr/windows .

# Run with Docker Compose (recommended)
docker-compose up -d

# Run with Docker CLI
docker run -it --rm --name windows -p 8006:8006 --device=/dev/kvm --device=/dev/net/tun --cap-add NET_ADMIN -v "${PWD:-.}/windows:/storage" --stop-timeout 120 dockurr/windows

# View logs
docker logs windows

# Stop gracefully (important for data integrity)
docker-compose down
```

### Development Commands
```bash
# Check shell scripts with shellcheck
shellcheck src/*.sh

# Test the build
docker build --no-cache -t test .
```

## Architecture

### Core Components

- **Entry Point** (`src/entry.sh`): Main orchestration script that loads and executes all components in sequence
- **Version Definitions** (`src/define.sh`): Handles Windows version parsing and URL mapping for automatic downloads
- **Installation Engine** (`src/install.sh`): Manages automatic Windows installation using unattended installation files
- **Network Configuration** (`src/network.sh`): Sets up networking for the VM including TUN/TAP devices
- **Storage Management** (`src/disk.sh`): Handles disk creation and management for Windows VMs
- **Display Configuration** (`src/display.sh`): Configures VNC/SPICE for remote access

### Installation Assets

The `assets/` directory contains Windows unattended installation XML files for each supported version:
- Format: `win{version}x{arch}.xml` (e.g., `win11x64.xml`)
- Contains automated installation sequences, drivers, and configuration
- Handles different editions (Pro, Enterprise, LTSC, IoT)

### Container Structure

- **Base Image**: Uses `qemux/qemu:7.12` for QEMU virtualization
- **Volume Mounts**: 
  - `/storage` - Persistent Windows VM data
  - `/data` - Shared folder between host and Windows
  - `/boot.iso` - Custom ISO mounting point
- **Required Devices**: `/dev/kvm`, `/dev/net/tun` for hardware virtualization and networking
- **Ports**: 8006 (web VNC), 3389 (RDP), 5900 (VNC)

### Key Environment Variables

- `VERSION`: Windows version to install (default: "11")
- `RAM_SIZE`: Memory allocation (default: "4G") 
- `CPU_CORES`: CPU cores (default: "2")
- `DISK_SIZE`: Virtual disk size (default: "64G")
- `MANUAL`: Skip automatic installation ("Y" to enable)
- `USERNAME`/`PASSWORD`: Windows user credentials

### Deployment Options

- **Docker Compose**: Primary development and deployment method
- **Kubernetes**: Production deployment with PVC storage
- **GitHub Codespaces**: Cloud development environment

## Development Workflow

1. **Shell Script Changes**: All core logic is in bash scripts under `src/`
2. **Windows Versions**: Add new versions by updating `src/define.sh` and creating corresponding XML files in `assets/`
3. **Testing**: Use the GitHub Actions workflow (`.github/workflows/build.yml`) for automated testing
4. **Image Building**: Multi-arch builds support both AMD64 and ARM64 architectures

## Important Notes

- The container requires KVM support for hardware virtualization
- Windows installation is fully automated using unattended installation files
- The system gracefully handles VM shutdown to prevent data corruption
- File sharing between host and Windows is handled via SMB/CIFS through `src/samba.sh`
- VirtIO drivers are automatically installed for better performance