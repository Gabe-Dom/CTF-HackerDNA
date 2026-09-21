# Packet Pursuit

| Challenge:     | Packet Pursuit                                 |
| -------------- | ---------------------------------------------- |
| **Platform**:  | HackerDNA                                      |
| **Lab URL:**   | https://hackerdna.com/labs/packet-pursuit      |
| **Category:**  | Digital Forensics & IR                         |
| **Objective:** | Examine a packet capture file to find the flag |
| **Author:**    | Gabriel Dom                                    |

---
## Reconnaissance
The challenge is to examine the `challenge.pcap` file to find the flag. An initial look at the file with `Wireshark` shows 118 packets using multiple protocols.

Applying the initial broad filter `frame matches "(?i)flag"` reveals that the flag is hidden in DNS, ICMP, and HTTP packets, as well as in frames identified as TCP retransmissions.

## DNS packets

In DNS query packets, parts of the flag are hidden in artificial hostnames in the `lab.hdna.me` domain. This is a well-known data-exfiltration technique. We can extract the flag using the following tools:

**tshark** to extract the fully qualified domain names (FQDNs) from DNS packets containing the flag:
```
tshark -r challenge.pcap \
  -Y 'dns.flags.response == 0 && dns.qry.name contains "flag_"' \
  -T fields -e dns.qry.name
```
The `-r` flag tells `tshark` to read the capture file. The `-Y` option specifies the filter, in this case DNS requests whose queries contain `flag_`. Finally, the `-T` and `-e` options instruct `tshark` to output the value of the `dns.qry.name` field, where the flag is hidden.

**sed** to extract the actual flag part from each FQDN:
```
sed -E 's/^flag_([^.]+)\..*/\1/'
```

**paste** to combine the extracted pieces into the final flag:
```
paste -s -d ''
```
The `-d` option specifies the output delimiter (an empty string in this case), and `-s` instructs `paste` to read all input from one file.

Combining the above tools in one pipeline produces the complete flag.

## ICMP packets

The second type of packets is ICMP "ping" traffic. In this case, the exfiltration technique is more sophisticated because it hides parts of the flag in the ICMP `data` field. The flag can be extracted with the following tools:

**tshark** to extract the data fields from ICMP type 8 (echo request aka "ping") packets containing flag pieces:
```
tshark -r challenge.pcap \
  -Y 'icmp.type == 8 && icmp.data contains "FLAG"' \
  -T fields -e icmp.data 
```

**xxd** to convert the hexadecimal output produced by `tshark` back to binary data:
```
xxd -r -p
```
The `-r` means reverse the usual operation: decode hex into binary. The `-p` means plain hex input.

**sed** to remove "FLAG_PART: " text and leave the actual flag part only:
```
sed -E 's/FLAG_PART: //g'

```
**paste** to ensure the final result is the flag ending with a new line:
```
paste -s -d ''
```

As before, combining the tools in one pipeline produces the complete flag. It is identical to the flag hidden in the DNS packets.

## HTTP packets

The third type of packets is HTTP requests. However, by default, some of these packets are identified by `Wireshark` as *TCP Spurious Retransmission* instead of regular HTTP packets. To prevent this, set the `tcp.analyze_sequence_numbers` option in `Wireshark` to `false`. The flag parts are sent as HTTP headers. The flag can be extracted using a combination of tools similar to the previous examples:

**tshark** to extract the header fields from HTTP packets containing flag parts:
```
tshark -r challenge.pcap \
  -o tcp.analyze_sequence_numbers:false \
  -Y 'http.request && http.request.line matches "^FLAG:.*"' \
  -T fields -e http.request.line 
```

**xxd** to convert hexadecimal numbers produced by `tshark` back to binary data
```
xxd -r -p
```
The `-r` means reverse the usual operation: decode hex into binary. The `-p` means plain hex input.

**sed** to remove everything except the flag part:
```
sed -E 's/.*FLAG: ([^\\]+)\\r\\n.*/\1/g' 
```
**paste** to concatenate the parts into the flag, ending with a newline:
```
paste -s -d ''
```
Once again, combining the tools in one pipeline produces the complete flag, which is identical to the flag hidden in the DNS and ICMP packets.

## Recommended mitigation

The exercise demonstrates three techniques that attackers may use to exfiltrate data from a compromised system. No single tool can protect against all these and similar techniques. Instead, implement layered egress controls, with the key principle that every outbound path should be attributable, policy-controlled, and observable.
