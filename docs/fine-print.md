# Fine Print

| Challenge:     | Fine Print                                |
| -------------- | ----------------------------------------- |
| **Platform**:  | HackerDNA                                 |
| **Lab URL:**   | https://hackerdna.com/labs/fine-print     |
| **Category:**  | Digital Forensics & IR                    |
| **Objective:** | Find the hidden flag                      |
| **Author:**    | Gabriel Dom                               |

---
## Reconnaissance

The challenge starts with a web page displaying the "Acceptable Use Policy & Data Compliance Framework" document for CipherCorp Industries.
The page source contains embedded CSS with a directive that hides any element with the `print-only` class during normal rendering:
```
.print-only {
    display: none;
}
```
Among the hidden elements is a `div` containing the location of the flag file.

## Enumeration

Making an HTTP request to the server for the path specified in the hidden element reveals the flag.

## Exploitation

No exploitation was needed, the flag was simply found by enumerating publicly accessible information.

## Recommended mitigation

### Primary
- Do not place non-public information in the HTML source. This content is sent to anyone requesting the page and can be easily found, regardless of whether it is displayed or not.

### Secondary
- Restrict access to any file that should not be public.
- If you need to implement a mechanism where "anyone with a link can access" the file, make sure the links are not easy to guess (use randomly generated values) and do not publish them anywhere.


