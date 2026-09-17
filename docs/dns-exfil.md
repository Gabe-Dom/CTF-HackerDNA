# DNS Exfil

| Challenge:     | DNS Exfil                                       |
| -------------- | ----------------------------------------------- |
| **Platform**:  | HackerDNA                                       |
| **Lab URL:**   | https://hackerdna.com/labs/dns-exfil            |
| **Category:**  | Digital Forensics & IR                          |
| **Objective:** | Analyze the traffic and extract the hidden flag |
| **Author:**    | Gabriel Dom                                     |

---
## Reconnaissance

The challenge starts with a packet capture file and instructions to find hidden data that has been exfiltrated through DNS tunneling.

The file contains a number of DNS queries. All but one query well-known public websites; the remaining one queries a suspicious hostname in the `attacker.com` domain. The hostname looks like Base64-encoded text. Decoding it with `base64 -d` reveals the flag.

## Recommended mitigation

## Primary
The primary mitigation against DNS exfiltration is to make resolvers under your control the only permitted path from your network to the Internet's DNS system. Additionally, you should consider blocking unapproved "DNS over HTTPS" (DoH) and "DNS over TLS" (DoT) traffic.
## Secondary
Log full DNS query names and response codes. Detect long labels, high rates of queries to unique subdomains, unusual TXT/NULL use, and sustained NXDOMAIN rates.