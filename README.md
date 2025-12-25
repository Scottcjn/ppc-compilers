# Modern Compilers for PowerPC Mac OS X Tiger & Leopard

**GCC 7, GCC 10, and Perl 5.34 - Prebuilt for PowerPC G4/G5!**

Run modern compilers on your vintage Mac. No cross-compiling needed - these are native PowerPC binaries.

## What's Included

| Compiler | Version | Target | Size |
|----------|---------|--------|------|
| **GCC 7.5.0** | Latest GCC 7 | ppc/ppc64 | ~200MB |
| **GCC 10.5.0** | Latest GCC 10 | ppc/ppc64 | ~250MB |
| **Perl 5.34.3** | Modern Perl 5 | ppc | ~50MB |

## Why You Need This

Mac OS X Tiger (10.4) and Leopard (10.5) ship with:
- **GCC 4.0** (Tiger) / **GCC 4.2** (Leopard) - Too old for modern code
- **Perl 5.8** - Missing modern Perl features

These compilers let you build:
- Modern C++11/14/17 code
- Software requiring GCC 7+
- Perl scripts using modern features
- Node.js, Python extensions, and more

## Installation

### GCC 7.5.0
```bash
cd gcc7
tar xzf gcc-7.5.0-ppc-tiger.tar.gz
sudo mv gcc-7.5.0 /usr/local/
export PATH=/usr/local/gcc-7.5.0/bin:$PATH
```

### GCC 10.5.0
```bash
cd gcc10
tar xzf gcc-10.5.0-ppc-leopard.tar.gz
sudo mv gcc-10.5.0 /usr/local/
export PATH=/usr/local/gcc-10.5.0/bin:$PATH
```

### Perl 5.34.3
```bash
cd perl5
tar xzf perl-5.34.3-ppc.tar.gz
sudo mv perl-5.34.3 /usr/local/
export PATH=/usr/local/perl-5.34.3/bin:$PATH
```

## Verification

```bash
# Check GCC version
gcc --version
# gcc (GCC) 7.5.0 or 10.5.0

# Check Perl version
perl -v
# This is perl 5, version 34, subversion 3

# Test C++17 features (GCC 7+)
echo 'int main() { auto x = [](auto a) { return a; }; }' | gcc -std=c++17 -x c++ -
```

## Building From Source

If you want to build these yourself:

### GCC Bootstrap
```bash
# On Leopard with Xcode 3.1 (GCC 4.2)
cd gcc-7.5.0
./contrib/download_prerequisites
mkdir build && cd build
../configure --prefix=/usr/local/gcc-7.5.0 \
    --enable-languages=c,c++ \
    --disable-multilib
make -j2 bootstrap
sudo make install
```

### Prerequisites
- Xcode 2.5 (Tiger) or Xcode 3.1 (Leopard)
- At least 2GB RAM
- 5GB free disk space
- Patience (bootstrap takes 4-8 hours)

## Hardware Tested

| Machine | CPU | RAM | OS | Status |
|---------|-----|-----|-----|--------|
| Power Mac G5 | Dual 2.0 GHz | 8GB | Leopard 10.5 | Works |
| Power Mac G4 | Dual 1.25 GHz | 2GB | Tiger 10.4 | Works |
| PowerBook G4 | 1.67 GHz | 2GB | Tiger 10.4 | Works |
| iMac G5 | 1.8 GHz | 2GB | Tiger 10.4 | Works |

## Related Projects

- [rust-ppc-tiger](https://github.com/Scottcjn/rust-ppc-tiger) - Rust compiler for PowerPC
- [llama-cpp-tigerleopard](https://github.com/Scottcjn/llama-cpp-tigerleopard) - LLM inference on G4/G5
- [ppc-tiger-tools](https://github.com/Scottcjn/ppc-tiger-tools) - More PowerPC tools

## Attribution

**A year of development, real PowerPC hardware, electricity bills, and a dedicated lab went into this.**

If you use these compilers, please give credit:

```
PowerPC Compilers by Scott (Scottcjn)
https://github.com/Scottcjn/ppc-compilers
```

If this helped you, please:
- ⭐ **Star this repo** - It helps others find it
- 📝 **Credit in your project** - Keep the attribution
- 🔗 **Link back** - Share the love

## License

MIT License - Free to use, but please keep the copyright notice and attribution.

---

*"Modern compilers on your 2005 Power Mac. Because vintage hardware deserves modern tools."*

**Keywords**: PowerPC, PPC, G4, G5, Mac OS X Tiger, Leopard, 10.4, 10.5, GCC, compiler, C++, C++11, C++14, C++17, Perl, vintage Mac, retro computing
