# GCC 7.5.0 for Mac OS X Tiger (PowerPC)

## Packages

### Full Installation (~59MB each)
- `gcc7-7.5.0-tiger-ppc.tar.gz` - Full GCC 7 for Tiger
- `gcc7-7.5.0-tiger-ppc32.tar.gz` - 32-bit variant

### Standalone C++ Package (~15MB) - NEW!
- `gcc7-cpp-ppc.tar.gz` - Portable C++11/14/17 with wrapper scripts

## Full Installation

```bash
cd /usr/local
sudo tar xzf gcc7-7.5.0-tiger-ppc.tar.gz
```

After installation:
- `/usr/local/bin/gcc-7` - C compiler
- `/usr/local/bin/g++-7` - C++ compiler
- `/usr/local/Cellar/gcc7/7.5.0/` - Full installation

## Standalone C++ Package

Self-contained package with wrapper scripts that handle the libstdc++ path.

```bash
# Extract anywhere
tar xzf gcc7-cpp-ppc.tar.gz
cd gcc7-cpp

# Compile with C++11
./g++11 mycode.cpp -o myprogram

# Compile with C++14
./g++14 mycode.cpp -o myprogram

# Compile with C++17
./g++17 mycode.cpp -o myprogram

# Run compiled binary (sets library path)
./run-cpp ./myprogram
```

**Why wrappers?** Tiger ships with ancient libstdc++ (GCC 4.0). The wrappers
set `DYLD_LIBRARY_PATH` so your C++11/14/17 code uses the GCC 7 runtime.

## Tested C++ Features

### C++11
- Lambdas, auto, range-based for
- std::shared_ptr, std::unique_ptr
- std::thread, std::mutex
- Move semantics

### C++14
- Generic lambdas
- std::make_unique
- Digit separators (1'000'000)

### C++17
- std::optional, std::variant
- std::string_view
- Structured bindings
- if/switch with initializer

## General Usage

```bash
# C compilation with AltiVec
gcc-7 -mcpu=7450 -maltivec -O2 -o program program.c

# C++ compilation (full install)
g++-7 -mcpu=7450 -maltivec -O2 -std=c++17 -o program program.cpp

# C++ with runtime path (needed if not using wrappers)
DYLD_LIBRARY_PATH=/usr/local/Cellar/gcc7/7.5.0/lib/gcc/7 ./myprogram
```

## Why This Matters

Stock Tiger has GCC 4.0.1 (2005). GCC 7.5.0 enables:
- Building modern software (QuickJS, Node.js, etc.)
- C++17 features
- Better optimization
- CVE patches for legacy code

## Architecture
- **Target**: powerpc-apple-darwin8 (Tiger), darwin9 (Leopard)
- **Source**: Tigerbrew

Built with love for vintage Macs - Christmas 2025
