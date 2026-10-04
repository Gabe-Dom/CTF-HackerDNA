# Memory Forensics

| Challenge:     | Memory Forensics                            |
| -------------- | ------------------------------------------- |
| **Platform**:  | HackerDNA                                   |
| **Lab URL:**   | https://hackerdna.com/labs/memory-forensics |
| **Category:**  | Reverse Engineering & Binary Exploitation   |
| **Objective:** | Find the flag hidden in a memory dump       |
| **Author:**    | Gabriel Dom                                 |

---
## Reconnaissance

The lab scenario is that we have been given a memory dump from a suspicious system, and our mission is to analyze it to find hidden data and extract the flag.

## Enumeration

Let’s first take a look at the file with `xxd`. It is a 32 MB binary file. It starts with `PROCESS_LIST_HEADER`, which is interesting to note, but does not help with finding the flag.

We can use `grep` with the `-aob` options to search for patterns in the file. These options tell `grep` to treat the binary as text and to print the offset of each match found.

We expect the flag to be in UUID format, so let’s search for it:
```
grep -aob -E '[0-9A-Fa-f]{8}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{4}-[0-9A-Fa-f]{12}' memory_dump.raw 
```
It yields no results.

The flag might be base64-encoded, so let’s search for candidate base64 strings. A UUID has 32 digits and 4 hyphens. It can be encoded with or without hyphens, with or without padding, so let’s look for strings from the base64 alphabet that are at least 43 characters long, since 43 is the minimum number of bytes needed to encode 32 hex digits.

```
>grep -aob E '[A-Za-z0-9+/]{43,}={0,2}' memory_dump.raw
262247:yZmFrZV9iYXNlNjRfZGF0YV90aGF0X2lzX25vdF90aGVfZmxhZw==
```

Great! We have a match. Let’s try to decode it:
```
> echo yZmFrZV9iYXNlNjRfZGF0YV90aGF0X2lzX25vdF90aGVfZmxhZw== | base64 -d
base64: invalid input
```
This turns out not to be valid base64 encoding. Let’s examine it more carefully. The string is 53 characters long, and we know that the length of padded base64 encoding is always a multiple of 4. We have one extra byte, either at the beginning or the end. The end looks like proper padding, so perhaps the actual encoded string starts at `Z`, and the leading `y` is just binary data preceding the encoding. Let’s test this theory:
```
> echo ZmFrZV9iYXNlNjRfZGF0YV90aGF0X2lzX25vdF90aGVfZmxhZw== | base64 -d
fake_base64_data_that_is_not_the_flag
```
The theory is confirmed, and the data decodes properly, but it turns out to be a red herring.

Let’s try a different approach and look for the word "flag" in a case-insensitive way:
```
> grep -aob -i 'flag' memory_dump.raw
458788:FLAG
524314:Flag
```
We got two matches. Let’s examine them more closely by looking at the bytes surrounding their positions.

For the first match, we get the following chunk:
```
Packet payload: FLAG=s38sopp3-4076-4r14-n970-4857qsrn5op1
```
For the second match, we see the following:
```
SecretFlag=s38sopp3-4076-4r14-n970-4857qsrn5op1
```
In both cases, this is the same flag. It is written in UUID format but uses the wrong alphabet. In a proper UUID, we expect letters from `[a-f]`, but instead we have `[n-s]`. The number of characters in both alphabets is 6, so this seems to be a simple substitution cipher. Moreover, we can see that `'n' - 'a' = 13`, which strongly suggests it was encrypted with ROT13.

Let’s decrypt it:
```
echo s38sopp3-4076-4r14-n970-4857qsrn5op1 | tr 'n-s' 'a-f'
```

This produced the final flag.

## Recommended mitigation

Storing sensitive data in unencrypted or weakly encrypted form in memory is a known security risk, covered by CWE-316: Cleartext Storage of Sensitive Information in Memory.

### Primary mitigations:
- Keep secrets in memory only as long as needed and wipe them immediately after use.
- Protect retained memory dumps.

### Secondary mitigations, if feasible:
- Disable unnecessary crash dumps.
- Make sensitive processes non-dumpable.
- Exclude sensitive memory mappings from dumps.