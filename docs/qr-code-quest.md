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

The challenge begins with a web page that contains an image with a QR code. Downloading the QR code and decoding it with `qrca` returns a sequence of hexadecimal digits. This indicates that the data is hex-encoded. It can be converted back to its original binary form with `xxd -r -p`. Once decoded, the flag is revealed.

## Recommended mitigation

For real secrets, use a strong cryptographic mechanism that matches your specific use case. Encoding alone, including QR code encoding, does not provide meaningful protection.
