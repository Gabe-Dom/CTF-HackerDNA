# Crack SHA1 Hash

| Challenge:     | Crack SHA1 Hash                            |
| -------------- | ------------------------------------------ |
| **Platform**:  | HackerDNA                                  |
| **Lab URL:**   | https://hackerdna.com/labs/crack-sha1-hash |
| **Category:**  | Cryptography                               |
| **Objective:** | Submit the cracked plaintext               |
| **Author:**    | Gabriel Dom                                |

---
## Reconnaissance
The exercise gives a single SHA1 hash and information that the  plaintext is a common English word in the following format: `s******y`. Before applying any brute-force guessing, it's worth to look at the given clues and try the most obvious word, taking into account the general domain of the HackerDNA platform.

It turn out the most obvious word was the correct answer, no SHA1 guessing was needed.

## Enumeration and Exploitation
No enumeration or exploitation was really needed, everything was in the given clues.

## Recommended mitigation
Use non-obvious passwords. With enough clues about the password even the strongest hashing schema will  not help. 


