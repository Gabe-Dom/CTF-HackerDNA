# Secrets in Source 2

| Challenge:     | Secrets in Source 2                            |
| -------------- | ---------------------------------------------- |
| **Platform**:  | HackerDNA                                      |
| **Lab URL:**   | https://hackerdna.com/labs/secrets-in-source-2 |
| **Category:**  | Web & API Security                             |
| **Objective:** | Bypass security through obscurity              |
| **Author:**    | Gabriel Dom                                    |

---
## Reconnaissance
The lab opens as a website of a fictional SecureVault company. Right mouse click is captured by the page, so there is no way to view source from the right-click menu. 

Dev Tools from the browser menu open as expected. The page source in Dev Tools reveals a comment with a TODO note. The note reveals a path the `flag.txt` file.
## Enumeration
The path in the comments starts with "/" which means it is an absolute path. However making a request to the URL built by concatenating the origin with the absolute path returns 404.

The next step is to try ignore the starting slash and treat the path as relative. This request succeeds and retrieves the file with the flag.
## Exploitation
No exploitation was needed, just enumerating what was publicly accessible. 
## Recommended mitigation
### Primary
- Lock access to any file which you don't want to be public. 
### Secondary
- If you need to implement the "anyone with a link can access" mechanism, make sure that the links are not easy to guess (randomly generated) and do not publish them anywhere.


