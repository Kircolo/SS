# SS: Schmidt-Samoa Cryptosystem in C

This directory contains a complete Schmidt-Samoa (SS) public-key cryptosystem implementation in C using GMP.

It provides:
- `keygen`: generate SS public/private keys
- `encrypt`: encrypt data with a public key
- `decrypt`: decrypt data with a private key
- unit/integration tests for number theory and SS round-trip behavior

## Project Layout

- `Makefile`: build targets for binaries and tests.
- `keygen.c`: CLI key generation program.
- `encrypt.c`: CLI file/stream encryption program.
- `decrypt.c`: CLI file/stream decryption program.
- `ss.c`, `ss.h`: SS key generation, key I/O, block encrypt/decrypt logic.
- `numtheory.c`, `numtheory.h`: `gcd`, `mod_inverse`, `pow_mod`, Miller-Rabin primality, prime generation.
- `randstate.c`, `randstate.h`: global GMP RNG initialization/cleanup.
- `tests_numtheory.c`: focused tests for number-theory helpers.
- `tests_ss.c`: round-trip tests for encrypt/decrypt over edge-case sizes.
- `check_keygen.sh`, `check_encrypt.sh`, `check_decrypt.sh`: shell-based behavior checks.
- `README.md`: this file.

## Dependencies

- `clang`
- `make`
- `pkg-config`
- `gmp` (headers + library)

The build uses:
- `CFLAGS`: `-g -Wall -Wpedantic -Werror -Wextra`
- `pkg-config --cflags gmp` and `pkg-config --libs gmp`

## Build

From the repo root:

```bash
make            # builds keygen, encrypt, decrypt
make tests      # builds tests_numtheory and tests_ss
```

Useful targets:

```bash
make all
make check-numtheory
make check-ss
make clean
make format
```

## Usage

### 1) Generate keys

```bash
./keygen
```

Defaults:
- public key file: `ss.pub`
- private key file: `ss.priv` (chmod `0600`)
- modulus size minimum: `1024` bits
- Miller-Rabin iterations: `50`
- RNG seed: `time(NULL)`

Help:

```bash
./keygen -h
```

### 2) Encrypt

```bash
./encrypt -i plaintext.txt -o ciphertext.txt -n ss.pub
```

Or stream mode:

```bash
echo "hello" | ./encrypt > ciphertext.txt
```

Help:

```bash
./encrypt -h
```

### 3) Decrypt

```bash
./decrypt -i ciphertext.txt -o recovered.txt -n ss.priv
```

Or stream mode:

```bash
cat ciphertext.txt | ./decrypt
```

Help:

```bash
./decrypt -h
```

## Key File Formats

- `ss.pub`
  - line 1: `n` in lowercase hex
  - line 2: username (from `$USER`)

- `ss.priv`
  - line 1: `pq` in lowercase hex
  - line 2: `d` in lowercase hex

## Encryption/Decryption Block Notes

- Public modulus is `n = p^2 * q`.
- Encryption uses block size:
  - `k = floor((log2(sqrt(n)) - 1) / 8)`
  - payload per block = `k - 1` bytes
- A leading sentinel byte `0xFF` is prepended before conversion to an integer.
- Ciphertext is written as one hex integer per line.
- Decryption drops the sentinel byte and reconstructs original bytes.

## Testing

### C test binaries

```bash
make tests
./tests_numtheory
./tests_ss
```

### Shell check scripts

```bash
./check_keygen.sh
./check_encrypt.sh
./check_decrypt.sh
```

These validate CLI behavior, key/ciphertext formatting, block boundary cases, and round-trip correctness.

## Verified Status

Last verified in this workspace:

```bash
make clean && make all tests
./tests_numtheory
./tests_ss
```

Result: all build targets and both C test suites passed.
