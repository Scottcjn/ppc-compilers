# Contributing to ppc-compilers

Thanks for helping improve the PowerPC compiler archive. This repository is
focused on modern compiler and toolchain packages for Mac OS X Tiger and
Leopard on PowerPC hardware.

## What to Contribute

Useful contributions include:

- Documentation fixes for installation, verification, and troubleshooting.
- Notes from testing on real PowerPC G4 or G5 hardware.
- Reproducible build instructions for GCC, Perl, Python, or supporting tools.
- Small scripts that make package verification easier.
- Corrections to package metadata, checksums, links, or compatibility notes.

Please keep changes scoped. If you want to add a new compiler package or a
large binary archive, open an issue first so maintainers can confirm the
version, target OS, license, and expected storage impact.

## Before Opening a Pull Request

1. Check existing issues and pull requests for related work.
2. Keep generated archives and large binaries unchanged unless the change is
   the point of the pull request.
3. Test documentation commands where practical on the target OS or clearly
   state when testing was limited to review only.
4. Run a whitespace check before committing:

   ```bash
   git diff --check
   ```

5. If you change scripts or build notes, include the exact command you used
   and the machine or OS version tested.

## Pull Request Checklist

- Explain which compiler, tool, or documentation area changed.
- Mention the PowerPC hardware and Mac OS X version used for validation, if
  applicable.
- Keep unrelated formatting and wording churn out of the diff.
- Confirm that license references still match the repository license.
- Follow the project code of conduct in `CODE_OF_CONDUCT.md`.

## Reporting Issues

When reporting a package or install problem, include:

- Machine model and CPU type.
- Mac OS X version.
- Package filename and version.
- The command that failed.
- The complete error message or the smallest useful excerpt.

This information helps maintainers reproduce issues on vintage hardware where
small OS and architecture differences matter.
