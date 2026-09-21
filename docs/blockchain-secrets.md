# Blockchain Secrets

| Challenge:     | Blockchain Secrets |
| -------------- | --------------------------------------------- |
| **Platform**:  | HackerDNA                                     |
| **Lab URL:**   | https://hackerdna.com/labs/blockchain-secrets |
| **Category:**  | Digital Forensics & IR                        |
| **Objective:** | Find the flag hidden in a blockchain transaction |
| **Author:**    | Gabriel Dom                                   |

---
## Reconnaissance

The challenge provides a blockchain transaction file in JSON format. This is a Bitcoin-style, UTXO-based transaction in the legacy Bitcoin P2PKH payment format.

The transaction data contains one input value and two output values.

This input value indicates which output from a previous transaction is being spent.

The output at index 0 shows that 0.0001 BTC is transferred to a public key identified by the specified hash. This is standard for Pay-to-Public-Key-Hash (P2PKH) transactions.

The output at index 1 contains a script that starts with the opcode `OP_RETURN`. This means it does not transfer any spendable BTC - it contains data only. This is confirmed by its type, `nulldata`. The data in the `asm` and `hex` fields for this value is not strictly encoded according to the specification, so it is worth examining with a binary-data tool such as `xxd`. The raw data from the `hex` field can be extracted with `jq`:
```
cat transaction_data.json | jq '.vout[1].scriptPubKey.hex'
```
Decoding the value back to its binary form with `xxd -r -p` reveals the flag.

## Enumeration and Exploitation

No enumeration or exploitation was needed; everything was included in the public transaction data.

## Recommended mitigation

Do not store sensitive data by simply encoding it in public blockchain transaction data.
