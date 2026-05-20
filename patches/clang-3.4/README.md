# clang 3.4 patches for Mac OS X 10.4 Tiger PowerPC

Patches that make `clang-3.4` produce working executables on Darwin PPC32 (Mac OS X 10.4 / 10.5 PowerPC). **Verified: patched clang 3.4 compiles and runs real C programs on a dual PowerMac G4 1.25 GHz / Mac OS X 10.4.12.**

## clang-3.4-darwin-ppc32-external-reloc-stubs.patch

**Fixes**: `ld: <object> has external relocation entries in non-writable section (__TEXT,__text)` from `ld64` when linking any program that references *or calls* an external symbol (`errno`, `printf`, `malloc`, …).

**Root cause** — two codegen paths, same underlying mistake. clang 3.4's PPC32 backend emits bare external references inside `__TEXT,__text` instead of routing through the indirection stubs `ld64` expects, because the stub logic short-circuits on `Reloc::Static`. On Darwin PPC32 the dynamic linker is always present, so the indirection is needed regardless of relocation model — Tiger's `ld64` does not auto-synthesize these stubs the way Leopard+ does.

### The fix — 5 hunks, 4 files

| # | File | Class | Role |
|---|---|---|---|
| 1 | `PPCSubtarget.cpp::hasLazyResolverStub` | data refs | **Load-bearing.** True for Darwin PPC32 even in `Reloc::Static` → `MO_NLP_FLAG` → `PPCMCInstLower` appends `$non_lazy_ptr` |
| 2 | `PPCAsmPrinter.cpp::printOperand` | data refs | Defensive — textual `.s` path |
| 3 | `PPCMachObjectWriter.cpp::RecordPPCRelocation` | data refs | Defensive — `report_fatal_error` guard against regressions |
| 4 | `PPCISelLowering.cpp` GlobalAddress-call | calls | Drops `!= Reloc::Static` gate on `MO_DARWIN_STUB` (block already pre-Leopard-scoped) |
| 5 | `PPCISelLowering.cpp` ExternalSymbol-call | calls | Same — external function calls route through `$stub` |

Data references (`&errno`) → `__DATA,__nl_symbol_ptr`. Function calls (`printf`) → `__TEXT,__symbol_stub1` branch islands. Both writable / linker-accepted.

**Dual-brain verified** (Claude Opus 4.7 + Codex `gpt-5.4`), then empirically corrected on real G4 — static analysis converged on hunks 1+2; hardware testing exposed that the SelectionDAG path bypasses `printOperand` (→ hunk 1 is the real gate) and that function calls are a separate path (→ hunks 4+5).

### How to apply

```bash
cd llvm-3.4.2.src
patch -p1 < clang-3.4-darwin-ppc32-external-reloc-stubs.patch
```

Build:

```bash
PATH=/opt/local/bin:$PATH \
  CC=/opt/local/bin/gcc-apple-4.2 CXX=/opt/local/bin/g++-apple-4.2 \
  PYTHON=/opt/local/bin/python2.7 \
  ./configure --prefix=/opt/local --enable-optimized --disable-assertions \
    --enable-targets=powerpc --with-python=/opt/local/bin/python2.7
make -j2 ENABLE_OPTIMIZED=1 DISABLE_ASSERTIONS=1
sudo make install
```

### Verification

```
$ clang -no-integrated-as he.c -o he       # extern int errno; printf("%p", &errno)
$ file he
he: Mach-O executable ppc
$ ./he
errno=0xa0011b1c

$ clang -no-integrated-as -std=c99 smoke.c -o smoke   # recursion + malloc + sprintf + strlen
$ ./smoke
clang 3.4 on Tiger PPC
fib(0) = 0  ...  fib(9) = 34
heap-1, strlen=6
```

Use `-no-integrated-as` — Tiger's `as` handles clang's emitted `.s`; clang 3.4's integrated assembler emits CFI pseudos Tiger's `as` rejects (cosmetic, separate issue). No `-mdynamic-no-pic` workaround needed — default invocation works.

### Test regression risk

PowerPC + Darwin + `Reloc::Static` corner only. Likely fallout in `test/CodeGen/PowerPC/*-darwin*.ll` or `test/MC/PowerPC/ppc-reloc.s` if any test asserts bare external relocs under `-relocation-model=static` — those expectations should now expect the indirection path.

### Tiger compatibility patches needed alongside

LLVM 3.4 won't build on Tiger without 2 unrelated compat fixes:
- `clang-3.4-tiger-compat-Path.patch` — gate `_CS_DARWIN_USER_TEMP_DIR` / `_CS_DARWIN_USER_CACHE_DIR` (Leopard+) behind `defined()` (clang 3.5 fixed this upstream).
- `clang-3.4-tiger-compat-Signals.patch` — gate `EXC_MASK_CRASH` / `MACH_EXCEPTION_CODES` (Leopard+ Mach API) behind `defined(EXC_MASK_CRASH)`.

To be added to this directory in a follow-up commit.

### MacPorts integration

```tcl
patchfiles-append clang-3.4-darwin-ppc32-external-reloc-stubs.patch
```

### Upstream status

The clang 3.4 release branch is archived. Maintained as a downstream Tiger PPC compatibility patch for the MacPorts `clang-3.4` port. The same bug + fix carries forward to clang 3.5 (the `hasLazyResolverStub` short-circuit is identical).
