# PDF Trap

| Challenge:     | PDF Trap                            |
| -------------- | ----------------------------------- |
| **Platform**:  | HackerDNA                           |
| **Lab URL:**   | https://hackerdna.com/labs/pdf-trap |
| **Category:**  | Digital Forensics & IR              |
| **Objective:** | Extract the hidden flag from a PDF  |
| **Author:**    | Gabriel Dom                         |

---
## Reconnaissance
The challenge starts with a PDF document to download and instructions to extract the hidden flag.

A safe way to examine a downloaded file is to use `xxd`. In this case, it shows that the first bytes are `%PDF-1.3`, which is consistent with the expected PDF file format. Now we can use the PDF-specific tool `pdfinfo` to examine it more closely:
```
$ pdfinfo challenge.pdf 
Title:           Daily Challenge PDF
Subject:         CTF PDF Forensics
Author:          HDNA Labs
Producer:        PyPDF2
Custom Metadata: yes
Metadata Stream: no
Tagged:          no
UserProperties:  no
Suspects:        no
Form:            none
JavaScript:      no
Pages:           1
Encrypted:       no
Page size:       612 x 792 pts (letter)
Page rot:        0
File size:       584 bytes
Optimized:       no
PDF version:     1.3
```

The interesting finding is:
```
Custom Metadata: yes
```

Let's examine the metadata with `exiftool`. It reveals a custom metadata field named `Custom Flag`, which contains the flag.

## Recommended mitigation

Metadata can contain a surprisingly large amount of information. Always sanitize the metadata of any document you share to remove potentially sensitive information.