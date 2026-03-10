---
layout: implementation
title: IPCrypt libsodium Implementation
description: High-performance C implementation of IPCrypt in libsodium, supporting all four encryption modes with hardware-accelerated AES.
permalink: /implementations/libsodium/
language: C
repository: https://github.com/jedisct1/libsodium
package_manager: libsodium
package_url: https://doc.libsodium.org/
examples:
  - title: Deterministic Encryption
    description: Encrypt an IP address using deterministic mode
    code: |
      #include <sodium.h>
      #include <stdio.h>

      int main(void) {
          unsigned char key[crypto_ipcrypt_KEYBYTES];
          unsigned char ip[16], encrypted_ip[16], decrypted_ip[16];

          if (sodium_init() < 0) return 1;

          /* Generate a random key */
          crypto_ipcrypt_keygen(key);

          /* Parse an IPv4 address into a 16-byte representation */
          /* 192.0.2.1 -> 0x00000000 0x00000000 0x0000FFFF 0xC0000201 */
          memset(ip, 0, 16);
          ip[10] = 0xff; ip[11] = 0xff;
          ip[12] = 192; ip[13] = 0; ip[14] = 2; ip[15] = 1;

          /* Encrypt */
          crypto_ipcrypt_encrypt(encrypted_ip, ip, key);

          /* Decrypt */
          crypto_ipcrypt_decrypt(decrypted_ip, encrypted_ip, key);
      }
  - title: Non-Deterministic Encryption (ND)
    description: Encrypt an IP address using non-deterministic mode with KIASU-BC
    code: |
      #include <sodium.h>

      int main(void) {
          unsigned char key[crypto_ipcrypt_KEYBYTES];
          unsigned char ip[16];
          unsigned char encrypted[crypto_ipcrypt_nd_BYTES]; /* 24 bytes */
          unsigned char decrypted_ip[16];

          if (sodium_init() < 0) return 1;

          crypto_ipcrypt_keygen(key);

          memset(ip, 0, 16);
          ip[10] = 0xff; ip[11] = 0xff;
          ip[12] = 192; ip[13] = 0; ip[14] = 2; ip[15] = 1;

          /* Encrypt (tweak is generated automatically) */
          crypto_ipcrypt_nd_encrypt(encrypted, ip, key);

          /* Decrypt */
          crypto_ipcrypt_nd_decrypt(decrypted_ip, encrypted, key);
      }
  - title: Non-Deterministic Extended Encryption (NDX)
    description: Encrypt an IP address using non-deterministic extended mode with AES-XTS
    code: |
      #include <sodium.h>

      int main(void) {
          unsigned char key[crypto_ipcrypt_ndx_KEYBYTES]; /* 32 bytes */
          unsigned char ip[16];
          unsigned char encrypted[crypto_ipcrypt_ndx_BYTES]; /* 32 bytes */
          unsigned char decrypted_ip[16];

          if (sodium_init() < 0) return 1;

          crypto_ipcrypt_ndx_keygen(key);

          memset(ip, 0, 16);
          ip[10] = 0xff; ip[11] = 0xff;
          ip[12] = 192; ip[13] = 0; ip[14] = 2; ip[15] = 1;

          /* Encrypt (tweak is generated automatically) */
          crypto_ipcrypt_ndx_encrypt(encrypted, ip, key);

          /* Decrypt */
          crypto_ipcrypt_ndx_decrypt(decrypted_ip, encrypted, key);
      }
---

## IPCrypt libsodium Implementation

