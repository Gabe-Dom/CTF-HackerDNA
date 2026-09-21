# HTTP Smuggling

| Challenge:     | HTTP Smuggling                            |
| -------------- | ------------------------------------------|
| **Platform**:  | HackerDNA                                 |
| **Lab URL:**   | https://hackerdna.com/labs/http-smuggling |
| **Category:**  | Digital Forensics & IR                    |
| **Objective:** | Analyze the captured network traffic<br> to uncover an HTTP request smuggling attack |
| **Author:**    | Gabriel Dom                               |

---
## Reconnaissance

The lab starts with a packet capture file and instructions to uncover an HTTP request smuggling attack and find the hidden administrative token in the smuggled request.

Looking at the captured traffic with `wireshark`, we can see a frame that contains a malformed HTTP request, characteristic of HTTP request smuggling. This request sets both `Content-Length` and `Transfer-Encoding` headers at the same time.

The request begins as an innocent POST:
```
POST /search HTTP/1.1\r\n
Host: vulnerable-app.hdna.me\r\n
Content-Length: 44\r\n
```
However, the `Content-Length` header set to 44 bytes is bogus, because the request also sets `Transfer-Encoding` to `chunked` and in the subsequent content it smuggles a completely different, malicious request:
```
POST /api/admin HTTP/1.1\r\n
Host: backend.internal\r\n
X-Admin-Token: ...
```
The `X-Admin-Token` header contains the hidden administrative token, which is the flag.

## Recommended mitigation
- Reject requests containing both Transfer-Encoding and Content-Length.
- Broader mitigation: enforce strict HTTP validation at a controlled edge proxy. Enable the proxy’s request-smuggling/desync protections, strict HTTP parsing, and invalid-header rejection features.

