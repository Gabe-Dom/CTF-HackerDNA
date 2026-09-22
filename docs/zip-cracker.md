# ZIP Cracker

| Challenge:     | ZIP Cracker                        |
| -------------- | ---------------------------------- |
| **Platform**:  | HackerDNA                          |
| **Lab URL:**   | https://hackerdna.com/labs/zip-cracker |
| **Category:**  | Cryptography                       |
| **Objective:** | Crack the password protected zip   |
| **Author:**    | Gabriel Dom                        |

---
## Reconnaissance

The lab starts with a page offering a password protected archive to download. We save it locally as `secret_archive.zip`.

Let's examine it:
```
$ zipinfo secret_archive.zip
Archive:  secret_archive.zip
Zip file size: 231 bytes, number of entries: 1
-rw-r--r--  3.0 unx       37 TX stor 25-Aug-17 20:52 flag.txt
1 file, 37 bytes uncompressed, 37 bytes compressed:  0.0%
```

Extract the information from the archive that `john` needs to verify the password during guessing:
```
> zip2john secret_archive.zip > archive.john
ver 1.0 efh 5455 efh 7875 secret_archive.zip/flag.txt PKZIP Encr: 2b chk, TS_chk, cmplen=49, decmplen=37, crc=AB3917CF
```
The output tells us that the archive uses weak, obsolete PKZIP encryption.

Let's run the tool:
```
john --wordlist=rockyou.txt archive.john 
john --show archive.john
```

Now we can extract the file from the archive and read the flag.

## Recommended mitigation

- Do not use traditional PKZIP encryption, also known as ZipCrypto. Modern ZIP implementations use AES-256 encryption.
- Do not use well-known passwords that appear in popular leak lists, even if they are relatively long.
