# Caesar Shift Decoder

| Challenge:     | Caesar Shift Decoder                             |
| -------------- | ------------------------------------------------ |
| **Platform**:  | HackerDNA                                        |
| **Lab URL:**   | https://hackerdna.com/labs/caesar-shift-decoder  |
| **Category:**  | Cryptography                                     |
| **Objective:** | Decode the flag encrypted with the Caesar cipher |
| **Author:**    | Gabriel Dom                                      |

---
## Reconnaissance

The lab starts with a web page that provides a flag encrypted with the Caesar cipher and instructions to decode it.

The ciphertext is in UUID format, but with the wrong alphabet. This is consistent with the way a standard implementation of Caesar cipher works: it keeps non-letter characters untouched and changes only letters. Therefore, numbers and hyphens remain unchanged after encryption.

The alphabet of the ciphertext contains six letters: from h to m. We know the UUID alphabet also contains six letters: from a to f. This means that `a` was shifted to `h`, `b` to `i`, and so on. Each letter was shifted by `7`.

## Exploitation

We can now decrypt the ciphertext using Caesar decryption with key `7` and it reveals the flag.

## Recommended mitigation

Use modern encryption algorithms. Classic algorithms like Caesar do not provide real security, especially when the attacker has some knowledge about the plaintext - in this case, we knew it was a UUID.


