# Contributing to ppc-compilers

Thank you for your interest in contributing to ppc-compilers! This project provides modern compilers (GCC 7/10, Python 3.7, Perl 5.34) for PowerPC Mac OS X Tiger and Leopard systems.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [Development Environment](#development-environment)
- [How to Contribute](#how-to-contribute)
- [Build System](#build-system)
- [Testing](#testing)
- [Style Guidelines](#style-guidelines)
- [Submitting Changes](#submitting-changes)
- [Community](#community)

## Code of Conduct

This project and everyone participating in it is governed by our commitment to:

- Be respectful and inclusive
- Welcome newcomers and help them learn
- Focus on constructive feedback
- Respect vintage hardware limitations

## Getting Started

### Prerequisites

To contribute effectively, you'll need:

**For Testing (Required):**
- Access to PowerPC Mac hardware (G4 or G5) OR
- QEMU setup for PowerPC emulation
- Mac OS X Tiger (10.4) or Leopard (10.5)

**For Development:**
- Modern macOS or Linux system
- Git
- Text editor with shell script support
- Basic understanding of compiler toolchains

### Repository Structure

```
ppc-compilers/
├── gcc7/           # GCC 7.5.0 build scripts and configs
├── gcc10/          # GCC 10.5.0 build scripts and configs
├── python3/          # Python 3.7.17 build scripts
├── perl5/          # Perl 5.34.3 build scripts
├── tools/            # Additional tools (tar, etc.)
├── scripts/          # Build automation scripts
├── patches/          # Platform-specific patches
└── docs/             # Additional documentation
```

## Development Environment

### Setting Up QEMU for PowerPC (No Hardware)

If you don't have physical PowerPC hardware:

```bash
# Install QEMU
brew install qemu  # macOS
sudo apt-get install qemu-system-ppc  # Ubuntu/Debian

# Download Mac OS X Tiger/Leopard ISO
# Create disk image
qemu-img create -f qcow2 tiger_ppc.qcow2 20G

# Run QEMU with PowerPC emulation
qemu-system-ppc -M mac99 -m 1024 \
  -hda tiger_ppc.qcow2 \
  -cdrom tiger.iso \
  -boot d
```

### Using Real Hardware

**Recommended Test Machines:**

| Machine | CPU | RAM | OS | Best For |
|---------|-----|-----|-----|----------|
| Power Mac G5 | Dual 2.0 GHz | 8GB | Leopard | GCC 10, full testing |
| Power Mac G4 | Dual 1.25 GHz | 2GB | Tiger | GCC 7, compatibility |
| PowerBook G4 | 1.67 GHz | 2GB | Tiger | Portability testing |
| iMac G5 | 1.8 GHz | 2GB | Tiger | Integration testing |

## How to Contribute

### Types of Contributions

We welcome:

1. **New Compiler Ports**
   - Additional GCC versions
   - Other languages (Rust, Go, etc.)
   - Cross-compiler variants

2. **Build Script Improvements**
   - Automation enhancements
   - Error handling
   - Progress reporting

3. **Documentation**
   - Setup guides
   - Troubleshooting
   - Performance tips

4. **Testing & Validation**
   - Hardware compatibility reports
   - Benchmark results
   - Bug reports

5. **Patches & Fixes**
   - Platform-specific fixes
   - Security updates
   - Performance optimizations

### Finding Issues

Check our issue tracker for:
- [Good first issues](https://github.com/Scottcjn/ppc-compilers/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22)
- [Help wanted](https://github.com/Scottcjn/ppc-compilers/issues?q=is%3Aissue+is%3Aopen+label%3A%22help+wanted%22)
- [Bug reports](https://github.com/Scottcjn/ppc-compilers/issues?q=is%3Aissue+is%3Aopen+label%3Abug)

## Build System

### GCC Build Process

```bash
# Standard GCC build workflow
cd gcc-7.5.0
./contrib/download_prerequisites
mkdir build && cd build

# Configure for PowerPC
../configure --prefix=/usr/local/gcc-7.5.0 \
  --enable-languages=c,c++ \
  --disable-multilib \
  --with-cpu=G4  # or G5 for ppc64

# Build (takes 4-8 hours)
make -j2 bootstrap

# Install
sudo make install
```

### Build Script Standards

All build scripts must:

1. **Include Error Handling:**
```bash
#!/bin/bash
set -e  # Exit on error
set -u  # Exit on undefined variable
set -o pipefail  # Exit on pipe failure
```

2. **Check Prerequisites:**
```bash
# Check for required tools
for tool in gcc make g++; do
  if ! command -v $tool &> /dev/null; then
    echo "Error: $tool is required but not installed"
    exit 1
  fi
done

# Check disk space
available=$(df /usr/local | tail -1 | awk '{print $4}')
if [ $available -lt 5242880 ]; then  # 5GB in KB
  echo "Error: At least 5GB free space required"
  exit 1
fi
```

3. **Provide Progress Feedback:**
```bash
echo "[1/5] Downloading prerequisites..."
echo "[2/5] Configuring build..."
echo "[3/5] Building (this will take several hours)..."
```

## Testing

### Test Requirements

Before submitting:

1. **Test on Target Hardware:**
   - Minimum: One G4 (Tiger) and one G5 (Leopard)
   - Test both 32-bit and 64-bit where applicable

2. **Verify Functionality:**
```bash
# Test GCC
gcc --version
# Expected: gcc (GCC) 7.5.0 or 10.5.0

# Test C++17 features
echo 'int main() { auto x = [](auto a) { return a; }; }' | \
  gcc -std=c++17 -x c++ -

# Test Perl
perl -v
# Expected: perl 5, version 34

# Test Python
python3.7 --version
# Expected: Python 3.7.17
```

3. **Run Integration Tests:**
```bash
# Test building a real project
cd test-projects
./test-llvm-build.sh
./test-perl-modules.sh
```

### Performance Testing

When optimizing:

```bash
# Benchmark compilation speed
time gcc -O2 -c large-file.c

# Compare binary sizes
ls -la a.out

# Test runtime performance
./benchmark-suite
```

## Style Guidelines

### Shell Scripts

Follow these conventions:

```bash
#!/bin/bash
# Header with description
# Author and date

# Variables: UPPER_CASE for constants
readonly PREFIX="/usr/local"
readonly VERSION="7.5.0"

# Functions: lower_case with underscores
build_gcc() {
  local target=$1
  local jobs=$2
  
  # Use local variables
  local build_dir="build-${target}"
  
  # Indent with 2 spaces
  if [[ -d "$build_dir" ]]; then
    rm -rf "$build_dir"
  fi
  
  # Quote all variables
  mkdir -p "$build_dir"
}

# Main execution
main() {
  build_gcc "ppc" "2"
}

main "$@"
```

### Documentation

- Use Markdown for all docs
- Include code examples
- Add hardware compatibility notes
- Keep line length under 80 characters

### Commit Messages

Format:
```
<type>: <subject>

<body>

<footer>
```

Types:
- `feat`: New compiler or feature
- `fix`: Bug fix
- `docs`: Documentation only
- `build`: Build system changes
- `test`: Testing additions
- `perf`: Performance improvements

Example:
```
feat: add GCC 10.5.0 support for Leopard

- Configure with --disable-multilib for ppc64
- Add G5-specific optimizations
- Include bootstrap build instructions

Tested on: Power Mac G5 Dual 2.0GHz
```

## Submitting Changes

### Pull Request Process

1. **Fork the repository**
2. **Create a feature branch:**
   ```bash
   git checkout -b feat/add-gcc-11
   ```
3. **Make your changes**
4. **Test thoroughly on PowerPC hardware**
5. **Commit with clear messages**
6. **Push to your fork**
7. **Open a Pull Request**

### PR Requirements

- [ ] Tested on real PowerPC hardware
- [ ] All scripts have error handling
- [ ] Documentation updated
- [ ] Commit messages follow conventions
- [ ] No merge conflicts
- [ ] PR description explains changes

### PR Template

```markdown
## Description
Brief description of changes

## Type
- [ ] New compiler
- [ ] Build script improvement
- [ ] Documentation
- [ ] Bug fix
- [ ] Performance optimization

## Testing
- Hardware tested: (e.g., Power Mac G5 Dual 2.0GHz)
- OS version: (e.g., Leopard 10.5.8)
- Tests passed: (list)

## Checklist
- [ ] Error handling