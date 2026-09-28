# Office Password Cracker

| Challenge:     | Office Password Cracker                            |
| -------------- | -------------------------------------------------- |
| **Platform**:  | HackerDNA                                          |
| **Lab URL:**   | https://hackerdna.com/labs/office-password-cracker |
| **Category:**  | Cryptography                                       |
| **Objective:** | Find a flag in a password-protected MS Office file |
| **Author:**    | Gabriel Dom                                        |

---
## Reconnaissance
The lab provides a password-protected Microsoft Office file `confidential_report.docx` and instructions to find a flag hidden inside.

## Enumeration

Let's examine the file with `john`:

```
office2john confidential_report.docx > report.john 
john --show=formats report.john 
```

As expected, `john` recognizes the hash as the `office` format, for which it has GPU and CPU-based implementations.

## Exploitation

Let's crack it now:
```
john --format=office-opencl --wordlist=rockyou.txt report.john 
john --format=office-opencl --show report.john 
```

Using the recovered password, we can now open the document and find the flag.

## Recommended mitigation

- Do not rely on a password as the sole protection for confidential material. Limit distribution of such material to an authenticated channel with authorization controls.
- Use a strong, unique password, not present in popular leak files.