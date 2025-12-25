# Perl 5.34.3 for Mac OS X Tiger (PowerPC)

## Overview
- **Version**: Perl 5.34.3 (released 2022)
- **Target**: powerpc-apple-darwin8
- **Source**: MacPorts
- **Size**: ~13MB compressed

## Features
- 26 major versions ahead of stock Tiger (5.8.6)
- Modern Perl with 2022-era features
- Unicode 13.0 support
- Improved regex engine
- try/catch syntax (experimental)

## Installation
```bash
cd /opt/local
sudo tar xzf perl5-5.34-tiger-ppc.tar.gz
```

## After Installation
- `/opt/local/bin/perl5.34` → Perl interpreter
- `/opt/local/lib/perl5/` → Perl libraries

## Usage
```bash
# Run script
/opt/local/bin/perl5.34 script.pl

# Or add to PATH
export PATH=/opt/local/bin:$PATH
perl5.34 --version
```

## Why Perl 5.34?
- Modern build systems require recent Perl
- Many configure scripts need Perl 5.10+
- Unicode handling vastly improved
- Security fixes not in 5.8.x

## Stock Tiger vs Modern
| Feature | Perl 5.8.6 (2005) | Perl 5.34.3 (2022) |
|---------|-------------------|-------------------|
| Unicode | 4.0 | 13.0 |
| Regex | Basic | Advanced |
| say() | No | Yes |
| state | No | Yes |
| try/catch | No | Experimental |
