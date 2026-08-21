# Hack the Cookie

| Challenge:     | Hack the Cookie                            |
| -------------- | ------------------------------------------ |
| **Platform**:  | HackerDNA                                  |
| **Lab URL:**   | https://hackerdna.com/labs/hack-the-cookie |
| **Category:**  | Web & API Security                         |
| **Objective:** | Cookie poisoning attack                    |
| **Author:**    | Gabriel Dom                                |

---
## Reconnaissance
The starting page of the lab shows a TechCorp Internal Portal login page, with guest credentials already filled-in and a `Login` button. 

The page source reveals the pre-filled password for the user `guest` is `guest`.

The Dev Tools show that no cookie is set yet.

## Enumeration
After clicking the `Login` button we get successfully logged in as `guest@techcorp.local`. There is a cookie set with the name `user-session` and the value:
```
eyJ1c2VyX2lkIjogMSwgInVzZXJuYW1lIjogImd1ZXN0IiwgInJvbGUiOiAiZ3Vlc3QiLCAiZW1haWwiOiAiZ3Vlc3RAdGVjaGNvcnAubG9jYWwifQ==
```
The value looks like base64 encoded. Decoding it in the browser console reveals a json object:
```
> atob('eyJ1c2VyX2lkIjogMSwgInVzZXJuYW1lIjogImd1ZXN0IiwgInJvbGUiOiAiZ3Vlc3QiLCAiZW1haWwiOiAiZ3Vlc3RAdGVjaGNvcnAubG9jYWwifQ==')
'{"user_id": 1, "username": "guest", "role": "guest", "email": "guest@techcorp.local"}'

```
The role is embedded in the cookie, which points to the possible exploitation.

## Exploitation
We modify the cookie value to change role to `admin' and re-encode as base64:

```
> btoa('{"user_id": 1, "username": "guest", "role": "admin", "email": "guest@techcorp.local"}')
'eyJ1c2VyX2lkIjogMSwgInVzZXJuYW1lIjogImd1ZXN0IiwgInJvbGUiOiAiYWRtaW4iLCAiZW1haWwiOiAiZ3Vlc3RAdGVjaGNvcnAubG9jYWwifQ=='
```

Now using Dev Tools we replace the cookie value in the application storage with the newly prepared one and reload the page. We are still logged in as the same user, but now with administrator access granted thanks to the `admin` role. The page reveals the flag in the "Sensitive Information" section.

## Recommended mitigation
### Primary
- Do not store the session information in the cookie itself. Store only the session id in the cookie and use it as a key to retrieve the values stored on the server side.
- If you don't have the ability to keep a server side storage and thus must store values in the browser cookies, make sure to use strong encryption for them. 

### Secondary
- Add additional protection for cookies using `HttpOnly`, `Secure` and `SameSite` options.


