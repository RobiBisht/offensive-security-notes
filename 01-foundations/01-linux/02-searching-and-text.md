# Linux Notes: Searching and Text Processing

[Back to Linux index](README.md)

## Finding things: find, locate, grep, which

```bash
find / -name "passwd" 2>/dev/null      # find a file by name
find /home -type f -name "*.txt"       # find all .txt files
find / -type f -size +10M 2>/dev/null  # files larger than 10 MB
find / -perm -4000 2>/dev/null         # find SUID files
find / -user root -type f 2>/dev/null  # files owned by root

locate passwd                          # fast search (run: sudo updatedb first)
which python3                          # where is this command located

grep "root" /etc/passwd                # lines containing root
grep -i "error" file.txt               # case-insensitive
grep -r "password" ~/practice          # search inside all files in a folder
grep -n "bash" /etc/passwd             # show line numbers
grep -v "nologin" /etc/passwd          # show lines that do NOT match
```

- `2>/dev/null` hides permission errors so the output stays readable.
- `find / -perm -4000 2>/dev/null` finds SUID files, which are important for privilege escalation. (Research permission bits to know more about this.)

## Pipes, redirection and text processing

A pipe (`|`) sends the output of one command into another. This is one of the most useful Linux skills for hacking.

```bash
cat /etc/passwd | grep "bash"        # users with a bash shell
ls -l /etc | wc -l                   # count items in /etc
cut -d ":" -f 1 /etc/passwd          # print only usernames (first field)
cut -d ":" -f 1,7 /etc/passwd        # username and shell
sort names.txt                       # sort lines
sort names.txt | uniq                # remove duplicate lines
sort names.txt | uniq -c | sort -nr  # count and rank duplicates
awk -F: '{print $1}' /etc/passwd     # same as cut, using awk
sed 's/bash/zsh/' file.txt           # replace text in output (does not change the file)
tr 'a-z' 'A-Z' < file.txt            # convert to uppercase
```

### Redirection

```bash
command > file     # write output to a file (overwrite)
command >> file    # append new output at the end of the file without deleting existing content
command 2> errors  # write only errors to a file
command &> all     # write output and errors
command < file     # read input from a file
```

## Chaining commands: &&, ||, ; and &

Operators let me control when the next command runs.

| Operator | Meaning |
|---|---|
| `&&` | Run the second command only if the first one succeeds |
| `\|\|` | Run the second command only if the first one fails |
| `;` | Run the second command no matter what happened to the first |
| `&` | Run the command in the background |

```bash
mkdir testdir && cd testdir && echo "made it"     # each step runs only if the last one worked
mkdir testdir && echo "created"                   # second time: mkdir fails, so echo does NOT run
ls /nonexistent || echo "that folder is missing"  # echo runs because ls failed
cd ~/practice ; ls ; pwd                          # all three run, one after another
sleep 60 &                                        # runs in the background
```

(In the table, `\|\|` is how I write `||` so the table doesn't break. In the terminal I type `||`.)

### Combine them for useful patterns

```bash
ping -c 1 192.168.56.101 &> /dev/null && echo "host is up" || echo "host is down"
sudo apt update && sudo apt upgrade -y  # upgrade only if the update worked
```

To see whether the last command succeeded, check the exit status: `0` means success and anything else means failure.

```bash
ls /etc
echo $?  # prints 0
ls /nonexistent
echo $?  # prints a non-zero number (2)
```
