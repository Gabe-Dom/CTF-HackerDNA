# PDF Password Cracker

| Challenge:     | PDF Password Cracker                            |
| -------------- | ----------------------------------------------- |
| **Platform**:  | HackerDNA                                       |
| **Lab URL:**   | https://hackerdna.com/labs/pdf-password-cracker |
| **Category:**  | Cryptography                                    |
| **Objective:** | Extract the flag from a password-protected PDF  |
| **Author:**    | Gabriel Dom                                     |

---
## Reconnaissance

The lab starts with a web page offering a PDF file to download, along with instructions to crack the password and extract the flag.

## Enumeration

Having downloaded the `secure_document.pdf`, we can take an initial look with `exiftool`.

```
> exiftool secure_document.pdf 
...
MIME Type    : application/pdf
PDF Version  : 1.4
Linearized   : No
Encryption   : Standard V2.3 (128-bit)
User Access  : Print, Modify, Copy, Annotate, Fill forms, Extract, Assemble, Print high-res
Warning      : Document is password protected (use Password option)
```
According to `exiftool` this indeed is a password protected PDF.

## Exploitation
Let’s check if the password can be cracked with `john` using a popular wordlist.
```
pdf2john.py secure_document.pdf >pdf.john
john --wordlist=rockyou.txt pdf.john
```

John quickly finds the password. The document can then be opened with the password to reveal the flag.

## Recommended mitigation

- Do not rely on a shared PDF password as the sole protection for confidential material.  Deliver such material through authenticated, authorization-controlled access.
- Use a strong, unique password only as supplementary protection, distributed through a separate authorized channel where necessary.