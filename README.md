# Hardening of PIN-Derived Key Protection Against Offline Brute-Force Attacks

**A controlled comparison of PBKDF2 and Argon2id**

Course project for *Information Security (23CSE332)*, VII Semester, Amrita Vishwa Vidyapeetham.

## Overview

Short numeric PINs are weak secrets, so the key derivation function (KDF) that turns a PIN into an encryption key is the main barrier against guessing. Kim and Choi [1] showed that Windows Hello private-key files on devices without TPM protection can be attacked offline, with PBKDF2 (10,000 iterations) as the key-derivation step.

This project builds a small controlled model of that protection layer and measures what changes when PBKDF2 is replaced by the memory-hard Argon2id:

- a **vault** that protects a dummy secret with a PIN, a random salt, a KDF and AES-GCM
- an **offline cracker** that brute-forces the PIN against the vault
- a **benchmark** that times the KDF alone and converts it to guesses per second

This is **not** a Windows Hello implementation, and no real system or account is attacked. Everything runs locally against vaults we create ourselves.

## Key results

Measured on Windows 11, Python 3.13.1, Intel Core i5-10210U, CPU only, 10 timed trials per setting.

| Configuration | Mean (ms) | Guesses/s | Slowdown |
| --- | ---: | ---: | ---: |
| PBKDF2-HMAC-SHA512 (10,000 it.) | 11.04 | 90.55 | 1.00x |
| Argon2id, 16 MiB | 19.75 | 50.63 | 1.79x |
| Argon2id, 32 MiB | 33.69 | 29.69 | 3.05x |
| Argon2id, 64 MiB | 66.15 | 15.12 | 5.99x |
| Argon2id, 128 MiB | 130.17 | 7.68 | 11.79x |

End-to-end attack on the 4-digit PIN `1234` took 1,235 guesses with both KDFs: about 13-15 s with PBKDF2 and 134-142 s with Argon2id (64 MiB).

Argon2id does not stop brute force. In this CPU-only setting it raises the cost of every guess, at the price of a slower derivation for legitimate users too.

## Repository contents

| File | Purpose |
| --- | --- |
| `vault.py` | Key derivation (PBKDF2, Argon2id), AES-GCM encryption, vault creation |
| `cracker.py` | Offline brute-force attacker (zero-padded PIN enumeration) |
| `benchmark.py` | KDF-only timing: 1 warm-up + 10 timed calls per configuration |
| `validation.py` | 8 correctness checks to run before trusting any timings |
| `kdf_benchmark_summary.csv` | Final benchmark dataset used in the report |
| `main.tex`, `figures/` | LaTeX source (IEEE conference template) and figures for the report |

## Experimental configuration

| Parameter | Value |
| --- | --- |
| Test PIN | `1234` (4 digits, 10,000 candidates) |
| Salt / nonce | 16 / 12 random bytes |
| Derived key | 32 bytes |
| Baseline KDF | PBKDF2-HMAC-SHA512, 10,000 iterations |
| Argon2id | `time_cost=3`, `parallelism=4`, memory 16 / 32 / 64 / 128 MiB |
| Encryption | AES-GCM (authentication failure is the cracker's wrong-guess signal) |

Only the Argon2id memory cost is varied between runs.

## Setup

Requires Python 3.10+ (developed on 3.13.1).

```bash
pip install argon2-cffi cryptography
```

Tested with `argon2-cffi 25.1.0` and `cryptography 50.0.2`. `hashlib` is part of the standard library.

## Usage

Run the checks first, then the benchmark and the attack.

```bash
python validation.py   # expect: Validation result: 8/8 checks passed
python benchmark.py    # writes the benchmark summary (mean, std, min, max, guesses/s)
python cracker.py      # attacks a PBKDF2 vault and an Argon2id vault for PIN 1234
```

<!-- TODO: add any command-line arguments your scripts take (KDF choice, PIN length, memory size) -->

## Building the report

The report source uses `\documentclass[conference]{IEEEtran}`.

1. Upload the project to Overleaf (New Project, then Upload Project) or compile locally with `pdflatex`.
2. Set `main.tex` as the main file and compile with pdfLaTeX.
3. Keep the `figures/` folder next to `main.tex`.

## Limitations

- Controlled local simulation only; no real Windows Hello installation was attacked.
- CPU-only, single process. No GPU or specialised-hardware benchmark was run, so the memory-hardness benefit against such attackers comes from the literature, not from these experiments.
- Results depend on our hardware and background load; 10 trials per setting is a small sample.
- Only a 4-digit PIN was attacked end to end. Longer-PIN figures are extrapolations from the benchmark rate.
- The end-to-end and KDF-only Argon2id rates differ (about 9 vs 15 guesses/s at 64 MiB) and the cause was not isolated.
- Only memory cost was varied; time cost and parallelism were fixed.

## Authors

| Name | Roll No. |
| --- | --- |
| N. Janvitha Reddy | AV.EN.U4CCE23027 |
| P. Hima Varshini | AV.EN.U4CCE23031 |
| K. Sravya Reddy | AV.EN.U4CCE23041 |

## References

1. E. Kim and H.-K. Choi, "Security analysis and bypass user authentication bound to device of Windows Hello in the wild," *Security and Communication Networks*, vol. 2021, Art. no. 6245306, 2021. doi:10.1155/2021/6245306.
2. K. Moriarty, B. Kaliski, and A. Rusch, "PKCS #5: Password-based cryptography specification version 2.1," RFC 8018, 2017.
3. M. S. Turan et al., "Recommendation for password-based key derivation, part 1: Storage applications," NIST SP 800-132, 2010.
4. A. Biryukov, D. Dinu, and D. Khovratovich, "Argon2: New generation of memory-hard functions for password hashing and other applications," IEEE EuroS&P, 2016.
5. A. Biryukov et al., "Argon2 memory-hard function for password hashing and proof-of-work applications," RFC 9106, 2021.
6. C. Percival, "Stronger key derivation via sequential memory-hard functions," BSDCan, 2009.
