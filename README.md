# Secure CLI Password Manager

A command-line credential manager built in Python with a focus on cryptographic security. Implements AES-CBC encryption at rest, SHA-256 master password hashing with salting, and a complete credential lifecycle — generation, storage, retrieval, rotation, backup, and secure erasure.

## Why I Built This

Commercial password managers are great, but I wanted to understand the cryptographic primitives under the hood. This project forced me to make real decisions about key derivation, encryption modes, IV management, and secure memory handling — things you don't learn from just reading about them.

## Security Architecture

```
  Master Password
        │
        ▼
  ┌─────────────┐
  │  PBKDF2 or  │   ← Salt (randomly generated per database)
  │  SHA-256    │   ← Iterations: 100,000+
  └──────┬──────┘
         │
    Derived Key
         │
    ┌────┴────┐
    │         │
    ▼         ▼
  Auth      Encryption
  Hash      Key (AES-256)
    │         │
    │    ┌────┴────┐
    │    │ AES-CBC │ ← IV (unique per record)
    │    └────┬────┘
    │         │
    ▼         ▼
  Stored    Encrypted
  in DB     Credentials
```

### Encryption

- **Algorithm:** AES-256-CBC via PyCryptodome
- **Key Derivation:** Master password → SHA-256 hash → used as AES key
- **IV Handling:** A unique, randomly generated 16-byte IV is created for each credential record and stored alongside the ciphertext
- **Padding:** PKCS7 padding applied before encryption

### Authentication

- **Master Password Verification:** SHA-256 hash of the master password is stored on first setup. On subsequent access, the entered password is hashed and compared.
- **No Plaintext Storage:** The master password is never written to disk. The hash is used only for verification — the actual encryption key is derived separately.

### Credential Lifecycle

| Feature | Description |
|---------|-------------|
| **Add** | Encrypt and store a new credential record |
| **Retrieve** | Decrypt and display a stored credential |
| **Generate** | Create a cryptographically random password with configurable length and character classes |
| **Rotate** | Replace an existing credential's password with a newly generated one |
| **Backup** | Export an encrypted copy of the full database |
| **Erase** | Securely delete the database file (overwrite before unlink) |

## Usage

```bash
# First run — set up master password
python3 password_manager.py --init

# Add a credential
python3 password_manager.py --add
  Site: github.com
  Username: rainejohnson
  Password: [auto-generate? Y/n]

# Retrieve a credential
python3 password_manager.py --get github.com

# Generate a standalone password
python3 password_manager.py --generate --length 24 --symbols

# Rotate a stored password
python3 password_manager.py --rotate github.com

# Backup the encrypted database
python3 password_manager.py --backup

# Erase everything
python3 password_manager.py --erase
  ⚠ This will permanently destroy all stored credentials. Type 'CONFIRM' to proceed:
```

## Known Limitations & Future Improvements

**Current limitations (documented intentionally — this is a learning project):**

- **ECB vs CBC:** The current implementation uses CBC, which is secure for this use case, but authenticated encryption (AES-GCM) would be stronger by providing integrity verification in addition to confidentiality.
- **Key Derivation:** Using SHA-256 directly is functional but not ideal. A proper KDF like PBKDF2 or Argon2 with a high iteration count would provide better resistance to brute-force attacks. Migrating to Argon2id is on the roadmap.
- **Memory Handling:** Python doesn't guarantee that sensitive strings are wiped from memory after use. A production tool would need to use `ctypes` or a language with manual memory management for key material.
- **No Clipboard Integration:** Passwords are printed to stdout. A production version would copy to clipboard with auto-clear.

**Planned improvements:**
- [ ] Migrate from SHA-256 to Argon2id for key derivation
- [ ] Add AES-GCM for authenticated encryption
- [ ] Implement clipboard copy with 15-second auto-clear
- [ ] Add TOTP (2FA) secret storage support
- [ ] Build an import feature for CSV exports from other password managers

## Technologies

![Python](https://img.shields.io/badge/-Python-3776AB?style=flat&logo=Python&logoColor=white)
![PyCryptodome](https://img.shields.io/badge/-PyCryptodome-333333?style=flat)
![Linux CLI](https://img.shields.io/badge/-Linux%20CLI-FCC624?style=flat&logo=linux&logoColor=black)
