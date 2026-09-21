# Base64 Detective

| Challenge:     | Base64 Detective                            |
| -------------- | ------------------------------------------- |
| **Platform**:  | HackerDNA                                   |
| **Lab URL:**   | https://hackerdna.com/labs/base64-detective |
| **Category:**  | Web & API Security                          |
| **Objective:** | Find the hidden flag                        |
| **Author:**    | Gabriel Dom                                 |

---
## Reconnaissance
The lab starts with a web page instructing the user to "Follow the clues..."

Looking at the page source code reveals clues in the form of short strings that appear to be Base64-encoded.

- The 1st clue is in the `value` attribute of a hidden input field with `id` equal to `clue1`. Decoding it with `atob()` returns *'Flag part 1: ...'*
- The 2nd clue is the `data-clue` attribute of a `div` element. It decodes to *'Flag part 2: ...'*
- The 3rd clue is a comment in the embedded CSS. It decodes to *'Flag part 3: ...'*
- The 4th clue is a `const` named `finalClue` in the script. It decodes to the final part of the flag.

Concatenating the four pieces gives us a string that *almost* matches the expected UUID format. There are two problems:

1. A hyphen is missing between the 2nd and 3rd segments of the UUID.
2. There is a character `v` in one place, which is not a hexadecimal digit.

For problem (1), it's enough to add the missing hyphen, bringing the string into the correct UUID format.

For problem (2), I assumed it was a typo, so I looked at characters located next to `v` on a standard keyboard that are valid hexadecimal digits: `c`, `b`, and `f`. Then I tried submitting the flag with `v` replaced by one of these characters. It turned out that `f` was the correct choice.

## Enumeration and Exploitation
No enumeration or exploitation was really needed, everything was in the source of the publicly available page.

## Recommended mitigation
For real secrets use strong cryptographic mechanism intended for your particular use case. Secrets protected only by encoding and/or security by obscurity are not secure.
