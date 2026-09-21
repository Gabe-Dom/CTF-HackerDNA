# Crypto Cipher

| Challenge:     | Crypto Cipher                            |
| -------------- | ---------------------------------------  |
| **Platform**:  | HackerDNA                                |
| **Lab URL:**   | https://hackerdna.com/labs/crypto-cipher |
| **Category:**  | Cryptography                             |
| **Objective:** | Decrypt the flag                         |
| **Author:**    | Gabriel Dom                              |

---
## Reconnaissance

The challenge starts with a simple webpage containing an encrypted flag and a hint saying that the flag is in UUID format and encrypted using a classic cipher.

The encrypted message uses a Base64 alphabet, so let's try decoding it first with `base64 -d`. It reveals a string formatted as a UUID:
`g7699o2s-35of-4s8q-scqw-90y67t7s202s`.

The decoded string is formatted like a UUID but is not a proper UUID because it uses characters that are not hexadecimal digits. Let's examine the alphabet carefully. It consists of the following characters: `cfgoqstwy023456789`. This is 18 characters, while there are only 16 hexadecimal digits. It means the message was not encrypted with any **monoalphabetic substitution cipher** such as Caesar or Atbash. These ciphers produce ciphertext that uses the same number of different characters as the plaintext. In this case, we know the plaintext is a UUID, so it has at most 16 different characters. The cipher is most probably a **polyalphabetic** one.

The most well-known polyalphabetic classic cipher is the Vigenère cipher. However, it requires a key. Let's try the words from the challenge title as keys. In both cases, we get an interesting result: the first letter is decoded into a proper hexadecimal digit, but later letters are not.

It might mean the key is correct, but we need a different method of advancing it. The standard Vigenère cipher encodes only letters, leaving other characters such as digits and hyphens unchanged. It advances the key only when it encodes a character. We want to keep the behavior of changing only letters, because the hyphens and digits are already in the right places for a UUID. So let's try a non-standard Vigenère implementation that advances the key on each character. If you cannot find a ready-to-use implementation, it can be quickly implemented in awk.

This version of the Vigenère cipher, using the first word of the challenge title as the key, decrypts the message to a proper UUID, which turns out to be the correct flag.

## Recommended mitigation

For real security, use one of the modern encryption algorithms, preferably from a well-known implementation you can trust. Implementing your own cryptography is usually a bad idea.

