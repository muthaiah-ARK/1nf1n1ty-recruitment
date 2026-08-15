# AES - How AES Works

## Date

06 June 2026

## Platform

CryptoHack - AES

## Module

How AES Works

---

# 1. Symmetric Ciphers

AES is a symmetric block cipher.

A symmetric cipher uses the same secret key for encryption and decryption.

AES works on fixed-size blocks of data.

For AES-128:

* Key size: 128 bits (16 bytes)
* Block size: 128 bits (16 bytes)
* Number of rounds: 10

A brute-force attack tries possible keys until the correct key is found. AES-128 has 2^128 possible keys, making direct brute-force searching impractical.

---

# 2. AES State

AES represents a 16-byte block as a 4 × 4 matrix of bytes called the State.

The State is modified during every AES round.

The conversion between bytes and the State matrix was implemented in Python.

Python implementation:

* `matrix.py`

---

# 3. Key Expansion

The original AES-128 key is expanded into 11 round keys.

They are:

```text
K0, K1, K2, ... K10
```

The initial key is K0 and each AES round uses a different round key.

Key expansion uses operations such as:

* RotWord
* SubWord using the S-box
* Rcon
* XOR

The expanded round keys are required for AES encryption and decryption.

Python implementation:

* `aes_decrypt.py`

---

# 4. AES Operations

AES encryption uses four main operations.

## Rounds 1 to 9

```text
SubBytes
ShiftRows
MixColumns
AddRoundKey
```

## Final Round - Round 10

```text
SubBytes
ShiftRows
AddRoundKey(K10)
```

MixColumns is skipped in the final round.

## SubBytes

SubBytes performs byte substitution using the AES S-box.

Each byte in the State is replaced with another byte according to the S-box.

Python implementation:

* `sbox.py`

## ShiftRows

ShiftRows changes the positions of bytes in the State matrix.

The rows are shifted by different offsets to provide diffusion across the State.

## MixColumns

MixColumns mixes the bytes within each column of the State.

It provides diffusion by combining the bytes of each column using finite-field arithmetic.

Python implementation:

* `diffusion.py`

## AddRoundKey

AddRoundKey combines the State with the round key using XOR.

```text
State XOR RoundKey
```

XOR is used because it is reversible:

```text
A XOR B XOR B = A
```

Python implementation:

* `add_round_key.py`

---

# 5. AES Round Structure

AES-128 begins with an initial AddRoundKey operation using K0.

Then:

```text
Initial:
AddRoundKey(K0)

Rounds 1-9:
SubBytes
ShiftRows
MixColumns
AddRoundKey(K1 ... K9)

Final Round 10:
SubBytes
ShiftRows
AddRoundKey(K10)
```

The final round does not contain MixColumns.

AES-128 therefore uses:

* 1 initial AddRoundKey
* 9 complete rounds
* 1 final round

This gives a total of 10 AES rounds.

---

# 6. AES Decryption

AES decryption reverses the encryption process.

The round keys are used in reverse order:

```text
K10 → K9 → K8 → ... → K1 → K0
```

The inverse operations are used:

* InvSubBytes
* InvShiftRows
* InvMixColumns
* AddRoundKey

AddRoundKey is used again because XOR is its own inverse.

---

# 7. Decryption Order Implemented

Starting with the ciphertext:

```text
Ciphertext
```

↓

```text
AddRoundKey(K10)
```

↓

```text
InvShiftRows
```

↓

```text
InvSubBytes
```

Then for the intermediate rounds:

```text
AddRoundKey(K9)
InvMixColumns
InvShiftRows
InvSubBytes
```

Then:

```text
AddRoundKey(K8)
InvMixColumns
InvShiftRows
InvSubBytes
```

The same process continues until K1.

Finally:

```text
AddRoundKey(K0)
```

This produces the original plaintext.

There is no InvMixColumns in the final step.

Python implementation:

* `aes_decrypt.py`

---

# 8. Python Implementation

The AES implementation was written using Python.

Important functions implemented or used:

```text
bytes2matrix()
matrix2text()
add_round_key()
sub_bytes()
inv_shift_rows()
mix_single_column()
mix_columns()
inv_mix_columns()
expand_key()
decrypt()
```

The State was represented as a 4 × 4 matrix and the AES transformations were applied to it.

The main Python files are:

* `matrix.py`
* `add_round_key.py`
* `sbox.py`
* `diffusion.py`
* `aes_decrypt.py`

---

# 9. What I Understood

* AES is a symmetric block cipher.
* AES-128 uses a 128-bit key and a 128-bit block.
* AES-128 operates on a 4 × 4 matrix of bytes called the State.
* AES-128 has 10 rounds.
* AES-128 uses 11 round keys: K0 to K10.
* Key Expansion generates the round keys.
* SubBytes performs byte substitution using the S-box.
* ShiftRows changes the positions of bytes in the State.
* MixColumns provides diffusion by mixing bytes within each column.
* AddRoundKey XORs the State with the round key.
* The final encryption round skips MixColumns.
* AES decryption uses the round keys in reverse order.
* Decryption uses inverse AES transformations.
* XOR is reversible and therefore can be used for both encryption and decryption in AddRoundKey.
* I implemented important AES operations in Python.
* I used the implemented operations to understand and perform AES decryption.

---

# 10. Files

The Python implementations for this section are stored in this folder:

```text
AES/
├── aes.md
├── matrix.py
├── add_round_key.py
├── sbox.py
├── diffusion.py
└── aes_decrypt.py
```

The files contain the implementations used while completing the AES challenges.

---

# 11. Challenges Completed

The following CryptoHack AES challenges were completed:

- Keyed Permutations
- Resisting Bruteforce
- Structure of AES
- Round Keys
- Confusion through Substitution
- Diffusion through Permutation
- Bringing It All Together

These challenges helped me understand the basic structure of AES, the AES State, key expansion, round keys, substitution, diffusion, permutation, and AES encryption and decryption.

---

# 12. Flags

| Challenge | Flag |
|---|---|
| Keyed Permutations | `crypto{bijection}` |
| Resisting Bruteforce | `crypto{biclique}` |
| Structure of AES | `crypto{inmatrix}` |
| Round Keys | `crypto{r0undk3y}` |
| Confusion through Substitution | `crypto{l1n34rly}` |
| Diffusion through Permutation | `crypto{d1ffUs3R}` |
| Bringing It All Together | `crypto{MYAES128}` |

---

# 13. Summary

This task introduced the internal structure of AES and how its main transformations work.

I learned about:

- Symmetric encryption
- AES-128
- AES State
- AES rounds
- Round keys
- Key Expansion
- S-box
- SubBytes
- ShiftRows
- MixColumns
- AddRoundKey
- Permutation
- AES encryption structure
- AES decryption structure
- Inverse AES operations
- Python implementation of AES operations

I also completed the seven CryptoHack AES challenges and stored the Python implementations in the AES folder.
****
