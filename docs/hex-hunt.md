# Hex Hunt

| Challenge:     | Hex Hunt                            |
| -------------- | ----------------------------------- |
| **Platform**:  | HackerDNA                           |
| **Lab URL:**   | https://hackerdna.com/labs/hex-hunt |
| **Category:**  | Web & API Security                  |
| **Objective:** | Find the hidden flag                |
| **Author:**    | Gabriel Dom                         |

---
## Reconnaissance
The lab starts with a web page instructing to uncover the hidden flag and enter the correct sequence.

Looking at the page source code shows data stored in a JavaScript array. When the page loads, the data is decoded into a `const` named `correctSequence`. When a user enters a value into the form on the page, the entered value is compared to the value of `correctSequence`.

There is also an event listener defined on `load`, that logs to the console some clues about `correctSequence`. 

With Developer Tools we can place the following *logpoint* in the listener: 
```
'correctSequence is:', correctSequence
```
After reloading the page, the value of `correctSequence` is printed to the console. 

## Enumeration
Submitting the value of `correctSequence` reveals that the `correctSequence` is actually the flag.

## Exploitation
No exploitation was needed, just standard interaction with a publicly available page. 

## Recommended mitigation
Do not place any secrets in web pages source code, even if the source code is generated. The source is sent to the client where it can be thoroughly analyzed. If you must send secrets to the client, use strong cryptographic mechanism intended for your particular use case to protect them. 