[libsodium](https://libsodium.org) includes a high-performance, production-ready implementation of IPCrypt. It supports all four encryption modes and leverages hardware AES instructions (AES-NI / ARMv8 Crypto Extensions) when available.

## Installation

libsodium is available on all major platforms:

```bash
# macOS
brew install libsodium

# Debian / Ubuntu
apt install libsodium-dev

# Fedora / RHEL
dnf install libsodium-devel

# From source
git clone https://github.com/jedisct1/libsodium.git
cd libsodium
./configure && make && make install
```

## Requirements

- libsodium 1.0.21 or higher
- A C compiler (gcc, clang, MSVC)

## Usage

The libsodium implementation provides a simple C API for all four encryption modes. IP addresses are represented as 16-byte arrays (IPv4-mapped IPv6 format).

### Deterministic Encryption

```c
#include <sodium.h>

unsigned char key[crypto_ipcrypt_KEYBYTES];
unsigned char ip[16], encrypted_ip[16], decrypted_ip[16];

crypto_ipcrypt_keygen(key);

crypto_ipcrypt_encrypt(encrypted_ip, ip, key);
crypto_ipcrypt_decrypt(decrypted_ip, encrypted_ip, key);
```

### Non-Deterministic Encryption (ND)

```c
#include <sodium.h>

unsigned char key[crypto_ipcrypt_KEYBYTES];
unsigned char ip[16];
unsigned char encrypted[crypto_ipcrypt_nd_BYTES];  /* 24 bytes: 8-byte tweak + 16-byte ciphertext */
unsigned char decrypted_ip[16];

crypto_ipcrypt_keygen(key);

crypto_ipcrypt_nd_encrypt(encrypted, ip, key);
crypto_ipcrypt_nd_decrypt(decrypted_ip, encrypted, key);
```

### Non-Deterministic Extended Encryption (NDX)

```c
#include <sodium.h>

unsigned char key[crypto_ipcrypt_ndx_KEYBYTES];  /* 32 bytes */
unsigned char ip[16];
unsigned char encrypted[crypto_ipcrypt_ndx_BYTES];  /* 32 bytes: 16-byte tweak + 16-byte ciphertext */
unsigned char decrypted_ip[16];

crypto_ipcrypt_ndx_keygen(key);

crypto_ipcrypt_ndx_encrypt(encrypted, ip, key);
crypto_ipcrypt_ndx_decrypt(decrypted_ip, encrypted, key);
```

## API Reference

### Constants

| Constant | Value | Description |
|---|---|---|
| `crypto_ipcrypt_KEYBYTES` | 16 | Key size for deterministic and ND modes |
| `crypto_ipcrypt_INPUTBYTES` | 16 | IP address size (IPv4-mapped IPv6) |
| `crypto_ipcrypt_nd_BYTES` | 24 | ND ciphertext size (8-byte tweak + 16-byte ciphertext) |
| `crypto_ipcrypt_ndx_KEYBYTES` | 32 | Key size for NDX mode |
| `crypto_ipcrypt_ndx_BYTES` | 32 | NDX ciphertext size (16-byte tweak + 16-byte ciphertext) |

### Deterministic Encryption

```c
void crypto_ipcrypt_keygen(unsigned char k[crypto_ipcrypt_KEYBYTES]);

int crypto_ipcrypt_encrypt(unsigned char out[16], const unsigned char in[16],
                           const unsigned char k[crypto_ipcrypt_KEYBYTES]);

int crypto_ipcrypt_decrypt(unsigned char out[16], const unsigned char in[16],
                           const unsigned char k[crypto_ipcrypt_KEYBYTES]);
```

### Non-Deterministic Encryption (ND)

```c
int crypto_ipcrypt_nd_encrypt(unsigned char out[crypto_ipcrypt_nd_BYTES],
                              const unsigned char in[16],
                              const unsigned char k[crypto_ipcrypt_KEYBYTES]);

int crypto_ipcrypt_nd_decrypt(unsigned char out[16],
                              const unsigned char in[crypto_ipcrypt_nd_BYTES],
                              const unsigned char k[crypto_ipcrypt_KEYBYTES]);
```

### Non-Deterministic Extended Encryption (NDX)

```c
void crypto_ipcrypt_ndx_keygen(unsigned char k[crypto_ipcrypt_ndx_KEYBYTES]);

int crypto_ipcrypt_ndx_encrypt(unsigned char out[crypto_ipcrypt_ndx_BYTES],
                               const unsigned char in[16],
                               const unsigned char k[crypto_ipcrypt_ndx_KEYBYTES]);

int crypto_ipcrypt_ndx_decrypt(unsigned char out[16],
                               const unsigned char in[crypto_ipcrypt_ndx_BYTES],
                               const unsigned char k[crypto_ipcrypt_ndx_KEYBYTES]);
```

## Implementation Details

The libsodium implementation includes:

1. **Hardware Acceleration**: Uses AES-NI on x86/x86_64 and Crypto Extensions on ARMv8 for maximum performance
2. **Constant-Time Operations**: All operations are designed to run in constant time to prevent timing side-channel attacks
3. **Secure Memory**: Keys can be stored in secure memory using `sodium_malloc()` and locked with `sodium_mlock()`
4. **Cross-Platform**: Works on Linux, macOS, Windows, iOS, Android, and WebAssembly

## Supported Features

- IPv4 address encryption/decryption
- IPv6 address encryption/decryption
- Deterministic encryption (AES-128)
- Non-deterministic encryption (KIASU-BC)
- Extended non-deterministic encryption (AES-XTS)
- Hardware-accelerated AES (AES-NI, ARMv8)
- Constant-time implementation

## Compilation

```bash
# Compile with pkg-config
cc -o example example.c $(pkg-config --cflags --libs libsodium)

# Or link directly
cc -o example example.c -lsodium
```

## License

libsodium is licensed under the ISC License.
