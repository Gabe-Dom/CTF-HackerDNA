# XOR Reuse

| Challenge:     | XOR Reuse                          |
| -------------- | ---------------------------------- |
| **Platform**:  | HackerDNA                          |
| **Lab URL:**   | https://hackerdna.com/labs/xor-reuse |
| **Category:**  | Cryptography                       |
| **Objective:** | Find the hidden flag               |
| **Author:**    | Gabriel Dom                        |

---
## Reconnaissance

The challenge provides a secret message and informs that it was encrypted using repeating-key XOR and the result was hex-encoded. It also gives a hint that the key is just a few bytes long and it repeats over and over across the plaintext.

We also know the encrypted message is the flag and it fits the following format:

```
5_______-____-____-____-____________b
```

This is enough information to try solving this challenge with cryptanalysis alone and no brute forcing.

Let's add the ciphertext and plaintext to our target format as hex digits for easier analysis.
```
   5  __ __ __ __ __ __ __ -  __ __ __ __ -  __ __ __ __ -  __ __ __ __ -  __ __ __ __ __ __ __ __ __ __ __ b
c: 0f 4d 27 d4 0d 1d 70 da 17 1e 24 81 5c 51 21 d1 58 48 38 da 58 48 24 ce 5c 4b 24 85 59 49 74 d7 03 4d 74 81 
p: 35 __ __ __ __ __ __ __ 2d __ __ __ __ 2d __ __ __ __ 2d __ __ __ __ 2d __ __ __ __ __ __ __ __ __ __ __ 62
```

Because XOR is self-inverse, for any given position in the message, if we know the values of both the ciphertext `c` and plaintext `p`, we can calculate the repeated key value for that position: `k = c xor p`.

In our case, we have 6 positions for which we know the plaintext, so let's calculate the key values for them:
```
   5  __ __ __ __ __ __ __ -  __ __ __ __ -  __ __ __ __ -  __ __ __ __ -  __ __ __ __ __ __ __ __ __ __ __ b
c: 0f 4d 27 d4 0d 1d 70 da 17 1e 24 81 5c 51 21 d1 58 48 38 da 58 48 24 ce 5c 4b 24 85 59 49 74 d7 03 4d 74 81 
p: 35 __ __ __ __ __ __ __ 2d __ __ __ __ 2d __ __ __ __ 2d __ __ __ __ 2d __ __ __ __ __ __ __ __ __ __ __ 62
k: 3a __ __ __ __ __ __ __ 3a __ __ __ __ 7c __ __ __ __ 15 __ __ __ __ e3 __ __ __ __ __ __ __ __ __ __ __ e3
```

We can see that `3a` appears twice in the repeated key, and the distance between occurrences is 8. The hint said the key is short, so we may continue with the hypothesis that the key is no longer than 8 bytes. This means that repeated occurrences of `3a` are due to repetition of the whole key, not because `3a` appears more than once in the key. So let's put `3a` in every 8th position in the key line.

We can do the same with `e3` which occurs every 12 characters.
```
   5  __ __ __ __ __ __ __ -  __ __ b  __ -  __ __ b  __ -  __ __ __ __ -  f  __ __ __ __ __ __ __ 9  __ __ b
c: 0f 4d 27 d4 0d 1d 70 da 17 1e 24 81 5c 51 21 d1 58 48 38 da 58 48 24 ce 5c 4b 24 85 59 49 74 d7 03 4d 74 81 
p: 35 __ __ __ __ __ __ __ 2d __ __ 62 __ 2d __ __ 62 __ 2d __ __ __ __ 2d 66 __ __ __ __ __ __ __ 39 __ __ 62
k: 3a __ __ __ __ __ __ __ 3a __ __ e3 __ 7c __ __ 3a __ 15 __ __ __ __ e3 3a __ __ __ __ __ __ __ 3a __ __ e3
```
Continuing the assumption that the key is not longer then 8 bytes, and taking into account that:
- `3a` occurs at least every 8 characters
- `e3` occurs at least every 12 characters
the key length would be at most 4 characters because 4 is the greatest common divisor (GCD) of 8 and 12.

Let's add `3a` and `e3` in the missing places, so they repeat every 4 positions:
```
   5  __ __ 7  7  __ __ 9  -  __ __ b  f  -  __ 2  b  __ -  9  b  __ __ -  f  __ __ f  c  __ __ 4  9  __ __ 5
c: 0f 4d 27 d4 0d 1d 70 da 17 1e 24 81 5c 51 21 d1 58 48 38 da 58 48 24 ce 5c 4b 24 85 59 49 74 d7 03 4d 74 81 
p: 35 __ __ 37 37 __ __ 39 2d __ __ 62 66 2d __ 32 62 __ 2d 39 62 __ __ 2d 66 __ __ 66 63 __ __ 34 39 __ __ 62
k: 3a __ __ e3 3a __ __ e3 3a __ __ e3 3a 7c __ e3 3a __ 15 e3 3a __ __ e3 3a __ __ e3 3a __ __ e3 3a __ __ e3
```
   
In the first step we discovered that the key also contains `7c` and `15`, which so far appeared only once each. Continuing with the hypothesis of a key length equal to 4, let's repeat these characters every 4th place as well.  

```
c: 0f 4d 27 d4 0d 1d 70 da 17 1e 24 81 5c 51 21 d1 58 48 38 da 58 48 24 ce 5c 4b 24 85 59 49 74 d7 03 4d 74 81 
p: 35 31 32 37 37 61 65 39 2d 62 31 62 66 2d 34 32 62 34 2d 39 62 34 31 2d 66 37 31 66 63 35 61 34 39 31 61 62
k: 3a 7c 15 e3 3a 7c 15 e3 3a 7c 15 e3 3a 7c 15 e3 3a 7c 15 e3 3a 7c 15 e3 3a 7c 15 e3 3a 7c 15 e3 3a 7c 15 e3
```
This completes our repeated key line and reveals the flag.

## Recommended mitigation

Do not use XOR for encryption. As demonstrated above, even the slightest hint about the plaintext allows us to decode the ciphertext completely manually, with no tools except a XOR calculator, and even that can be done manually. Use real cryptographic algorithms for anything that needs to remain secret.
