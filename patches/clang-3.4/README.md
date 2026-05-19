# clang 3.4 patches for Mac OS X 10.4 Tiger PowerPC

This directory holds patches that make `clang-3.4` produce working
executables on Darwin PPC32 (Mac OS X 10.4 / 10.5 PowerPC).

## clang-3.4-darwin-ppc32-external-reloc-stubs.patch

**Fixes**: `ld: <object> has external relocation entries in non-writable section (__TEXT,__text) for symbol <name>` from `ld64` when linking any program that references an external symbol (`errno`, `printf`, `fopen`, etc.).

**Root cause**: clang 3.4's PPC32 backend doesn't route external symbol references through the `$non_lazy_ptr` stub indirection when the relocation model is `Reloc::Static`. On Darwin PPC32 the dynamic linker is always present, so external references need NLP indirection regardless of relocation model — otherwise the bare `lis r3, ha16(_sym)` materializes as a `PPC_RELOC_HA16` Mach-O relocation with `r_extern=1` against the non-writable `__TEXT,__text` section, which `ld64` rejects.

### The fix has three hunks

1. **`lib/Target/PowerPC/PPCSubtarget.cpp::hasLazyResolverStub`** *(load-bearing — verified end-to-end on real G4 hardware)* — this is the predicate `PPCISelLowering::GetLabelAccessInfo` queries to decide whether to set `MO_NLP_FLAG` on the operand. Returning true for Darwin PPC32 even in `Reloc::Static` causes the existing `PPCMCInstLower::GetSymbolFromOperand` machinery to append `$non_lazy_ptr` and emit the canonical scattered `SECTDIFF` relocation against the local NLP label.

2. **`lib/Target/PowerPC/PPCAsmPrinter.cpp::printOperand`** *(defensive)* — covers the textual `.s` emission path for `MO_ExternalSymbol` and `MO_GlobalAddress` (external-or-weak branch). Same Darwin-static-routes-to-stub treatment.

3. **`lib/Target/PowerPC/MCTargetDesc/PPCMachObjectWriter.cpp::RecordPPCRelocation`** *(defensive)* — `report_fatal_error` if any external HI16/LO16/HA16 reloc ever tries to land in `__TEXT,__text` going forward. Future-regression net.

**Dual-brain verification**: Claude Opus 4.7 and Codex `gpt-5.4` independently arrived at hunks 1 and 2 (Codex contributed hunk 3). The actual load-bearing site was confirmed by empirical test on real G4 hardware — the asm-printer-only fix from the initial round was insufficient because the SelectionDAG codegen path bypasses `printOperand` entirely for global-address materialization. The `PPCSubtarget::hasLazyResolverStub` change was required to make the predicate return true for Darwin PPC32 in static mode.

**Verified on**: dual PowerMac G4 1.25 GHz / Mac OS X 10.4.12 Tiger / `Lee-Crockers-Powermac-G4.local`. End-to-end: source → patched `llc` → Tiger `as` → `gcc-apple-4.2` link → `Mach-O executable ppc` that runs.

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

Hand-written test case `extref.ll`:

```llvm
target triple = "powerpc-apple-darwin8"
@errno = external global i32
define i32* @get_errno_addr() { ret i32* @errno }
```

Patched `llc -relocation-model=static` produces:

```asm
lis r2, ha16(L_errno$non_lazy_ptr)
lwz r3, lo16(L_errno$non_lazy_ptr)(r2)
blr

.section __DATA,__nl_symbol_ptr,non_lazy_symbol_pointers
L_errno$non_lazy_ptr:
    .indirect_symbol  _errno
    .long 0
```

`otool -rv extref.o`:

```
External relocation information 0 entries
Local relocation information 0 entries
Relocation information (__TEXT,__text) 4 entries
  00000004  LO16  → 3 (__DATA,__nl_symbol_ptr)
            PAIR  half = 0x0000
  00000000  HA16  → 3 (__DATA,__nl_symbol_ptr)
            PAIR  half = 0x000c
```

End-to-end with a `main()` that returns `&errno`:

```
$ gcc-apple-4.2 mainref.o -o mainref
$ file mainref
mainref: Mach-O executable ppc
$ ./mainref ; echo "EXIT: $?"
EXIT: 28
```

(Exit code 28 = `&errno` cast to `int` = real runtime address of `errno` in Tiger's libSystem.)

### MacPorts integration

Add to the `clang-3.4` Portfile:

```tcl
patchfiles-append clang-3.4-darwin-ppc32-external-reloc-stubs.patch
```

### Test regression risk

Tightly scoped: PowerPC + Darwin + `Reloc::Static` corner only. Most likely test fallout would be in `test/CodeGen/PowerPC/*-darwin*.ll` or `test/MC/PowerPC/ppc-reloc.s` if any test asserts on bare external relocs under `-relocation-model=static`. Those would need updates to expect the indirection path; that's the correct expectation now.

### Tiger compatibility patches needed alongside

LLVM 3.4 doesn't build cleanly on Tiger without two unrelated compat fixes. These are unrelated to the relocation bug above but needed to get the build through:

- `clang-3.4-tiger-compat-Path.patch` — gates `_CS_DARWIN_USER_TEMP_DIR` / `_CS_DARWIN_USER_CACHE_DIR` (Leopard+ symbols) behind `defined(_CS_DARWIN_USER_TEMP_DIR)` so Tiger falls through to TMPDIR.
- `clang-3.4-tiger-compat-Signals.patch` — gates `EXC_MASK_CRASH` / `MACH_EXCEPTION_CODES` (Leopard+ Mach API) behind `defined(EXC_MASK_CRASH)` so Tiger skips the crash-override Mach exception block.

To be added to this directory in a follow-up commit.

### Upstream status

The clang 3.4 release branch is archived. This patch is maintained as a downstream Tiger PPC compatibility patch and is intended for the MacPorts `clang-3.4` port. Apply via the port's `patchfiles` mechanism.
