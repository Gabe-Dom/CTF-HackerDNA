# Infiltrator

| Challenge:     | Infiltrator                                 |
| -------------- | ------------------------------------------- |
| **Platform**:  | HackerDNA                                   |
| **Lab URL:**   | https://hackerdna.com/labs/infiltrator      |
| **Category:**  | Web & API Security                          |
| **Objective:** | Conduct a comprehensive security assessment |
| **Author:**    | Gabriel Dom                                 |

---
## Reconnaissance

The lab gives us the IP address of the target and asks us to conduct a comprehensive security assessment.

We can start reconnaissance by visiting the target with a regular web browser. There is a simple CyberSec Corp. website with options to log in or sign up. Examining the source of the home, login, and signup pages does not reveal any weaknesses or clues.

The page allows anyone to create an account, so let's do it. We sign up as user `foo`. We can now log in to the dashboard and are greeted with the message, "Welcome, foo! You have user privileges."

As part of the reconnaissance, we can also perform a non-intrusive port scan with `nmap`:
```
$ nmap -p- -sS $TARGET
...
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
...
```
The scan confirms an HTTP server on port 80 and additionally identifies a possible SSH service accessible from an external network.

## Enumeration for the user flag

### Website authentication mechanism
Let's investigate the authentication mechanism used by the website. Looking at the cookies after logging in as `foo`, we can see a cookie named `jwt` containing a JSON Web Token. We can decode the token, for example, with [jwt.io](https://www.jwt.io/) from Okta. Its decoded contents are:

Header:
```
{
  "alg": "HS256",
  "typ": "JWT"
}
```
Payload:
```
{
  "username": "foo",
  "role": "user",
  "exp": 1789640796
}
```
The token is signed with an HMAC-based algorithm, so we cannot verify the signature without knowing a shared secret.

Let's check for weaknesses in the target's token verification.

#### Attempt 1
Let's keep the token contents unchanged to make sure the payload and header are what the website expects, and sign it with the same HS256 algorithm using an arbitrary secret. Once again, we can use [jwt.io](https://www.jwt.io/) for this purpose. With the new token ready, use the browser's developer tools to change the cookie contents and reload the dashboard page.

The website responds with "Invalid token!", which means it actually does *some* signature verification.

#### Attempt 2
Let's keep the token payload unchanged, but modify the header to look like this:
```
{
  "alg": "none"
}
```
This means the token is valid without a signature. If the website uses general-purpose token validation, it will pass. Once again, replace the `jwt` cookie contents and try.

The website responds with "Invalid token!", which means it does not accept tokens that are valid but unsigned.

#### Attempt 3
It is still possible that the website cuts corners on token verification, so let's make a small change to the timestamp to ensure the contents are still what the website expects, and reuse the signature from the original token. The signature does not match the modified content, but perhaps the website does not check.

An attempt to submit this token again results in "Invalid token!", which strongly suggests that the website performs proper token verification.

#### Attempt 4
Let's try to "guess" the shared secret used for the token signature. We will use John the Ripper for this purpose. First, we need to prepare the token in the `<MESSAGE>#<SIGNATURE>` format expected by `john`. `<MESSAGE>` consists of the first two Base64-encoded chunks of the JWT. `<SIGNATURE>` is the last chunk, encoded as hexadecimal instead of Base64.

Assuming we have the original token in `$token`, the following bash snippet prepares the file for `john`:
```
message=${token%.*}
sig=${token##*.}
echo -n "${message}#" > jwt.txt 
echo "$sig" | basenc -d --base64url | xxd -p -c0 >>jwt.txt
```

Now we can run:
```
$ john --format=HMAC-SHA256 --wordlist=wordlists/rockyou.txt jwt.txt 
```
This quickly finds the password that can be used to sign JWT tokens for the CyberSec Corp. website.

To verify that everything works as expected, we decode the original token, re-sign it with [jwt.io](https://www.jwt.io/) using the discovered password, and try to access the website with this token. We are greeted as the `foo` user.

## Exploitation for the user flag

### Exploit 1: JWT manipulation

We can use the token-signing mechanism discovered during enumeration to escalate the privileges of the `foo` user. Let's get a fresh token for user `foo`, decode it, modify the payload by replacing `"role": "user"` with `"role": "admin"`, and re-sign it.

When accessing the dashboard with the modified token, we get a new message: "Welcome, foo! You have **admin** privileges." There is also an additional button labeled "System Administration." Clicking the button takes us to a page listing credentials for SSH access.

### Exploit 2: SSH exploitation

With the newly discovered credentials, we can exploit the externally accessible OpenSSH service and log in to the target as user `ctf`. The file containing the flag is in the `$HOME` directory. This shell access gives us a foothold from which to enumerate further possibilities.

## Enumeration for the root flag

Looking at the filesystem, we can see a non-standard `start.sh` file in `/`. Examining its contents reveals that it starts a couple of regular services, but at the end it does one particularly interesting thing: it runs a shell script with `sudo`:
```
sudo /usr/local/bin/log_watcher.sh
```
Running a script as root always deserves further investigation.

The file is write-protected, so we cannot modify its contents:
```
$ ls -l /usr/local/bin/log_watcher.sh
-rwxr-xr-x 1 root root 253 Aug 30 08:26 /usr/local/bin/log_watcher.sh
```
However, we can investigate what it does. As it turns out, it continuously reads the `/var/log/custom.log` file and runs `eval` on each line it reads.

```
$ cat /usr/local/bin/log_watcher.sh
...
while read -r line; do
    [ -n "$line" ] && eval "$line"
done < "$LOG_FILE"
```
Let's check the `/var/log/custom.log` file:
```
$ ls -l /var/log/custom.log
-rw-rw-rw- 1 root root 0 Sep  9 07:28 /var/log/custom.log
```
It is world-writable!

This means we can add whatever we want to the file, and it will be executed with `root` privileges by `eval` in `log_watcher.sh`.

## Exploitation for the root flag

To exploit the vulnerability found during enumeration, we can add a line to `/var/log/custom.log`:
```
$ echo "echo root:you_have_been_pwned | chpasswd; touch /tmp/done" > /var/log/custom.log
```
This changes the password of `root` to `you_have_been_pwned` and also creates a file `/tmp/done`, so we can see that it was executed. Now we wait a moment and check `/tmp`:

```
$ ls -l /tmp/
total 0
-rw-r--r-- 1 root root 0 Sep 17 16:11 done
```
The file was created, which means we can now try to log in as `root` with the newly set password:

```
$ su -
Password: 
# whoami
root
```
The root flag is located in the $HOME of `root`.

## Recommended mitigation

### For the user flag exploit

- Use strong keys with high entropy for shared secrets used to sign and verify JWTs.
- Consider storing roles and access rights on the server side instead of relying solely on content sent by the user agent, even if it is inside a JWT.
- Do not share secrets, such as an SSH password, by placing them in plain text on a web page, even if the page is protected. Instead, use a dedicated secret-sharing tool.
- Do not expose SSH externally. If you need remote access, use a virtual private network.

### For the root flag exploit
- Avoid `eval` on unsanitized input.
- Avoid running shell scripts with `sudo` unless absolutely necessary.
- Follow the principle of least privilege.



