# Cryptography

## Overview

Cryptography covers the science of securing data — from foundational concepts like encryption and hashing, to hands-on use of tools for decrypting messages and cracking password hashes. The room builds an understanding of how data is protected and how those protections can be tested or broken in a controlled, ethical context.

---

## Core Concepts

### Encryption Terminology

| Term         | Definition                                                                                                   |
| ------------ | ------------------------------------------------------------------------------------------------------------ |
| **Plaintext**  | The original, readable message or data before encryption                                                   |
| **Ciphertext** | The scrambled, unreadable output after encryption                                                          |
| **Cipher**     | The algorithm used to convert plaintext to ciphertext and back                                             |
| **Key**        | The string of bits the cipher uses to encrypt or decrypt — must remain secret                              |
| **Encryption** | The process of converting plaintext into ciphertext using a cipher and key                                 |
| **Decryption** | The reverse — converting ciphertext back to plaintext; impossible without the correct key                  |

---

### Symmetric Encryption

Both sender and receiver share the same key for encryption and decryption.

| Algorithm | Key Size          | Notes                                                                 |
| --------- | ----------------- | --------------------------------------------------------------------- |
| DES       | 56-bit            | Adopted 1977 — broken in under 24 hours by 1999; no longer secure    |
| 3DES      | 168-bit (112 effective) | DES applied three times; deprecated in 2019; may exist in legacy systems |
| AES       | 128 / 192 / 256-bit | Current standard, adopted in 2001                                 |

---

### Asymmetric Encryption

Uses a **public key** (shared openly) and a **private key** (kept secret). Data encrypted with the public key can only be decrypted with the private key. Examples include RSA, Diffie-Hellman, and Elliptic Curve Cryptography (ECC).

Asymmetric encryption tends to use larger keys and is slower than symmetric encryption — it is typically used to establish a secure channel, after which symmetric encryption handles the bulk transfer.

---

### Digital Signatures

Digital signatures use asymmetric cryptography to verify **authenticity** and **integrity** — confirming that a message genuinely came from the claimed sender and has not been altered. A signature is produced using the sender's private key and verified using their public key.

Key signature algorithms include DSA, ECDSA, ECDSA-SK, Ed25519, and Ed25519-SK.

---

### Hashing

Hashing is a one-way process — a hash function takes an input of any size and produces a fixed-length output (the **hash value** or **digest**). Unlike encryption, there is no key and no intended way to reverse it.

Common hashing tools and commands:

```bash
md5sum filename        # Produces a 32-character hash
sha1sum filename       # Produces a 40-character hash
sha256sum filename     # Produces a 64-character hash
```

---

### Attack Methods

| Attack Type       | Description                                                          |
| ----------------- | -------------------------------------------------------------------- |
| Brute-Force       | Tries every possible key or password combination                     |
| Dictionary Attack | Tests words from a wordlist (e.g. `rockyou.txt`) against the hash   |

---

## Tools Used

### GPG (GNU Privacy Guard)

GPG is an open-source implementation of the OpenPGP standard, used for encrypting files and verifying digital signatures.

```bash
# Import a key from a backup file
gpg --import backup.key

# Decrypt an encrypted message
gpg --decrypt confidential_message.gpg
```

> Both the key file and the encrypted message must be present in the working directory, or the full path must be specified.

---

### Hashcat

Hashcat is a GPU-accelerated password recovery tool that cracks hashes using various attack modes.

**Core syntax:**

```bash
hashcat -m [hash-type] -a [attack-type] [target hash file] [wordlist/source]
```

**Common hash type flags (`-m`):**

| Flag    | Hash Type              |
| ------- | ---------------------- |
| `-m 0`  | MD5                    |
| `-m 100`| SHA1                   |
| `-m 1800` | sha512crypt ($6$)   |
| `-m 3200` | bcrypt ($2a$)       |

**Attack type flags (`-a`):**

| Flag   | Attack Type |
| ------ | ----------- |
| `-a 0` | Dictionary  |
| `-a 1` | Combinator  |
| `-a 3` | Mask        |
| `-a 6` | Hybrid      |

**Example — cracking a sha512crypt hash with a dictionary attack:**

