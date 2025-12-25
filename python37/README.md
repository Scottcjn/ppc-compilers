# Python 3.7.17 for PowerPC G4 Tiger

**Modern Python 3 on your vintage Mac!**

## Installation

```bash
tar xzf python3.7-tiger-ppc32.tar.gz
sudo cp -r tiger_packages/python37_install/usr/local/* /usr/local/
export PATH=/usr/local/bin:$PATH
```

## Verification

```bash
python3 --version
# Python 3.7.17

python3 -c "import ssl; print(ssl.OPENSSL_VERSION)"
```

## Features

- Full Python 3.7 stdlib
- pip included
- SSL support (where available)
- Compatible with Tiger 10.4 on G4

## Hardware

Built and tested on Power Mac G4 Dual 1.25 GHz running Mac OS X Tiger 10.4.
