# Book Cipher Challenge

| Challenge:     | Book Cipher Challenge                            |
| -------------- | ------------------------------------------------ |
| **Platform**:  | HackerDNA                                        |
| **Lab URL:**   | https://hackerdna.com/labs/book-cipher-challenge |
| **Category:**  | Cryptography                                     |
| **Objective:** | Decrypt a message using a Book Cipher            |
| **Author:**    | Gabriel Dom                                      |

---
## Reconnaissance

The lab provides a message encrypted with a Book Cipher and the reference text (the book). Let's save the ciphertext as `cipher.txt` and the reference as `reference.txt`.

## Enumeration

A quick look at the ciphertext suggests it mostly references line 5, with a few occasional references to other lines. Let's use a simple one-liner to create a unique list of referenced lines:
```
cat cipher.txt | awk 'BEGIN { RS=" "; FS=":" } !used[$1]++ { print $1 }' | sort
```
It produces:
```
1
3
5
```
These are the only referenced lines. It means we may truncate the reference text to the first 5 lines:
```head -n 5 reference.txt > ref.txt```

## Exploitation

The trimmed reference text is small enough to load into memory, so we can implement a simple decrypt tool in awk:

```
#!/usr/bin/awk -f

# This block is called for the first file (reference text)
NR == FNR {
  # Store each line split into words in the lines array
  split($0, lines[NR], /[[:space:]]+/)
  next
}
# This block is called for the second file (ciphertext)
{ 
  # Split the ciphertext into array of references
  len = split($0, refs, " ")
  for (i = 1; i <= len; i++) {
    # Split the reference into 3 pointers (line, word, letter)
    split(refs[i], p, ":")
    # Print the character referenced by the pointers
    printf("%s", substr(lines[p[1]][p[2]], p[3], 1))
  }
}
END { print "" }
```
Let's save the program as `book_decrypt.awk` and run it:
```
./book_decrypt.awk ref.txt cipher.txt
```
This produces the plaintext and reveals the flag.

## Recommended mitigation

Not applicable; the exercise was about decrypting the cipher with all the information given up front.
