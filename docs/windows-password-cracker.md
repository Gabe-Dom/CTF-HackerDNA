# Windows Password Cracker

| Challenge:     | Windows Password Cracker           |
| -------------- | ---------------------------------- |
| **Platform**:  | HackerDNA                          |
| **Lab URL:**   | https://hackerdna.com/labs/windows-password-cracker |
| **Category:**  | Cryptography                       |
| **Objective:** | Crack the NTLM hash                |
| **Author:**    | Gabriel Dom                        |

---
## Reconnaissance

The scenario of this exercise starts at the moment we gained access to a Windows system and extracted the Security Account Manager (SAM) database with password hashes for user accounts. Our objective is to crack the NTLM hash for the `secretuser` account.

Our goal is to recover the password for one user only, so let's extract the corresponding line from the downloaded file:
```cat sam_hashes.txt | grep '^secretuser:' > secretuser.txt```

## Enumeration

The `secretuser.txt` file is already in the PWDUMP format, so there is no need to run any additional tool to prepare it for `john`. We can examine the file directly with:
```
john --show=formats ./secretuser.txt | jq
```
John detects two hashes: LM (Legacy LAN Manager format) and NT (newer Windows NT hash):
```...
{
    "label": "LM",
    "prepareEqCiphertext": true,
    "canonHash": [
        "$LM$aad3b435b51404ee"
    ]
 },
 {
    "label": "NT",
    "canonHash": [
        "$NT$5fc5d15aac645ea16c1d363ab539724e"
    ]
},...
```
However, the `aad3b435b51404ee` value is well known - it is actually the hash of an empty password. This value is used as a placeholder for "Windows has no stored LM hash for this account." It means the interesting hash is the NT hash.


## Exploitation
With this knowledge gathered, we can recover the password with:

```
john --format=nt-opencl --wordlist=rockyou.txt ./secretuser.txt 
john --format=nt-opencl --show ./secretuser.txt 
```

John finds the password, and we achieve the objective.

## Recommended mitigation

- Always use strong, unique passwords. Make sure they are not present in popular leak lists. Even if the password is stored using a modern scheme, if it is easy to guess, it provides weak security.

