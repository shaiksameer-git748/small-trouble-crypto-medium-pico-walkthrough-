# small-trouble-crypto-medium-pico-walkthrough-
# 🔐 picoCTF - Small d (RSA) Writeup

## Challenge Information

| Category | Cryptography |
|----------|--------------|
| Algorithm | RSA |
| Difficulty | Medium |
| Attack | Small Private Exponent Attack (Wiener's Attack) |

---

# Challenge Description

The challenge provided three values:

```python
n = ...
e = ...
c = ...
```

along with the following source code.

```python
p = getPrime(1048)
q = getPrime(1048)

n = p*q
phi = (p-1)*(q-1)

d = getPrime(256)

e = inverse(d, phi)

c = pow(m, e, n)
```

The objective was to recover the original plaintext.

---

# Initial Analysis

Immediately I recognized

```
n
e
c
```

as an RSA cryptosystem.

However, the private exponent **d** was missing.

Without **d**, RSA decryption is impossible.

The first goal therefore became:

> Recover the private key.

---

# Step 1 — Analyze the Source Code

Instead of immediately searching for a decryption script, I carefully read the implementation.

The following line immediately stood out.

```python
d = getPrime(256)
```

Normally RSA is generated as

```python
e = 65537
d = inverse(e, phi)
```

Instead, the developer intentionally generated a **256-bit private exponent**.

This is highly unusual.

---

# Step 2 — Check FactorDB

Whenever I encounter RSA challenges I first verify whether the modulus is factorable.

Using

https://factordb.com

I searched for

```
n
```

Result

```
Composite
```

No factors were available.

Therefore normal RSA recovery using

```
p
q
φ(n)
```

was not possible.

---

# Step 3 — Form a Hypothesis

At this point I had:

✅ Large modulus

❌ Not factorable

✅ Small private exponent

This suggested the challenge was not about factoring.

It was about attacking a **small private exponent**.

---

# Step 4 — Read the Hint

The official hint stated:

> "This might be a job for Boneh-Durfee."

Initially I investigated Boneh–Durfee because the hint suggested a small-`d` attack.

While researching, I also studied Wiener's Attack to understand the relationship between small private exponents and RSA vulnerabilities.

After testing the challenge parameters, the standard Wiener's Attack implementation successfully recovered the private exponent for this instance.

This demonstrated an important lesson:

**Always verify which attack actually matches the challenge rather than assuming from the hint alone.**

---

# Step 5 — Recover the Private Key

After selecting the appropriate attack, I used a standard implementation of Wiener's Attack.

The recovered value was

```
d = ...
```

The ciphertext could now be decrypted using

```python
m = pow(c, d, n)
```

Finally

```python
long_to_bytes(m)
```

converted the plaintext back into the original flag.

---

# Solver

```python
from Crypto.Util.number import long_to_bytes

# Recover d using Wiener's Attack

m = pow(c, d, n)

print(long_to_bytes(m))
```

(The full Wiener's Attack implementation is omitted here for brevity.)

---

# Attack Flow

```
Challenge
      │
      ▼
Receive n,e,c
      │
      ▼
Identify RSA
      │
      ▼
Read Source Code
      │
      ▼
Notice Small d
      │
      ▼
Check FactorDB
      │
      ▼
Not Factorable
      │
      ▼
Research Small-d Attacks
      │
      ▼
Apply Wiener's Attack
      │
      ▼
Recover d
      │
      ▼
Decrypt Ciphertext
      │
      ▼
Recover Flag
```

---

# Problems Faced

This challenge took approximately **2 hours** to solve.

The biggest challenges were not the mathematics but the investigation process.

## Challenge 1

Understanding why

```python
d = getPrime(256)
```

was insecure.

---

## Challenge 2

Learning the difference between

- Normal RSA
- Wiener's Attack
- Boneh–Durfee

---

## Challenge 3

Determining whether the modulus should be factored or whether another attack was intended.

---

## Challenge 4

Setting up the environment.

During the process I encountered

- Missing Python packages
- Minimal Kali installation
- Python package management issues
- Multiple attack implementations

---

## Challenge 5

Reading several papers, GitHub repositories, and walkthroughs before selecting the correct approach.

---

# Mistakes I Made

## Mistake 1

Initially searching for a decryption script.

Instead I should first identify the vulnerability.

---

## Mistake 2

Assuming every RSA challenge is solved using FactorDB.

This challenge proved otherwise.

---

## Mistake 3

Trying attacks before fully understanding the implementation.

Reading the source code first would have saved time.

---

# What I Learned

This challenge fundamentally changed how I approach RSA CTFs.

Instead of asking

> "Which script decrypts RSA?"

I now ask

> "What weakness exists in this RSA implementation?"

That single question determines the correct attack.

---

# My RSA Methodology

```
Receive Challenge
        │
        ▼
Identify RSA
        │
        ▼
Read Source Code
        │
        ▼
Understand Implementation
        │
        ▼
Check FactorDB
        │
 ┌──────┴────────┐
 │               │
 ▼               ▼
Factored      Not Factored
 │               │
 ▼               ▼
Normal RSA   Analyze Weakness
                     │
                     ▼
           Small d ?
                     │
                     ▼
          Wiener / Boneh–Durfee
                     │
                     ▼
               Recover d
                     │
                     ▼
                 Decrypt
                     │
                     ▼
                    Flag
```

---

# Key Takeaways

- Always identify the cryptographic algorithm first.
- Read the source code before writing scripts.
- FactorDB should be one of the first checks for RSA challenges.
- Not every RSA challenge is solved by factoring.
- Understanding the implementation is more important than memorizing scripts.
- Choosing the correct attack is the most important part of solving cryptographic CTFs.

---

# References

- RSA Cryptosystem
- Wiener's Attack
- Boneh–Durfee Attack
- FactorDB
- Crypto.Util.number (PyCryptodome)

---

# Final Thoughts

This challenge was one of the most educational RSA challenges I have solved.

Although I relied on existing implementations of the attack—as many CTF players do—the real learning came from understanding **why** the attack worked and **how** to identify it from the source code.

The most valuable lesson was that successful cryptanalysis begins with analyzing the implementation, not with searching for scripts.

> **"Understand the vulnerability first. The correct attack comes afterwards."**
