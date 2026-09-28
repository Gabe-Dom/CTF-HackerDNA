# Shadow Cracker

| Challenge:     | Shadow Cracker                            |
| -------------- | ----------------------------------------- |
| **Platform**:  | HackerDNA                                 |
| **Lab URL:**   | https://hackerdna.com/labs/shadow-cracker |
| **Category:**  | Cryptography                              |
| **Objective:** | Recover credentials for a shadow file     |
| **Author:**    | Gabriel Dom                               |

---
## Reconnaissance

The lab provides a Linux `shadow` file and instructions to recover credentials. 

## Enumeration

Examining the file, we can see that only one line, for the user `admin`, actually contains credentials. Let's extract this line to a separate `shadow.admin` file.

The credentials field is in MCF format (also called `crypt(3)` format). It starts with `$6$`, which indicates that SHA-512 is used as the hashing algorithm. Let's identify the hash format with `john`:

```
john --show=formats shadow.admin | jq
```
John confirms that it is `sha512crypt`, for which it has GPU and CPU based implementations.

## Exploitation

We can now crack the password using GPU-based implementation and the popular `rockyou.txt` wordlist:

```
john --format=sha512crypt-opencl --wordlist=rockyou.txt shadow.admin 
john --format=sha512crypt-opencl --show shadow.admin 
```
John quickly finds the password because it appears near the top of the `rockyou.txt` wordlist.

## Recommended mitigation
- Always use strong, unique passwords. Even if the password is stored using a modern, secure scheme such as Argon2, if it is easy to guess, it provides weak security.

