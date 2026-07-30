# Hack the Login

| Challenge:     | Hack the Login                            |
| -------------- | ----------------------------------------- |
| **Platform**:  | HackerDNA                                 |
| **Lab URL:**   | https://hackerdna.com/labs/hack-the-login |
| **Category:**  | Web & API Security                        |
| **Objective:** | Authentication bypass                     |
| **Author:**    | Gabriel Dom                               |

---
## Reconnaissance
The starting page of the lab shows  "Access Restricted", followed by a form with text fields for `Username` and `Password` and a submit button labeled `Login`.

Looking at the page source shows that the form does not have any associated address to which it should be submitted.

The source reveals that additional JavaScript code is loaded from `script.js`.
## Enumeration
Accessing the source of  `script.js` reveals that the code adds an event listener to the form's `submit` event.  This means that clicking `Login` does not send anything to the server directly, but just triggers the JavaScript listener, which checks the `username` and `password` on the client side. The check is done by comparing them to hard-coded string values. On success, the listener fetches a flag file from the server.
## Exploitation
There are two independent ways of getting the flag:
### Exploit 1
You can pass authentication by providing the hard-coded username and password copied from the `script.js`  and clicking the `Login` button.  This reveals the flag on the main page.
### Exploit 2
The URL to the flag file is hard-coded in the event listener code in `script.js`,  so you can bypass authentication completely and just request the file directly. It loads the text file and shows the flag in the browser.
## Recommended mitigation
### Primary
- Move the authentication (checking of the credentials) to the server side. The current implementation on the client can be tampered with by an attacker.
- Protect access to the flag file on the server-side. Currently, anyone who knows and guesses the URL to the flag file can load it.
### Secondary
- Even with authentication and authorization moved to the server side, it is better not to use credentials hard-coded in the code. Instead store a salted hash of the password in a secure store on the server.


