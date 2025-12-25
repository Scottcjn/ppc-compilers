# Modern Tools for PowerPC Tiger/Leopard

## GNU tar 1.30

Modern tar with xz/lzma support for Mac OS X Tiger.

**Features:**
- xz compression (`tar -J`)
- lzma compression (`--lzma`)
- gzip, bzip2 support
- Auto-detect compression (`-a`)

### Installation

```bash
tar xzf tar-1.30-tiger-ppc32.tar.gz
sudo cp tar-1.30-tiger-ppc32/bin/tar /usr/local/bin/
```

### Usage

```bash
# Extract xz archives
/usr/local/bin/tar xJf file.tar.xz

# Create xz archive
/usr/local/bin/tar cJf archive.tar.xz directory/
```

### Why You Need This

Tiger's built-in tar (1.14 from 2004) doesn't support xz or lzma compression.
Many modern source tarballs use `.tar.xz` format.
