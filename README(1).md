# Bandit Writeups — OverTheWire

Notes from working through [OverTheWire Bandit](https://overthewire.org/wargames/bandit/),
a beginner-friendly Linux/security wargame. Each entry focuses on the
**technique and underlying concept** rather than the literal password/flag,
in line with the Bandit community's norm against publishing spoilers.

**Progress:** Level 24+ (levels 0 → 24 documented below)

## Index

| Level | Topic |
|---|---|
| 0 → 1 | SSH basics, non-default port |
| 1 → 2 | Referencing filenames starting with special characters |
| 2 → 3 | Filenames containing spaces |
| 3 → 4 | Hidden files (`-a` flag) |
| 4 → 5 | Identifying file type with `file` |
| 5 → 6 | `find` with multiple filters (size, type, permissions) |
| 6 → 7 | `find` across the whole filesystem + stderr redirection |
| 7 → 8 | Searching text with `grep` |
| 8 → 9 | `sort` + `uniq` to find a unique line |
| 9 → 10 | `strings` + `grep` for readable text in a binary  |
| 10 → 11 | Base64 decoding |
| 11 → 12 | ROT13 |
| 12 → 13 | Iterative decompression (gzip/bzip2/tar) |
| 13 → 14 | SSH private key authentication |
| 14 → 15 | Submitting data to a local port |
| 15 → 16 | Submitting data over SSL/TLS |
| 16 → 17 | Port scanning to find the right service |
| 17 → 18 | Comparing files with `diff` |
| 18 → 19 | Running a remote command over SSH without an interactive shell |
| 19 → 20 | SUID binary, no arguments |
| 20 → 21 | SUID binary with an argument + background listener |
| 21 → 22 | Reading a cron job |
| 22 → 23 | Cron job writing to a predictable path |
| 23 → 24 | Cron job + script injection |

---

## Level 0 → 1

**Goal:**
Log in to the game using SSH on port 2220; find and print a file called readme.

**Method:**
SSH into the server using port 2220 with the provided credentials: `ssh bandit0@bandit.overthewire.org -p 2220`. Once logged in, list files with `ls` to find the readme file, then display its contents with `cat readme`.

**Concept:**
SSH creates an encrypted, authenticated tunnel to a remote server — the `-p` flag specifies a non-standard port (2220 instead of the default 22), useful when the standard SSH port is blocked or reserved. Once connected, the shell environment switches to the remote user's context, meaning `ls` and `cat` now operate on the remote server's filesystem, not the local machine. This is foundational to all remote system access in Linux.

---

## Level 1 → 2

**Goal:**
Read a file called `-`.

**Method:**
Use `cat ./-` to print its contents.

**Concept:**
A filename in Linux that starts with a special character needs to be referenced explicitly by its path. In this case, `./-` — where `.` means "this current directory" — tells the shell "target a file named `-` inside this directory," instead of letting `-` be interpreted as an option flag.

---

## Level 2 → 3

**Goal:**
Read a file called `--spaces in this filename--`.

**Method:**
Use `cat "./--spaces in this filename--"` to print its contents.

**Concept:**
In a Linux terminal, spaces separate arguments. If a filename containing spaces is passed without quotes, the shell treats each space-separated piece as a *separate* argument. Wrapping the name in double quotes makes it read as a single string instead.

---

## Level 3 → 4

**Goal:**
Read a hidden file.

**Method:**
Use `cd` to change directory, then list files with `ls -a`.

**Concept:**
Linux directories can be navigated using either an absolute path (starting from root `/`) or a relative path (starting from the current directory, e.g. `./`). Hidden files (names starting with `.`) are used to store configuration/system data without cluttering a normal directory listing — they're revealed with the `-a` flag.

---

## Level 4 → 5

**Goal:**
Find a human-readable file among several files in a directory.

**Method:**
Use `file ./*` to check the type of every file in the directory at once.

**Concept:**
Linux can inspect a file's metadata to determine its actual type (e.g., ASCII text vs. binary), regardless of its filename — useful since a file's extension (or lack of one) doesn't reliably indicate its content.

---

## Level 5 → 6

**Goal:**
Find a file containing the password matching specific criteria: human-readable, 1033 bytes in size, not executable.

**Method:**
Use `find` in the current directory and its subdirectories, filtering with `-type f`, `-size 1033c`, and `! -executable`.

**Concept:**
Every file in Linux has characteristics (type, size, permissions) that can be filtered on. Using `find` with these flags automates the search instead of manually inspecting every file in every directory.

*(Correction from the original notes: the flag is `-type f`, not `-file f` — worth double-checking against your actual command history.)*

---

## Level 6 → 7

**Goal:**
Find a file containing the password matching specific criteria: owned by user `bandit7`, group `bandit6`, and 33 bytes in size.

**Method:**
Use `find` starting from root (`/`), searching the whole filesystem, filtering with `-user`, `-group`, and `-size`. Because it scans the entire system, it also tries to access directories without permission, producing thousands of "Permission denied" lines. Appending `2>/dev/null` discards those error messages, leaving only the actual results.

**Concept:**
Linux commands have two separate output streams: stdout (1) for normal results and stderr (2) for error messages. `2>/dev/null` discards only the error stream without touching the actual output. This stdout/stderr split applies to almost every Linux command, not just `find`.

---

## Level 7 → 8

**Goal:**
Find the word "millionth" in a large file.

**Method:**
Use `grep` to search for the word `millionth`.

**Concept:**
`grep` searches for a specific string pattern within text. It's useful for searching through files with millions of lines/characters without reading them manually.

---

## Level 8 → 9

**Goal:**
Find the line that occurs only once in a file.

**Method:**
Sort the data, then pipe it into `uniq`.

**Concept:**
`sort` arranges lines alphabetically. Piping (`|`) feeds the output of one command as input to the next. `uniq` filters out duplicate lines — but it only removes *consecutive* duplicates, which is exactly why the data has to be sorted first.

---

## Level 9 → 10

**Goal:**
Find a few human-readable strings preceded by several `=` characters.

**Method:**
Use `strings` to filter readable text out of a binary file, then pipe it to `grep` to narrow down further.

**Concept:**
The `strings` command scans a binary file and extracts sequences of printable characters (letters, numbers, punctuation) above a minimum length, filtering out everything else that isn't human-readable. That's what makes it possible to spot readable text hidden inside an otherwise unreadable binary file.

---

## Level 10 → 11

**Goal:**
Decode Base64-encoded data.

**Method:**
Use the `base64` command (with the decode flag) on the data file.

**Concept:**
Base64 is an *encoding* scheme, not encryption — it represents binary data as printable ASCII text using a fixed, publicly known algorithm with no secret key. Anyone can decode it; it's meant for safe data transport (e.g., through systems that only handle text), not confidentiality.

*(Correction from the original notes: I reworded this from "encryption" to "encoding" — worth knowing the distinction, since it's a common mix-up and this kind of precision matters for a technical role.)*

---

## Level 11 → 12

**Goal:**
Rotate letters back by 13 positions (ROT13).

**Method:**
Read the data, then transform each letter's position by shifting it 13 places through the alphabet.

**Concept:**
ROT13 is a simple substitution cipher — it shifts each letter's position in the alphabet to obscure text. Like Base64, it has no secret key and isn't meant to be secure; it's a well-known, trivially reversible scheme, more obfuscation than encryption.

---

## Level 12 → 13

**Goal:**
Decompress a file repeatedly until a readable file is retrieved.

**Method:**
Copy the data file to a temp directory, then decompress it there using `gzip`, `bzip2`, and `tar` in sequence, until it ends in a human-readable file.

**Concept:**
Compression encodes data into a smaller representation, primarily to reduce storage space and transmission time. It needs to be reversed (decompressed) before the original data can be used, and different tools/formats can be chained if a file was compressed multiple times with different methods.

---

## Level 13 → 14

**Goal:**
Log in to the next level using a private SSH key.

**Method:**
Copy the SSH key to the local machine using `scp`, change its file permissions, then use the `-i` flag when logging in to the next level.

**Concept:**
SSH supports key-based authentication as an alternative to passwords. Private keys require restrictive file permissions (SSH will refuse to use an overly permissive key file) — this connects directly to how Linux file permissions govern access.

---

## Level 14 → 15

**Goal:**
Submit the password of the current level to port 30000 on localhost.

**Method:**
Read the current level's password, then send it to port 30000 on localhost.

**Concept:**
Computers use virtual "ports" to send and receive data between two or more devices/components — a single machine can run multiple services simultaneously, each listening on its own port.

---

## Level 15 → 16

**Goal:**
Submit the current level's password to port 30001 using SSL/TLS encryption.

**Method:**
Read the current level's password, then send it to the specified port using an SSL/TLS-encrypted connection.

**Concept:**
Not all network connections are equal — some (like this one) require an encrypted connection to communicate correctly. Command flags like "quiet mode" are also useful here, suppressing noisy output so the actual result is easier to read.

---

## Level 16 → 17

**Goal:**
Find which open port returns the next level's password, rather than just echoing back the current one.

**Method:**
Scan the range of open ports, then test each one by sending the current level's password and observing the response.

**Concept:**
Ports can be scanned to discover which ones are open for connection. Different services behave differently — some just echo input back, others process it and respond with something new — so each open port has to be tested individually.

---

## Level 17 → 18

**Goal:**
Find what changed between two similar files.

**Method:**
Use `diff` to compare the two files.

**Concept:**
Files often change over time or between versions. `diff` highlights exactly what changed between two files, instead of manually comparing potentially thousands of lines line-by-line.

---

## Level 18 → 19

**Goal:**
Retrieve the password even though the SSH session gets terminated immediately after login.

**Method:**
Run `cat readme` directly from the local machine's terminal, as part of the SSH command itself, rather than logging in interactively first.

**Concept:**
SSH can execute a single remote command without opening a full interactive shell (e.g. `ssh user@host command`). This matters here because the interactive shell gets killed right after login — running the command inline avoids needing that shell at all.

---

## Level 19 → 20

**Goal:**
Use a SUID binary to escalate privileges, running it without arguments.

**Method:**
Run the level's SUID helper binary, which executes with the privileges of its owner (the next level's user) to read the password file.

**Concept:**
A SUID (Set User ID) permission bit allows an executable to run with the privileges of the file's *owner*, rather than the user who runs it — which is how this bypasses a restriction that would otherwise apply.

---

## Level 20 → 21

**Goal:**
Use a SUID binary that requires an argument this time, to connect to a port and retrieve the next password.

**Method:**
Set up a listener on a local port (running in the background so the terminal stays free for other commands), then run the level's SUID binary with that port as an argument, piping the current password in as input.

**Concept:**
Running a listener in the background frees up the terminal to run other commands at the same time — necessary here since the SUID binary and the listener both need to be active simultaneously.

---

## Level 21 → 22

**Goal:**
Investigate a cron job in `/etc/cron.d/`.

**Method:**
Go to the cron directory, list the files there, read bandit22's cron job entry, then read the script it runs. The script reads a specific file, then redirects the level 22 password into a separate output file — reading that output file gives the password.

**Concept:**
Capture-the-flag challenges often hide information in unconventional places — in this case, a scheduled task definition rather than a normal file.

---

## Level 22 → 23

**Goal:**
Investigate a cron job in `/etc/cron.d/`.

**Method:**
Go to the cron directory, list the files there, read bandit23's cron job entry, read the shell script it runs, then echo the string mentioned in that script (piped through the same command/flag the script itself uses) to find where its output file is written, and read that file.

**Concept:**
This exploits a loophole in the script's logic: echoing a specific known phrase through it produces a predictable result, and the script performs no further verification beyond checking that one expected value — providing it is enough to get the script to act, without proving any real authorization.

---

## Level 23 → 24

**Goal:**
Investigate a cron job in `/etc/cron.d/`.

**Method:**
Go to the cron directory, list the files there, read bandit24's cron job entry, read the script it runs, create a temp folder, write a script with `nano`, change that script's permissions, place the script where the cron job scans for scripts to run, then wait for the next scheduled run.

**Concept:**
The target directory has overly permissive file permissions, allowing a script to be placed there by a user who shouldn't be able to modify it. The cron job doesn't verify who created or owns the script before running it — it simply executes whatever is present at the scheduled time, using the cron job's own (higher) privileges rather than the privileges of whoever placed the script there.

*(Last sentence expanded from your notes with the standard reason this is a privilege-escalation technique — worth confirming this matches what you actually observed.)*




