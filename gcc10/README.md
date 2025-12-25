# GCC 10.5.0 for Mac OS X Tiger (PowerPC)

## Overview
- **Version**: GCC 10.5.0
- **Target**: powerpc-apple-darwin9
- **Source**: MacPorts bootstrap
- **Size**: ~86MB compressed

## Features
- Full C17/C2x support
- Full C++17/C++20 support
- Latest optimizations
- Link-time optimization (LTO)

## Known Issue
Requires libiconv 7.0+. Stock Tiger has libiconv 5.0.

**Fix**: Install updated libiconv from MacPorts or create symlink.

## Installation
```bash
cd /opt/local/libexec
sudo tar xzf gcc10-10.5.0-tiger-ppc.tar.gz
```

## After Installation
- `/opt/local/libexec/gcc10-bootstrap/bin/gcc` → C compiler
- `/opt/local/libexec/gcc10-bootstrap/bin/g++` → C++ compiler

## Wrapper Script
Create `/usr/local/bin/gcc-10`:
```bash
#!/bin/bash
export DYLD_LIBRARY_PATH=/opt/local/lib:$DYLD_LIBRARY_PATH
exec /opt/local/libexec/gcc10-bootstrap/bin/gcc "$@"
```

## Usage
```bash
gcc-10 -mcpu=7450 -maltivec -O3 -std=c17 -o program program.c
g++-10 -mcpu=7450 -maltivec -O3 -std=c++20 -o program program.cpp
```

## Why GCC 10?
- C++20 coroutines, concepts, ranges
- Better diagnostics
- Improved vectorization for AltiVec
