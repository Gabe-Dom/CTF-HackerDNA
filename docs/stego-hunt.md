# Stego Hunt

| Challenge:     | Stego Hunt |
| -------------- | --------------------------------------|
| **Platform**:  | HackerDNA                             |
| **Lab URL:**   | https://hackerdna.com/labs/stego-hunt |
| **Category:**  | Digital Forensics & IR                |
| **Objective:** | Find the hidden flag                  |
| **Author:**    | Gabriel Dom                           |

---
## Reconnaissance
The lab starts with a web page. The heading reads "Stego Hunt," and the text below it reads, "Can you find the flag?" There is a dark rectangle below the text. Inspecting the page with Developer Tools shows that the dark rectangle is an image named `hidden.png`.

We can save the image to the local filesystem and examine it with `exiftool`. This reveals the flag in the `Comment` tag.

## Enumeration and Exploitation
No enumeration or exploitation was needed because the flag was hidden in the publicly available image.

## Recommended mitigation
Do not store secrets in metadata unless they are additionally protected, for example with strong encryption.

