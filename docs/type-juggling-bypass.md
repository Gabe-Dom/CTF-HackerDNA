# Type Juggling Bypass

| Challenge:     | Type Juggling Bypass                            |
| -------------- | ----------------------------------------------- |
| **Platform**:  | HackerDNA                                       |
| **Lab URL:**   | https://hackerdna.com/labs/type-juggling-bypass |
| **Category:**  | Web & API Security                              |
| **Objective:** | Break into the admin panel                      |
| **Author:**    | Gabriel Dom                                     |

---
## Reconnaissance
The lab starts with a login page for the "Admin Portal." There is a form with "Username" and "Password" fields, a "Login" button, and a link labeled "View Source Code."

Clicking the link shows the PHP source code of the web page. We can see that the data entered into the form and submitted by the user is compared to hard-coded values. The `username` is compared directly to `admin`, and the `password` is hashed with MD5 and compared to a hard-coded hash. There is no need to crack the hash, because the password value is written in the comment next to it.

The flag is also clearly visible in the PHP source, so we could use it without actually breaking into the admin panel. However, let's achieve the objective and actually enter the panel.

## Enumeration
Submitting the username `admin` and the password copied from the PHP source gets us into the "Admin Portal" and reveals the flag.

## Exploitation
No exploitation was needed; it was simply a matter of using credentials listed on a publicly available page.

## Recommended mitigation
### Primary
- Do not hardcode secrets in your source code, even if that source code is intended to be executed on the server.
- Protect your server-side source code from being accessed unless you deliberately want to make it public; in that case, use dedicated tools to scan it for secrets before publishing.

### Secondary
- Do not use MD5 to hash passwords. Use one of the dedicated password hashing schemes, such as Argon2, bcrypt, or scrypt.
