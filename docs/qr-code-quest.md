# QR Code Quest

| Challenge:     | QR Code Quest |
| -------------- | ---------------------------------------- |
| **Platform**:  | HackerDNA                                |
| **Lab URL:**   | https://hackerdna.com/labs/qr-code-quest |
| **Category:**  | Digital Forensics & IR                   |
| **Objective:** | Analyze a QR code and extract the flag.  |
| **Author:**    | Gabriel Dom                              |

---
## Reconnaissance

Decoding the QR code with `qrca` returns a string of hexadecimal digits. This suggests the data is hex-encoded. It can be decoded back to its original binary form with `xxd -r -p`. Once decoded, the flag is revealed.

## Recommended mitigation

For real secrets, use a strong cryptographic mechanism that matches your specific use case. Encoding alone, including QR codes, does not provide meaningful protection.
