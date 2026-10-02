# Rail Fence

| Challenge:     | Rail Fence                            |
| -------------- | ------------------------------------- |
| **Platform**:  | HackerDNA                             |
| **Lab URL:**   | https://hackerdna.com/labs/rail-fence |
| **Category:**  | Cryptography                          |
| **Objective:** | Decrypt a Rail Fence ciphertext       |
| **Author:**    | Gabriel Dom                           |

---
## Reconnaissance

The lab starts with an encrypted message and the following hints:
- the plaintext is a UUID
- the plaintext starts with `3` and ends with `d`
- the cipher used is Rail Fence

The goal is to decrypt the message.

## Enumeration

The encrypted message is `3bd4af06-c958-c418-1528-4e0df7801e02`.

We can see that the hyphens are in the correct positions, so we can assume that only the hexadecimal digits were encrypted and the other characters were ignored. This means that the actual ciphertext is `3bd4af06c958c41815284e0df7801e02`, and its length is `L = 32`.

The number of rails `N` must be greater than `1` and less than `L = 32`, because otherwise the ciphertext would be equal to the plaintext. We know they differ because the ciphertext does not end with `d`.

The letter `d` appears twice in the ciphertext, at positions 2 and 23 (0-based).

It is impossible to determine the number of rails (which is the cipher key) based solely on the information above, but the range of possible values is small enough for a brute-force approach of trying them all.

## Exploitation

Using CyberChef to try possible values of the key, we find that the keys that result in the plaintext ending with `d` are 3 and 17. Both can be used to reconstruct two different flags, both satisfying the criteria specified in the exercise. However, only one of them is accepted as the correct answer by the lab.

## Recommended mitigation

Use modern encryption algorithms. Classic algorithms like Rail Fence do not provide real security, especially when the attacker has some knowledge about the plaintext.
