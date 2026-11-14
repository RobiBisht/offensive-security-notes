# Linux Notes: Basics and Files

[Back to Linux index](README.md)

## Setup: practice environment

What I need (all free):

- VirtualBox or VMware Workstation Player
- Kali Linux VM (my attacker machine)
- Ubuntu Server or Desktop VM (my practice target; also used to practice SSH)
- Or WSL, to install Kali and Ubuntu inside Windows (easy setup, good for practicing commands)

Book: *The Linux Command Line* by William Shotts (free PDF at linuxcommand.org).

**Network setting:** In VirtualBox, set both VMs to the same Host-only Adapter (or NAT Network) so they can reach each other. Take a snapshot of each VM before starting, so I can undo mistakes.

Create a safe practice folder (in Kali or Ubuntu):

```bash
mkdir ~/practice  # create a practice folder
cd ~/practice     # do all the exercises inside this folder so I do not break anything
```

The **man pages** (manual pages) are the built-in Linux documentation that explains how to use commands. On my VM I type `man ls` and press `q` to quit.

## Getting help and flags (switches)

Most commands accept flags that change their behavior. A flag usually starts with `-` (short option) or `--` (long option).

```bash
ls         # list files
ls -l      # long format (permissions, owner, size, date)
ls -a      # show hidden files (names starting with .)
ls -lah    # combine flags: long, all, human-readable sizes
ls --help  # quick help
man ls     # full manual (press q to quit)
```

## Navigating and managing files

```bash
pwd                       # print current directory
cd /etc                   # go to an absolute path
cd ..                     # go up one level
cd ~                      # go to your home directory
cd -                      # go back to the previous directory

mkdir notes               # create a folder
mkdir -p a/b/c            # create nested folders in one go
touch file1.txt           # create an empty file
cp file1.txt file2.txt    # copy a file
cp -r a/ backup/          # copy a folder
mv file2.txt renamed.txt  # move or rename
rm renamed.txt            # delete a file (no recycle bin!)
rm -r backup/             # delete a folder and its contents
rmdir notes               # delete an empty folder
```

- `cp file1.txt file2.txt` copies `file1.txt` to `file2.txt`. If `file2.txt` exists it is overwritten, and if it doesn't exist it is created and then filled with the copy.
- `-r` means recursive: performing an operation on a directory and everything inside it, including its subdirectories and their contents.

### Viewing file contents

```bash
cat file.txt           # print the whole file
less /etc/passwd       # scroll through a file (q to quit)
head -n 5 /etc/passwd  # print first 5 lines
tail -n 5 /etc/passwd  # print last 5 lines
wc -l /etc/passwd      # count lines
file /bin/ls           # what type of file is this
```

### Writing to files

```bash
echo "hello" > hello.txt         # > writes (overwrites the file)
echo "second line" >> hello.txt  # >> appends
nano hello.txt                   # simple text editor (Ctrl+O save, Ctrl+X exit)
```

- `>` overwrites the file.
- `>>` appends new content at the end of the file without deleting existing content.

## Important directories

| Directory | What it holds |
|---|---|
| `/` | Root of the whole file system |
| `/home` | Normal users' personal folders |
| `/root` | Home folder of the root user |
| `/etc` | System configuration files (`passwd`, `shadow`, `hosts`, `ssh/`) |
| `/var/log` | System and application logs |
| `/tmp` | Temporary files (anyone can write here) |
| `/usr/bin`, `/bin` | Command programs |
| `/opt` | Optional third-party software |
| `/proc` | Live system and process information |
| `/dev` | Device files |

```bash
ls /etc
cat /etc/passwd       # list of users
sudo cat /etc/shadow  # hashed passwords (root only)
cat /etc/hosts
cat /etc/os-release   # which distro and version
ls /var/log
```
