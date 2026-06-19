# CryptoHack Writeup

## June 8, 2026

### Symmetric Starters

#### Modes of Operation Starter

the server had two endpoints, encrypt_flag() which encrypts the flag and decrypt() which decrypts anything i give it using ECB mode

i called encrypt_flag() and got the ciphertext back, then i just passed that same ciphertext into decrypt() and it gave me the plaintext hex, then i decoded it using the hex decoder on the page

the reason this worked is ECB mode has no IV and encrypts each block independently, and the server let me decrypt anything without restriction so i just decrypted the flag directly

Flag: `crypto{bl0ck_c1ph3r5_4r3_f457_!}`

#### Passwords as Keys

the server was using a random english dictionary word, hashing it with MD5, and using that hash as the AES key to encrypt the flag

since the wordlist was known and had about 99000 words i wrote a python script to try every word, hash it with MD5 and send to the server's decrypt endpoint, when the decrypted output started with the hex for crypto{ i knew i found the right word which turned out to be bluebell

```python
import hashlib, requests
from concurrent.futures import ThreadPoolExecutor

words = requests.get("https://gist.githubusercontent.com/wchargin/8927565/raw/d9783627c731268fb2935a731a618aa8e95cf465/words").text.splitlines()
ciphertext = "c92b7734070205bdf6c0087a751466ec13ae15e6f1bcdd3f3a535ec0f4bbae66"
found = []

def check_word(word):
    key_hash = hashlib.md5(word.encode()).hexdigest()
    r = requests.get(f"https://aes.cryptohack.org/passwords_as_keys/decrypt/{ciphertext}/{key_hash}/")
    if r.status_code == 200:
        result = r.json()
        if "plaintext" in result and result["plaintext"].startswith("63727970746f7b"):
            found.append((word, result["plaintext"]))
            print("FOUND:", word)

with ThreadPoolExecutor(max_workers=50) as executor:
    executor.map(check_word, words)
```

the lesson here is never use a dictionary word as an encryption key, always use a proper random key generator

Flag: `crypto{k3y5__r__n07__p455w0rdz?}`

---

### Block Ciphers 1

#### ECB CBC WTF

the server encrypted the flag in CBC mode but only allowed decryption in ECB mode, the challenge title hints at the weakness

CBC decryption works like this: plaintext_block[i] = AES_decrypt(ciphertext[i]) XOR ciphertext[i-1], and for the first block it XORs with the IV instead

since i could only ECB-decrypt, i got AES_decrypt(ciphertext) without the XOR step, so i just did the XOR manually using python

i called encrypt_flag() to get IV + ciphertext, split them, passed the ciphertext into decrypt(), then XORed the result with IV for block 1 and with the first ciphertext block for block 2

```python
ciphertext_full = "b111a0206ef06305d6e25b03d9b7854a5d93f8744fc3f828918e36bd092fbaee2c763ff63781b5f0c85140cf9c7f12a3"
plaintext = "d263d9501a9f1836b5800436acd4ee7f02a78e447ea7a719a6d1179c280e9b93"

iv = ciphertext_full[0:32]
c1 = ciphertext_full[32:64]
p1 = plaintext[0:32]
p2 = plaintext[32:64]

def xor_hex(a, b):
    return bytes(x ^ y for x, y in zip(bytes.fromhex(a), bytes.fromhex(b))).hex()

flag_hex = xor_hex(p1, iv) + xor_hex(p2, c1)
print(bytes.fromhex(flag_hex).decode())
```

Flag: `crypto{3cb_5uck5_4v01d_17_!!!!!}`

#### ECB Oracle

the server prepended my input to the secret flag and encrypted everything together with ECB, i had no decrypt function this time

the weakness of ECB is identical plaintext blocks always produce identical ciphertext blocks, so i could recover the flag one byte at a time by controlling how many padding bytes i sent before the unknown flag byte

i padded my input so exactly one unknown flag byte fell at the end of a block, then tried all 256 possible byte values in that position and compared ciphertext blocks, when they matched i found that byte, then repeated for every byte

wrote a python script to automate this, it was slow because each byte needed up to 256 server requests but eventually recovered the full flag

