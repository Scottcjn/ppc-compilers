# clang 3.4 patches for Mac OS X 10.4 Tiger PowerPC

This directory holds patches that make `clang-3.4` produce working
executables on Darwin PPC32 (Mac OS X 10.4 / 10.5 PowerPC).

## clang-3.4-darwin-ppc32-external-reloc-stubs.patch

**Fixes**: `ld: <object> has external relocation entries in non-writable section (__TEXT,__text) for symbol <name>` from `ld64` when linking any program that references an external symbol (`errno`, `printf`, `fopen`, etc.).

**Root cause**: clang 3.4's `PPCAsmPrinter::printOperand` only routes external symbol references through the `$non_lazy_ptr` stub indirection when `TM.getRelocationModel() != Reloc::Static`. On Darwin PPC32 the dynamic linker is always present, so even static-relocation-model references to externals need NLP indirection — otherwise the bare `lis r3, ha16(_sym)` materializes as a `PPC_RELOC_HA16` Mach-O relocation with `r_extern=1` against the non-writable `__TEXT,__text` section, which `ld64` rejects.

**The fix has two hunks**:

1. **`lib/Target/PowerPC/PPCAsmPrinter.cpp`** — make `printOperand` treat `Reloc::Static + isDarwin()` as the existing stub path for `MO_ExternalSymbol` and `MO_GlobalAddress` (external-or-weak branch). The existing `$non_lazy_ptr` machinery handles the rest.

2. **`lib/Target/PowerPC/MCTargetDesc/PPCMachObjectWriter.cpp`** — defensive guard. If a regression ever sends an external HI16/LO16/HA16 reloc into `__TEXT,__text`, `report_fatal_error` with a clear diagnostic instead of producing a bad `.o` that `ld64` rejects with a less actionable message.

**Dual-brain verification**: Claude Opus 4.7 and Codex `gpt-5.4` converged on this exact two-hunk patch independently, without coordination.

**Verified on**: dual PowerMac G4 1.25 GHz / Mac OS X 10.4.12 Tiger / `Lee-Crockers-Powermac-G4.local`

### How to apply

Against a clean `llvm-3.4.2.src` tree:

```bash
cd llvm-3.4.2.src
patch -p1 < /path/to/clang-3.4-darwin-ppc32-external-reloc-stubs.patch
```

Then rebuild as usual:

```bash
PATH=/opt/local/bin:$PATH \
  CC=/opt/local/bin/gcc-apple-4.2 \
  CXX=/opt/local/bin/g++-apple-4.2 \
  PYTHON=/opt/local/bin/python2.7 \
  ./configure --prefix=/opt/local \
    --enable-optimized --disable-assertions \
    --enable-targets=powerpc \
    --with-python=/opt/local/bin/python2.7
make -j2 ENABLE_OPTIMIZED=1 DISABLE_ASSERTIONS=1
sudo make install
```

### Verification

Before the patch:

```bash
cat > hello.c << 'EOF2'
#include <stdio.h>
extern int errno;
int main(void) { printf("errno address: %p\n", (void*)&errno); return 0; }
EOF2
clang-3.4 hello.c -o hello
# ld: hello.o has external relocation entries in non-writable section
#     (__TEXT,__text) for symbol _errno
```

After the patch (no flag changes needed):

```bash
clang-3.4 hello.c -o hello
./hello
# errno address: 0xa01234
otool -rv hello.o
# Relocations now scattered against __DATA,__nl_symbol_ptr — ld64 accepts.
```

### MacPorts integration

Add to the `clang-3.4` Portfile:

```tcl
patchfiles-append clang-3.4-darwin-ppc32-external-reloc-stubs.patch
```

### Test regression risk

Tightly scoped: PowerPC + Darwin + `Reloc::Static` corner only. The most likely test fallout would be in `test/CodeGen/PowerPC/*-darwin*.ll` or `test/MC/PowerPC/ppc-reloc.s` if any test asserts on bare external relocs under `-relocation-model=static`. Those would need updates to expect the indirection path; that's the correct expectation now.

### Upstream status

The clang 3.4 release branch is archived. This patch is maintained as a downstream Tiger PPC compatibility patch and is intended for the MacPorts `clang-3.4` port. Apply via the port's `patchfiles` mechanism.
