# Binary Secrets

| Challenge:     | Binary Secrets                            |
| -------------- | ----------------------------------------- |
| **Platform**:  | HackerDNA                                 |
| **Lab URL:**   | https://hackerdna.com/labs/binary-secrets |
| **Category:**  | Reverse Engineering & Binary Exploitation |
| **Objective:** | Find the flag hidden in a binary file     |
| **Author:**    | Gabriel Dom                               |

---
## Reconnaissance

The challenge starts with a web page offering a binary file to download and instructions to extract the hidden flag.

Downloading the file and examining it with `xxd` shows that this is a binary file with pieces of text hidden inside. One of them looks like the UUID format, which is the expected format of the flag. We can extract the text pieces with `strings secret_binary.bin`. This reveals the flag in a ready-to-use plain-text format.

## Recommended mitigation

Never hardcode secrets in binary files - they are easy to extract. This includes hardcoding secrets in the source code,because they transferred to the binary file during compilation. 