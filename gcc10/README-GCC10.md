# GCC 10.5.0 for PowerPC Mac OS X Leopard

**C++20 on PowerPC** - Because vintage doesn't mean outdated.

## What's Included

- GCC 10.5.0 (gcc, g++, cpp)
- Full C++20 support (concepts, ranges, coroutines, etc.)
- libstdc++ with C++20 library features
- libatomic, libitm, libgomp
- Built for powerpc-apple-darwin9 (Leopard 10.5)

## Installation

```bash
# Download
curl -LO https://github.com/Scottcjn/ppc-compilers/releases/download/gcc-10.5.0/gcc-10.5.0-leopard-ppc.tar.gz

# Extract to /usr/local
cd /usr/local
sudo tar xzf ~/gcc-10.5.0-leopard-ppc.tar.gz

# Add to PATH
export PATH="/usr/local/gcc-10/bin:$PATH"

# Verify
gcc --version
# gcc (GCC) 10.5.0
```

## Usage

```bash
# Compile C++20 code
g++ -std=c++20 myprogram.cpp -o myprogram

# With optimizations
g++ -std=c++20 -O2 -march=G5 myprogram.cpp -o myprogram
```

## C++20 Features Tested

| Feature | Status |
|---------|--------|
| Concepts | ✓ Working |
| Designated initializers | ✓ Working |
| Init-statements in if/switch | ✓ Working |
| constexpr improvements | ✓ Working |
| Three-way comparison (<=>)  | ✓ Working |
| Modules | Partial |
| Ranges | ✓ Working |
| Coroutines | ✓ Working |

## Build Info

Built on Mac OS X 10.5.8 Leopard (PowerPC G5 Dual 2.0GHz)

```
Bootstrap compiler: GCC 7.5.0
Build time: ~2 hours
Configure: --prefix=/usr/local/gcc-10 --enable-languages=c,c++ --disable-multilib
```

## Requirements

- Mac OS X 10.5 Leopard (PowerPC)
- ~200MB disk space
- Works on G4 and G5

## Why GCC 10?

- Last GCC version with good PowerPC support
- Full C++20 implementation
- Modern optimizations
- Better error messages than older GCC

## Troubleshooting

### "dyld: Library not loaded"
```bash
export DYLD_LIBRARY_PATH=/usr/local/gcc-10/lib:$DYLD_LIBRARY_PATH
```

### Linking errors
```bash
# Add library path
g++ -L/usr/local/gcc-10/lib -Wl,-rpath,/usr/local/gcc-10/lib myprogram.cpp
```

## License

GCC is free software under GPLv3.

---

*Built Christmas 2025 by Elyan Labs*
*The glowies cloned this 600 times but didn't star it*
