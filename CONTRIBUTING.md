# Contributing to ppc-compilers

Thank you for your interest in contributing to ppc-compilers! This project provides modern compilers (GCC 7, GCC 10, Python 3.7, Perl 5.34) for PowerPC Mac OS X Tiger and Leopard systems.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [How Can I Contribute?](#how-can-i-contribute)
- [Development Setup](#development-setup)
- [Style Guidelines](#style-guidelines)
- [Submitting Changes](#submitting-changes)
- [Community](#community)

## Code of Conduct

This project adheres to a code of conduct that expects all participants to:
- Be respectful and inclusive
- Welcome newcomers
- Focus on constructive feedback
- Respect differing viewpoints and experiences

## Getting Started

### Prerequisites

To contribute to this project, you'll need:

- **Hardware**: PowerPC Mac (G4 or G5) or QEMU emulation setup
- **OS**: Mac OS X Tiger (10.4) or Leopard (10.5)
- **Xcode**: Xcode 2.5 (Tiger) or Xcode 3.1 (Leopard)
- **Resources**: 2GB+ RAM, 5GB+ free disk space
- **Patience**: Bootstrap builds take 4-8 hours

### Repository Structure

```
ppc-compilers/
├── gcc7/           # GCC 7.5.0 build scripts and patches
├── gcc10/          # GCC 10.5.0 build scripts and patches
├── python3/        # Python 3.7.17 build scripts
├── perl5/          # Perl 5.34.3 build scripts
├── patches/        # Shared patches for all compilers
├── scripts/        # Build automation scripts
└── tests/          # Test suites for compiled binaries
```

## How Can I Contribute?

### Reporting Bugs

Before creating a bug report:

1. **Check existing issues** to avoid duplicates
2. **Test on real hardware** if possible (QEMU may behave differently)
3. **Gather system information**:
   ```bash
   # System profiler
   system_profiler SPHardwareDataType
   
   # OS version
   sw_vers
   
   # Available RAM
   top -l 1 | grep PhysMem
   ```

**Bug report template:**
```markdown
**Compiler**: GCC 7.5.0 / GCC 10.5.0 / Python 3.7 / Perl 5.34
**Host OS**: Mac OS X Tiger 10.4.11 / Leopard 10.5.8
**Hardware**: Power Mac G5 Dual 2.0GHz / PowerBook G4 1.67GHz
**Xcode Version**: 2.5 / 3.1

**Description**:
Clear description of the bug

**Steps to Reproduce**:
1. Step one
2. Step two
3. Step three

**Expected Behavior**:
What you expected to happen

**Actual Behavior**:
What actually happened

**Build Log**:
Attach relevant build output (last 50 lines minimum)
```

### Suggesting Enhancements

Enhancement suggestions are welcome! Please include:
- Clear use case for vintage PowerPC development
- Compatibility considerations for Tiger/Leopard
- Performance impact assessment

### Adding New Compiler Versions

When adding support for new compiler versions:

1. **Research compatibility** with PowerPC architecture
2. **Check dependencies** on system libraries
3. **Test bootstrap build** on both Tiger and Leopard
4. **Document any new requirements**

### Improving Documentation

Documentation improvements are always welcome:
- README clarifications
- Installation troubleshooting
- Performance optimization tips
- Compatibility matrices

## Development Setup

### Setting Up Build Environment

1. **Install Xcode**:
   - Tiger: Xcode 2.5 from ADC (requires free registration)
   - Leopard: Xcode 3.1 from ADC or install DVD

2. **Install Developer Tools**:
   ```bash
   # Verify compiler
   gcc --version
   # Should show: gcc (GCC) 4.0.1 (Tiger) or 4.2.1 (Leopard)
   ```

3. **Configure Environment**:
   ```bash
   # Add to ~/.bash_profile
   export PATH=/Developer/usr/bin:$PATH
   ```

### Using QEMU for Testing (No Hardware Required)

If you don't have PowerPC hardware:

```bash
# Install QEMU
brew install qemu

# Create Tiger disk image
qemu-img create -f qcow2 tiger.qcow2 20G

# Boot Tiger installation
qemu-system-ppc -M mac99 -m 1024 \
  -hda tiger.qcow2 \
  -cdra tiger-install.iso \
  -boot d
```

**Note**: QEMU emulation is slower than real hardware but sufficient for testing.

### Building GCC from Source

```bash
# Download GCC source
cd gcc-7.5.0
./contrib/download_prerequisites

# Create build directory
mkdir build && cd build

# Configure for PowerPC
../configure --prefix=/usr/local/gcc-7.5.0 \
  --enable-languages=c,c++ \
  --disable-multilib \
  --with-cpu=powerpc \
  --enable-threads=posix

# Build (this takes 4-8 hours)
make -j2 bootstrap

# Install
sudo make install
```

## Style Guidelines

### Shell Script Style

Build scripts should follow these conventions:

```bash
#!/bin/bash
set -euo pipefail

# Variables: UPPER_CASE for constants
readonly PREFIX="/usr/local"
readonly GCC_VERSION="7.5.0"

# Functions: lowercase with underscores
build_gcc() {
    local build_dir="$1"
    cd "$build_dir"
    # ...
}

# Error handling
die() {
    echo "ERROR: $*" >&2
    exit 1
}

# Main execution
main() {
    [[ $# -eq 0 ]] && die "Usage: $0 <build-dir>"
    build_gcc "$1"
}

main "$@"
```

### Patch Format

Patches should be generated with:

```bash
# Create patch from modified source
diff -u original.c modified.c > fix-description.patch

# Or using git
git diff > fix-description.patch
```

**Patch naming convention**:
- `gcc-7.5.0-tiger-fix.patch` - GCC 7.5.0 specific fix for Tiger
- `common-powerpc-atomic.patch` - Shared patch for all versions
- `python3.7-configure.patch` - Python-specific patch

### Documentation Style

- Use clear, concise language
- Include command examples
- Specify which OS version (Tiger/Leopard) for each instruction
- Note hardware requirements where relevant

## Submitting Changes

### Pull Request Process

1. **Fork the repository** on GitHub
2. **Clone your fork**:
   ```bash
   git clone https://github.com/YOUR_USERNAME/ppc-compilers.git
   cd ppc-compilers
   ```
3. **Create a branch**:
   ```bash
   git checkout -b feature/your-feature-name
   # or
   git checkout -b fix/bug-description
   ```
4. **Make your changes** following style guidelines
5. **Test your changes** on PowerPC hardware or QEMU
6. **Commit with clear messages**:
   ```bash
   git commit -m "feat: add GCC 11 support for Leopard

   - Adds build scripts for GCC 11.3.0
   - Tested on Power Mac G5 Dual 2.3GHz
   - Requires 4GB RAM for bootstrap"
   ```
7. **Push to your fork**:
   ```bash
   git push origin feature/your-feature-name
   ```
8. **Open a Pull Request** on GitHub

### Commit Message Format

Follow conventional commits:

- `feat:` - New compiler version or feature
- `fix:` - Bug fix
- `docs:` - Documentation changes
- `test:` - Test additions or improvements
- `chore:` - Build process or auxiliary tool changes

**Examples**:
```
feat: add Python 3.8 support for Leopard
fix: resolve Tiger SDK path detection
docs: update QEMU setup instructions
test: add C++17 feature detection test
```

### PR Checklist

Before submitting:

- [ ] Code follows style guidelines
- [ ] Changes tested on Tiger or Leopard (real hardware or QEMU)
- [ ] Documentation updated if needed
- [ ] Commit messages are clear and descriptive
- [ ] No breaking changes without clear justification
- [ ] Build scripts include error handling

### Testing Requirements

All changes must be tested:

**Minimum testing**:
```bash
# Test compiler builds successfully
./scripts/build-gcc7.sh

# Verify installation
/usr/local/gcc-7.5.0/bin/gcc --version

# Test basic functionality
echo 'int main() { return 0; }' | \
  /usr/local/gcc-7.5.0/bin/gcc -x c - -o /tmp/test && \
  /tmp/test && echo "PASS"
```

**For C++ features**:
```bash
# Test C++11/14/17 support
echo '#include <iostream>
int main() {
    auto x = [](auto a) { return a * 2; };
    std::cout << x(21) << std::endl;
    return 0;
}' | /usr/local/gcc-7.5.0/bin/g++ -std=c++17 -x c++ - -o /tmp