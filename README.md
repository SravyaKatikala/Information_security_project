# 🔐 PIN Vault Cracker: PBKDF2 vs Argon2id

A small Python toolkit that encrypts a secret with a PIN, brute-forces it offline, and measures how much slower the attack gets when you switch from **PBKDF2** to **Argon2id**.

![Python](https://img.shields.io/badge/python-3.10%2B-blue)
![Crypto](https://img.shields.io/badge/AES--GCM-PBKDF2%20%7C%20Argon2id-green)
![Use](https://img.shields.io/badge/use-educational%20only-orange)

## What it does

- 🗄️ **Vault**: encrypts a secret with a PIN-derived key (random salt, AES-GCM)
- 🔨 **Cracker**: offline brute-force of every PIN (`0000`, `0001`, ...)
- ⏱️ **Benchmark**: times the KDF alone and reports guesses per second
- ✅ **Validation**: 8 sanity checks to run before trusting any numbers

Based on the offline PIN-guessing scenario from Kim & Choi, *Windows Hello in the Wild* (2021). This is a simplified local model, not a Windows Hello implementation.

## Results

Tested on Windows 11, Python 3.13.1, Intel Core i5-10210U (CPU only).

| KDF | Guesses/s | Slower than PBKDF2 |
| --- | ---: | ---: |
| PBKDF2-HMAC-SHA512 (10,000 it.) | 90.55 | 1.00x |
| Argon2id, 16 MiB | 50.63 | 1.79x |
| Argon2id, 32 MiB | 29.69 | 3.05x |
| Argon2id, 64 MiB | 15.12 | 5.99x |
| Argon2id, 128 MiB | 7.68 | 11.79x |

Cracking PIN `1234` (1,235 guesses): **~13-15 s** with PBKDF2 vs **~134-142 s** with Argon2id (64 MiB).

> Argon2id doesn't stop brute force. It makes every guess more expensive, for attackers and for legitimate logins.

## Quick start

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
pip install argon2-cffi cryptography
```

```bash
python validation.py   # should print: 8/8 checks passed
python benchmark.py    # KDF timing and guesses/s
python cracker.py      # attack a PBKDF2 vault and an Argon2id vault
```

## How it works

```
Vault:    PIN + salt -> KDF -> 32-byte key -> AES-GCM -> ciphertext
Cracker:  next PIN -> same KDF + salt -> candidate key -> AES-GCM decrypt
          decrypts OK -> PIN found | auth fails -> next guess
```

AES-GCM rejects a wrong key at decryption time, which is what tells the cracker whether a guess was right.

## Configuration

| Setting | Value |
| --- | --- |
| Test PIN | `1234` (4 digits) |
| Salt / nonce | 16 / 12 random bytes |
| Key length | 32 bytes |
| PBKDF2 | HMAC-SHA512, 10,000 iterations |
| Argon2id | `time_cost=3`, `parallelism=4`, memory 16/32/64/128 MiB |

## Project structure

```
├── vault.py                    # KDFs, AES-GCM, vault creation
├── cracker.py                  # offline brute-force attacker
├── benchmark.py                # KDF-only timing
├── validation.py               # correctness checks
├── kdf_benchmark_summary.csv   # benchmark results
└── report/                     # LaTeX report (IEEE format)
```

## Limitations

- CPU-only, single process. GPU attackers are not tested.
- Only a 4-digit PIN was attacked end to end.
- Results vary with hardware and background load (10 trials per setting).

## ⚠️ Disclaimer

For education and research only. It runs against vaults you create yourself. Don't use it against systems or data you don't own.

## Authors

- N. Janvitha Reddy (AV.EN.U4CCE23027)
- P. Hima Varshini (AV.EN.U4CCE23031)
- K. Sravya Reddy (AV.EN.U4CCE23041)

Amrita Vishwa Vidyapeetham · Information Security (23CSE332)
