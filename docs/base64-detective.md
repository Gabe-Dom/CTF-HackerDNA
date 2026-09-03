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
The lab starts with a web page instructing to "Follow the clues..."

Looking at the page source code reveals clues in form of short strings that seem to be base64 encoded.

- 1st clue is in the `value` attribute of a hidden input field with `id` equal `clue1`. Decoding it with `atob()` returns *'Flag part 1: ...'*
- 2nd clue is the `data-clue` attribute of a `div` element. It decodes to *'Flag part 2: ...'*
- 3rd clue is a comment in the embedded CSS. It decodes to *'Flag part 3: ...'*
- 4th clue is a `const` named `finalClue` in the script. It decoded to the final part of the flag.

Concatenating the four pieces gives us a string, which *almost* matches the expected format of UUID. There are two problems:

1. a hyphen is missing between the 2nd and 3rd segment of UUID
2. there is a character `v` in one place, which is not a hexadecimal digit.

For problem (1), it's enough to just add a missing hyphen, bringing the string to the correct UUID format.

For problem (2), I assumed a typo, so I looked at characters that are located on a standard keyboard next to 'v' and are proper hex digits: 'c', 'b', and 'f'. Then I tried submitting the flag with `v` replaced wit one of these characters. It turned out that 'f' did the trick. 

## Enumeration and Exploitation
No enumeration or exploitation was really needed, everything was in the source of the publicly available page.

## Recommended mitigation
For real secrets use strong cryptographic mechanism intended for your particular use case. Secrets protected only by encoding and/or security by obscurity are not secure.
