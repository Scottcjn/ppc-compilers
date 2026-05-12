# Contributing to ppc-compilers

Thanks for helping maintain ppc-compilers. This repository provides prebuilt
toolchains for PowerPC Mac OS X Tiger and Leopard, so accuracy, reproducibility,
and clear compatibility notes matter more than broad refactors.

## Useful Contributions

- Improve installation notes for GCC, Perl, Python, or related runtime packages.
- Add verified compatibility notes for specific Power Mac hardware and OS
  combinations.
- Document checksums, archive contents, or unpacking steps more clearly.
- Fix broken links, outdated package names, or confusing command examples.
- Add troubleshooting notes for common Tiger and Leopard build failures.

## Development Workflow

1. Fork the repository and create a small topic branch.
2. Keep changes focused on one toolchain, document, or compatibility issue.
3. Avoid replacing binary artifacts unless the PR explains the build source,
   build host, flags, checksum, and test result.
4. Include exact validation steps in the PR description.

## Validation

- Documentation-only changes: run `git diff --check`.
- Archive or checksum changes: verify the archive can be listed and extracted.
- Toolchain changes: include the host system, OS version, compiler command, and
  a minimal compile test.

Example minimal compile test:

```bash
cat > hello.c << 'EOF'
#include <stdio.h>
int main(void) { puts("hello ppc"); return 0; }
EOF
gcc hello.c -o hello
./hello
```

## Pull Request Checklist

- The PR states which package or OS target is affected.
- Install or validation commands are included.
- Compatibility risks for Tiger, Leopard, G4, or G5 are noted.
- Checksums are updated when archives change.
- Generated temporary files are not committed.

## Reporting Issues

Include the package name, archive name, Mac model, OS version, shell command,
and complete error output. For runtime failures, include `uname -a` and the
compiler or interpreter version when available.
