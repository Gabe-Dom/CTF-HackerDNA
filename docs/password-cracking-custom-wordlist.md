# Password Cracking - Custom Wordlist Attack

| Challenge:       | Password Cracking - Custom Wordlist Attack                         |
| ---------------- | ------------------------------------------------------------------ |
| **Platform**:    | HackerDNA                                                          |
| **Lab URL:**     | https://hackerdna.com/labs/password-cracking-custom-wordlist       |
| **Category:**    | Cryptography                                                       |
| **Objective:**   | Crack passwords using custom wordlists                             |
| **Author:**      | Gabriel Dom                                                        |

---
## Reconnaissance

The challenge starts with a website for Bitmarmot Games, a small game development studio. It consists of four pages: Home, Team, Devlog, and a login page for the Dev Portal.

There are two interesting tidbits of information on the Devlog page:
- Entry 15 talks about retiring an old forum and exporting the accounts to move them to a new one.
- Entry 19 is about a kart racer called Radish Rally that was shelved, with the entire prototype archived by Otso.

The team page introduces four team members and includes trivia about their professional and personal lives.

The website's source code does not reveal any additional clues.

## Enumeration for the user flag

Let's enumerate some of the most obvious locations on the website to see if anything is publicly available. Trying the `/backup` path reveals that the directory exists and is publicly accessible. The content is:
```
forum-users.sql                 866
README.txt                      150
```

This is the account export from a retired forum, mentioned in Entry 15. The `forum-users.sql` file contains usernames and unsalted MD5 hashes of their passwords. There are four users in the SQL file, with usernames matching those on the Team page. It is safe to assume that these are the team members' accounts.

Let's extract the usernames and hashes from `forum-users.sql` into a separate file in a format suitable for `john`, and name it `forum.hash`. This can be done manually or with a simple `awk` script. Now, using information from the website, let's build a short wordlist of possible passwords - names of favorite games, pets, and hobbies - in `forum.lst`. At this point, we do not need to worry about capitalization or variants. Those will be handled by `rules` in `john`.

## Exploitation for the user flag.txt

It's time to run the tool:
```
john --format=Raw-MD5 --wordlist=forum.lst --rules forum.hash 
john --format=Raw-MD5 --show forum.hash 
```
With the wordlist we prepared, `john` managed to guess the password of the forum user `mika`.

Let's see if Mika Sundstrom reused her forum password for the Dev Portal. She did! We successfully logged in as Mika to the Dev Portal and found the user flag.

## Enumeration for the root flag

With the newly gained user access, we can enumerate the website further. In the Dev Portal, there is a link to Otso's hand-rolled archives. We follow the link and reach the "Studio backups" page, from which we can download a file named `vault.zip`. This is the prototype of a kart racer mentioned in Entry 19 of the Devlog. The "Studio backups" page includes an additional hint: Otso set the archive passphrase himself.

We can examine the `vault.zip` file:
```
$ unzip -l vault.zip
Archive:  vault.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
       37  2026-09-07 07:08   flag-root.txt
      228  2026-09-07 07:08   handover.txt
---------                     -------
      265                     2 files
```

Let's extract the hash in a format suitable for `john`:
```
zip2john -o flag-root.txt vault.zip > vault.hash 
```

We know that what we are looking for is actually a passphrase, and that it was set personally by Otso. Let's prepare a new wordlist of base versions of possible passphrases based on what we know about Otso.

## Exploitation for the root flag
Running `john` with the default rules did not produce a result.
```
john --wordlist=otso.lst --rules vault.hash 
```
However, with an extended set of rules, it cracked the password:
```
john --wordlist=otso.lst --rules=all vault.hash 
john --show vault.hash 
```

## Recommended mitigation
### For admins
- Protect any directories that are not intended to be public, especially anything containing credentials.
- Leaving backups unprotected is a common mistake. Make a special effort to ensure backups are secured by default.
- Do not use unsalted MD5 for passwords. Instead, use one of the dedicated password hashing schemes such as Argon2, bcrypt, or scrypt.

### For users
- Do not reuse your passwords.
- Do not base your passwords or passphrases on public information about you.
- Despite the popular misconception, adding special characters or digits to a password does not meaningfully improve its strength. Use long passwords or passphrases: length beats complexity.
