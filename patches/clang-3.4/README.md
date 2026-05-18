# clang 3.4 patches for Mac OS X 10.4 Tiger PowerPC

This directory holds patches that make `clang-3.4` produce working
executables on Darwin PPC32 (Mac OS X 10.4 / 10.5 PowerPC).

## clang-3.4-darwin-ppc32-external-reloc-stubs.patch

**Fixes**: `ld: <object> has external relocation entries in non-writable section (__TEXT,__text) for symbol <name>` from `ld64` when linking any program that references an external symbol (`errno`, `printf`, `fopen`, etc.).

**Root cause**: clang 3.4's `PPCAsmPrinter::printOperand` only routes external symbol references through the `$non_lazy_ptr` stub indirection when `TM.getRelocationModel() != Reloc::Static`. On Darwin PPC32 the dynamic linker is always present, so even static-relocation-model references to externals need NLP indirection — otherwise the bare `lis r3, ha16(_sym)` materializes as a `PPC_RELOC_HA16` Mach-O relocation with `r_extern=1` against the non-writable `__TEXT,__text` section, which `ld64` rejects.

**Fix**: Two-line change at `lib/Target/PowerPC/PPCAsmPrinter.cpp:174` (`MO_ExternalSymbol`) and `:197` (`MO_GlobalAddress` external-or-weak branch). Both conditions now also fire when `Subtarget.isDarwin()`, regardless of relocation model. The existing `$non_lazy_ptr` machinery (`GetSymbolWithGlobalValueBase`, `MachineModuleInfoMachO::getGVStubEntry`) handles the rest — relocations land in `__DATA,__nl_symbol_ptr` (writable) as `PPC_RELOC_HA16_SECTDIFF` against a local NLP label, which is exactly the shape `ld64` expects.

**Diagnosed by**: dual-brain review (Dr. Claude Opus 4.7 + Codex `gpt-5.4`)
**Verified on**: dual PowerMac G4 1.25 GHz / Mac OS X 10.4.12 Tiger / `Lee-Crockers-Powermac-G4.local`

### How to apply

Against a clean `llvm-3.4.2.src` tree:

```bash
cd llvm-3.4.2.src
patch -p1 < /path/to/clang-3.4-darwin-ppc32-external-reloc-stubs.patch
```

Then rebuild as usual:

```bash
./configure --prefix=/opt/local --enable-optimized
make -j2          # 2 cores on dual G4 1.25
sudo make install
```

### Verification

Before the patch:

```bash
cat > hello.c << 'EOF2'
#include <stdio.h>
extern int errno;
int main(void) {
    printf("errno address: %p\n", (void*)&errno);
    return 0;
}
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
# Relocation information (__TEXT,__text)
# address  pcrel length extern type    scattered symbolnum/value
# 00000018 False long   n/a    HA16    True      0x00000038
# ...
# Relocation information (__DATA,__nl_symbol_ptr)
# 00000000 False long   True   VANILLA False     _errno
```

External relocations now sit in `__DATA,__nl_symbol_ptr` (writable), and `ld64` accepts them.

### Upstream status

The clang 3.4 release branch is archived. This patch is maintained as a downstream Tiger PPC compatibility patch and is intended for the MacPorts `clang-3.4` port. Apply via the port's `patchfiles` mechanism.
