# KeePass Breaker

| Challenge:     | KeePass Breaker                            |
| -------------- | ------------------------------------------ |
| **Platform**:  | HackerDNA                                  |
| **Lab URL:**   | https://hackerdna.com/labs/keepass-breaker |
| **Category:**  | Cryptography                               |
| **Objective:** | Extract the flag from a KeePass 4 database |
| **Author:**    | Gabriel Dom                                |

---
## Reconnaissance

The lab offers a KeePass 4 database to download, along with instructions to retrieve a flag from it.

## Enumeration

First, we prepare the file for processing with `john`:
```
keepass2john challenge_vault.kdbx > challenge_vault.john
```

Now we can examine the format of the hash:
```
john --show=formats challenge_vault.john | jq 
```
John recognized the hash as "KeePas-Argon2".

## Exploitation

Let’s crack the password with `john` using OpenCL implementation of KeePas Argon2:
```
john --format=KeePass-Argon2-opencl --wordlist=rockyou.txt challenge_vault.john 
```
John found the password, so we can now access and enumerate the database:

```
> keepassxc-cli ls challenge_vault.kdbx
Enter password to unlock challenge_vault.kdbx: 
Confidential Documents/

> keepassxc-cli ls challenge_vault.kdbx "Confidential Documents"
Enter password to unlock challenge_vault.kdbx: 
Server Admin Access
Database Connection
Flag
Backup System

> keepassxc-cli show -s challenge_vault.kdbx "Confidential Documents/Flag"
```
The final command reveals the flag.

## Recommended mitigation
- Always use strong, unique passwords. Even if the password is stored using a modern, secure scheme such as Argon2, if it is easy to guess, it provides weak security.


