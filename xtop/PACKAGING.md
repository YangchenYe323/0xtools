# XTOP Debian Packaging Guide

This directory contains Debian packaging specifications for distributing xtop. The package installs xtop in an isolated Python virtual environment using the system Python.

## Package Structure

The Debian package installs xtop in the following structure:
- `/opt/xtop/` - Application files and isolated venv
- `/opt/xtop/venv/` - Python virtual environment with dependencies
- `/opt/xtop/core/` - Core application modules
- `/opt/xtop/tui/` - TUI components
- `/opt/xtop/sql/` - SQL query templates
- `/opt/xtop/xtop` - Main executable script
- `/usr/bin/xtop` - Wrapper script that invokes the venv Python

## Dependencies

Runtime dependencies (installed in venv):
- textual - Terminal UI framework
- duckdb - In-memory analytical database

System dependencies:
- python3 (>= 3.8)
- python3-pip
- python3-venv

## Debian Packaging Files

The `debian/` directory contains:

### `debian/control`
Package metadata, dependencies, and build requirements. Specifies `debhelper-compat (= 13)` for modern compat level handling.

### `debian/rules`
Makefile for building the package. Contains:
- `override_dh_auto_build`: Empty (nothing to compile for Python)
- `override_dh_auto_install`: Copies application files, creates venv, installs Python dependencies, and creates wrapper script
- `override_dh_install`: Skipped (we handle everything in override_dh_auto_install)
- `override_dh_auto_clean`: Cleanup logic

**Important**:
- Must be executable (`chmod +x debian/rules`)
- Venv creation happens in `override_dh_auto_install` (after `dh_prep` cleans staging directory)
- We override `dh_install` to prevent conflicts with manual installation

### `debian/changelog`
Version history in Debian format. First line format:
```
xtop (1.0.0-1) unstable; urgency=medium
```

### `debian/install` (Optional, not used)
This file exists but is ignored because we override `dh_install` in `debian/rules`. All file installation is handled manually in `override_dh_auto_install`.

### `debian/source/format`
Specifies source package format: `3.0 (native)`

**Note**: No `debian/compat` file is needed. The compat level is specified via the `debhelper-compat (= 13)` build dependency in `debian/control`. Using both methods causes build failures.

## Building Debian Package

### Prerequisites
```bash
# Install Debian packaging tools
sudo apt-get install debhelper dh-python python3 python3-pip python3-venv build-essential
```

### Build Process

```bash
# From the xtop source directory
cd /path/to/0xtools/xtop

# Build the package
dpkg-buildpackage -us -uc -b

# Or using debuild (if devscripts is installed)
debuild -us -uc -b
```

The resulting package will be in the parent directory: `../xtop_1.0.0-1_all.deb`

### Installing the Debian Package
```bash
sudo dpkg -i xtop_1.0.0-1*.deb

# If dependencies are missing, install them
sudo apt-get install -f
```