```bash
hashcat -m 1800 -a 0 hash3.txt /usr/share/wordlists/rockyou.txt
```

**Displaying the cracked result after status shows "Cracked":**

```bash
hashcat -m 1800 hash3.txt --show
```

> The hash type is identified by its prefix or character count — for example, a 32-character hex string indicates MD5 (`-m 0`), while a `$6$` prefix indicates sha512crypt (`-m 1800`). Refer to the [Hashcat hash type reference](https://hashcat.net/wiki/doku.php?id=hashcat) when unsure.

---

### John the Ripper

John the Ripper is a versatile password cracking tool that supports a wide range of hash formats and attack modes.

**Core syntax:**

```bash
john [options] [path to hash file]
```

**Automatic cracking (when hash format is unknown):**

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash_to_crack.txt
```

**Format-specific cracking (recommended — more reliable):**

```bash
john --format=[format] --wordlist=/usr/share/wordlists/rockyou.txt hash_to_crack.txt
```

**Displaying results cleanly:**

```bash
john --show --format=raw-md5 hash_to_crack.txt
```

**Common format flags:**

| Hash Type             | `--format=` value |
| --------------------- | ----------------- |
| MD5 (32 chars)        | `raw-md5`         |
| SHA1 (40 chars)       | `raw-sha1`        |
| SHA256 (64 chars)     | `raw-sha256`      |
| SHA512 (`$6$`)        | `sha512crypt`     |
| bcrypt (`$2a$`)       | `bcrypt`          |
| NTLM (Windows)        | `nt`              |
| Whirlpool (128 chars) | `whirlpool`       |

> When using standard hash types like MD5, prefix the format with `raw-` (e.g. `raw-md5`). To confirm the exact format string, run: `john --list=formats | grep -iF "md5"`

---

#### Cracking `/etc/shadow` (Linux Password Hashes)

Linux stores password hashes in `/etc/shadow`, which requires root privileges to access. To crack these with John, it must first be combined with `/etc/passwd` using the `unshadow` tool.

```bash
unshadow local_passwd local_shadow > unshadowed.txt
john --wordlist=/usr/share/wordlists/rockyou.txt unshadowed.txt
```

---

#### Single Crack Mode

Uses the username itself to generate password guesses via **word mangling** — variations in capitalisation, appended numbers, and symbols.

```bash
john --single --format=raw-sha256 hashes.txt
```

> The hash file must be formatted as `username:hash` for Single Crack Mode to work. For example: `mike:1efee03cdcb96d90ad48ccc7b8666033`

---

#### Cracking Password-Protected Archives

**ZIP files:**

```bash
zip2john zipfile.zip > zip_hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt zip_hash.txt

# Extract once the password is recovered
unzip zipfile.zip
```

**RAR files:**

```bash
rar2john rarfile.rar > rar_hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt rar_hash.txt

# Extract once the password is recovered
unrar x rarfile.rar
```

---

#### Cracking SSH Private Keys

SSH private keys (`id_rsa`) can be passphrase-protected. The hash can be extracted and cracked with John.

```bash
/opt/john/ssh2john.py id_rsa > id_rsa_hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa_hash.txt
```

---

### Online Tools (for hashes without salts)

When a hash has no salt, online rainbow table lookups can crack it instantly without any local compute:

- [CrackStation](https://crackstation.net)
- [hashes.com](https://hashes.com)

For salted hashes, local tools like Hashcat or John the Ripper are required.

---

## Connection to Security+

Cryptography is a core domain in Security+. The exam covers symmetric vs. asymmetric encryption, hashing algorithms, digital signatures, and PKI concepts. Understanding these fundamentals — and having hands-on experience with how hashes are cracked — provides practical grounding for what Security+ tests at a conceptual level.

---

## Key Takeaway

Cryptography sits at the heart of nearly every security control — from HTTPS and file integrity verification to password storage and authentication. This room built both the conceptual vocabulary and the hands-on tool proficiency to engage with cryptographic systems practically: understanding how encryption and hashing work, and knowing how to test them using tools like Hashcat and John the Ripper, are skills that apply directly to both offensive and defensive security roles.
