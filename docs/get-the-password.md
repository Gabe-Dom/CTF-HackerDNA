# Get the Password

| Challenge:     | Get the Password                        |
| -------------- | ---------------------------------- |
| **Platform**:  | HackerDNA                          |
| **Lab URL:**   | https://hackerdna.com/labs/get-the-password |
| **Category:**  | Cryptography                    |
| **Objective:** | Crack the password              |
| **Author:**    | Gabriel Dom                        |

---
## Reconnaissance

The challenge starts with a simple web page containing a form with two fields labeled `Username` and `Password`, and a `Login` button. Below the form is a hint: "The password is the flag."

Looking at the page source, we can see that when `Login` is clicked, a JavaScript function called `checkLogin` is invoked. It ignores the username and checks the password by calculating an MD5 hash and then comparing it to a hardcoded value stored in the `storedHash` constant.

## Enumeration

No enumeration was needed; everything we needed was plainly available in the source of the main page.

## Exploitation

If our objective were to break the authentication, we could modify the client-side script. However, the goal is to get the password, so instead we need to exploit the weakness of storing passwords in an unsalted MD5 hash. We will find the password with repeated guesses, which is an effective attack against MD5 hashes.

We will use John the Ripper for this purpose. First, we need to put the target hash in a file. Let's name it `md5.john`. The contents of the file are:
```
target:5416d7cd6ef195a0f7622a9c56b55e84
```
Then run the tool with the popular `rockyou.txt` wordlist:
```
john --format=Raw-MD5 --wordlist=rockyou.txt md5.john 
john --show --format=Raw-MD5 md5.john
```
This reveals the password, which is the flag.

## Recommended mitigation
- Do not use MD5 to hash passwords; use one of the dedicated password hashing schemes such as Argon2, bcrypt, or scrypt.
- Do not hardcode any hashes in client-side source code; use secure server-side storage.


