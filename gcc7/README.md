# GCC 7.5.0 for Mac OS X Tiger (PowerPC)

## Overview
- **Version**: GCC 7.5.0
- **Target**: powerpc-apple-darwin8 (Tiger/Leopard)
- **Source**: Tigerbrew
- **Size**: ~59MB compressed

## Features
- Full C11/C17 support
- Full C++14/C++17 support
- Modern optimizations for G4/G5
- AltiVec/SIMD support

## Installation
```bash
cd /usr/local
sudo tar xzf gcc7-7.5.0-tiger-ppc.tar.gz
```

## After Installation
- `/usr/local/bin/gcc-7` → C compiler
- `/usr/local/bin/g++-7` → C++ compiler
- `/usr/local/Cellar/gcc7/7.5.0/` → Full installation

## Usage
```bash
# C compilation with AltiVec
gcc-7 -mcpu=7450 -maltivec -O2 -o program program.c

# C++ compilation
g++-7 -mcpu=7450 -maltivec -O2 -std=c++17 -o program program.cpp
```

## Why This Matters
Stock Tiger has GCC 4.0.1 (2005). GCC 7.5.0 enables:
- Building modern software (QuickJS, etc.)
- C++17 features
- Better optimization
- CVE patches for legacy code
