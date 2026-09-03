# Nmap Commands Lab

| Challenge:     | Nmap Commands Lab                      |
| -------------- | -------------------------------------- |
| **Platform**:  | HackerDNA                              |
| **Lab URL:**   | https://hackerdna.com/labs/learn-102   |
| **Category:**  | Network & Infrastructure               |
| **Objective:** | Port Scanning to Privilege Escalation  |
| **Author:**    | Gabriel Dom                            |

---
## Reconnaissance
We're given the IP address of the target system. In the "ATTACK Terminal" the IP is stored in the `$TARGET` environment variable.

Basic recon reveals an HTTP server running on a standard port:
```
$ curl -i http://$TARGET
HTTP/1.1 200 OK
Date: Thu, 03 Sep 2026 07:10:39 GMT
Connection: close
Content-type: text/html
Accept-Ranges: bytes
Last-Modified: Tue, 24 Feb 2026 19:26:26 GMT
ETag: "699dfb62-59"
Content-Length: 89

<html><body><h1>Server is Running</h1><p>There is nothing to see here.</p></body></html>
```

This exercise is about `nmap`, so let's start with a scan of open ports. We're still in the reconnaissance phase so we do "stealth" mode, that is sending `SYN` only:
```
$ nmap -sS $TARGET
Starting Nmap 7.98 ( https://nmap.org ) at 2026-09-03 07:19 +0000
Nmap scan report for localhost (108.130.16.39)
Host is up (0.0000030s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
23/tcp open  telnet
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 0.12 seconds
```
We got confirmation of the HTTP server plus information about `telnet` service running on the target system.

## Enumeration

Let's see if we can actually connect to the `telnet` service:
```
$ telnet $TARGET
Trying 108.130.16.39...
Connected to 108.130.16.39.
Escape character is '^]'.

108.130.16.39 login: 
```
Yes, the server accepts connections and we may try to exploit it to gain shell access.

## Exploitation
### Exploit 1 (user flag)
When manually trying the most obvious credentials, it turns out that username `user` can login to the target via `telnet` without being asked for password:

```
$ telnet $TARGET
Trying 108.130.16.39...
Connected to 108.130.16.39.
Escape character is '^]'.

108.130.16.39 login: user
Welcome to Alpine!
...
```
We are now logged in as `user`. The list of processes matches the list of services found by `nmap`, which additionally confirms we are logged in to the real target system:
```
$ ps aux
PID   USER     TIME  COMMAND
    1 root      0:00 /usr/sbin/inetd -f
    8 root      0:00 httpd -p 80 -h /var/www
 1066 root      0:00 telnetd -i
 1067 root      0:00 /bin/login
 1068 user      0:00 -sh
 1115 user      0:00 ps aux
```

The user flag is located in a text file in `user`'s home dir.

### Exploit 2 (root flag)
Let's try the most simple privilege escalation using `su`.
```
$ su -
Password: 
```
It accepts the password `root` and gives us the root access to the target.

The flag is located in a text file in `root`'s home dir.

## Recommended mitigation
### Primary
- Enforce strong passwords for local users and in particular for root.
### Secondary
- Consider disabling `telnet` in favor of `ssh` if you need to allow remote access to shell.