```python
import requests, time

URL = "https://aes.cryptohack.org/ecb_oracle/encrypt/{}/"

def encrypt(hex_plaintext):
    for attempt in range(10):
        try:
            r = requests.get(URL.format(hex_plaintext), timeout=15)
            return r.json()["ciphertext"]
        except:
            time.sleep(1)
    raise Exception("Failed")

block_size = 16
known = b""

for i in range(80):
    pad_amount = (2 * block_size) - 1 - (i % block_size)
    padding = "00" * pad_amount
    block_num = (i // block_size) + 2
    target = encrypt(padding)[: block_num * block_size * 2]

    for b in range(256):
        guess = bytes.fromhex(padding) + known + bytes([b])
        if encrypt(guess.hex())[: block_num * block_size * 2] == target:
            known += bytes([b])
            print("Known so far:", known)
            break

    if b'}' in known:
        break

print("FLAG:", known)
```

Flag: `crypto{p3n6u1n5_h473_3cb}`

#### Flipping Cookie

the server issued cookies encrypted with CBC mode containing `admin=False;expiry=...` and i needed to make it say `admin=True` without knowing the key

in CBC decryption, flipping a bit in ciphertext block N-1 flips the same bit in plaintext block N, since the IV is XORed with block 1 during decryption i could flip bits directly in the IV to change characters in the first plaintext block

`admin=False` was in block 1 with `False` at positions 6 to 10, i XORed positions 6-10 of the IV with the difference between `False` and `True;` to flip those exact characters

```python
iv = "758ee4561e48d0a8afc1dc444d8ec964"
iv_bytes = bytearray(bytes.fromhex(iv))

xor_diff = bytes(a ^ b for a, b in zip(b"False", b"True;"))

for i in range(5):
    iv_bytes[6 + i] ^= xor_diff[i]

print("New IV:", bytes(iv_bytes).hex())
```

sent the modified IV with the original ciphertext to check_admin() and got the flag

Flag: `crypto{4u7h3n71c4710n_15_3553n714l}`

#### Lazy CBC

the developer used the key itself as the IV, `AES.new(KEY, AES.MODE_CBC, KEY)`, which is a serious mistake

when IV equals KEY there is a known attack to recover the key, i sent a 3-block ciphertext where block 1 and block 3 were the same (C1) and block 2 was all zeros

the decryption gave:
- block1_decrypted = AES_decrypt(C1) XOR KEY
- block3_decrypted = AES_decrypt(C1) XOR C2 = AES_decrypt(C1) XOR 0 = AES_decrypt(C1)

so block1_decrypted XOR block3_decrypted = KEY

the server returned an error showing the decrypted hex so i extracted the key directly from that

```python
decrypted = "000000000000000000000000000000007caf7efd32dae25c35b814209c68b8d063344a9bcf600e1b0ffba05f41256c9f"

block1 = decrypted[0:32]
block3 = decrypted[64:96]

key = bytes(a ^ b for a, b in zip(bytes.fromhex(block1), bytes.fromhex(block3)))
print("KEY:", key.hex())
```

used the recovered key in get_flag() to get the flag

Flag: `crypto{50m3_p30pl3_d0n7_7h1nk_IV_15_1mp0r74n7_?}`

#### Triple DES

DES has known weak keys where encrypting a value twice gives back the original plaintext, meaning encrypt acts as its own inverse

the server XORed with a secret IV before and after encryption, but since encrypt(encrypt(x)) = x with a weak key the IVs cancel out and the output equals the input

i found valid 3DES weak keys in python by testing combinations of known DES weak keys, then called encrypt_flag() with the weak key to get the encrypted flag, then called encrypt() again on that ciphertext with the same key to decrypt it

```python
from Crypto.Cipher import DES3

weak_keys = ["0000000000000000","ffffffffffffffff","e0e0e0e0f1f1f1f1","1f1f1f1f0e0e0e0e"]

for k1 in weak_keys:
    for k2 in weak_keys:
        if k1 != k2:
            key = k1 + k2 + k1
            try:
                c = DES3.new(bytes.fromhex(key), DES3.MODE_ECB)
                pt = bytes(8)
                ct = c.encrypt(pt)
                c2 = DES3.new(bytes.fromhex(key), DES3.MODE_ECB)
                if c2.encrypt(ct) == pt:
                    print("Weak key:", key)
            except:
                pass
```

used key `0000000000000000ffffffffffffffff0000000000000000` to get the flag

Flag: `crypto{n0t_4ll_k3ys_4r3_g00d_k3ys}`
