# Linux Notes: Basics and Files

[Back to Linux index](README.md)

## Setup: practice environment

Needs (all free):

- VirtualBox or VMware Workstation Player
- Kali Linux VM (my attacker machine)
- Ubuntu Server or Desktop VM (my practice target; also used to practice SSH)
- Or WSL, to install Kali and Ubuntu inside Windows (easy setup, good for practicing commands)

Book: *The Linux Command Line* by William Shotts (free PDF at linuxcommand.org).

**Network setting:** In VirtualBox, set both VMs to the same Host-only Adapter (or NAT Network) so they can reach each other. Take a snapshot of each VM before starting, so you can undo mistakes.

Create a safe practice directory (in Kali or Ubuntu):

```bash
mkdir ~/practice  # create a practice directory
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

- **Filesystem:** The overall structure used by Linux to organize and manage files and directories.
- **Directory:** A location in the filesystem used to organize files and other directories. Commonly called a folder.
- **File:** A filesystem object used to store data, such as text, code, images, or other information.

## Navigating and managing files

```bash
pwd                       # print current directory
cd /etc                   # go to an absolute path
cd ..                     # go up one level
cd ~                      # go to your home directory
cd -                      # go back to the previous directory

mkdir notes               # create a directory
mkdir -p a/b/c            # create nested directories in one go
touch file1.txt           # create an empty file
cp file1.txt file2.txt    # copy a file
cp -r a/ backup/          # copy a directory
mv file2.txt renamed.txt  # rename a file
mv file.txt a/            # move a file into a directory
rm renamed.txt            # delete a file (no recycle bin!)
rm -r backup/             # delete a directory and its contents
rm -rf backup/            # forcefully delete a directory and its contents
rmdir notes               # delete an empty directory
```

- `cp file1.txt file2.txt` copies `file1.txt` to `file2.txt`. If `file2.txt` exists it is overwritten, and if it doesn't exist it is created and then filled with the copy.
- `mv file2.txt renamed.txt`: both names are in the same directory, so Linux renames `file2.txt` to `renamed.txt`.
- `mv file.txt a/`: Linux moves `file.txt` into the directory `a/`.
- `rm`: This command permanently deletes the specified file. It does not move the file to a recycle bin. Having a backup is recommended if the file is important.
- `-f` means force: suppresses most prompts and ignores nonexistent files.
- `-r` means recursive: performing an operation on a directory and everything inside it, including its subdirectories and their contents.
- `rm -rf` is dangerous because it can permanently delete files without confirmation.

### Viewing file contents

```bash
cat file.txt           # display the entire contents of a file
less /etc/passwd       # view a file interactively (q to quit)
head -n 5 /etc/passwd  # display the first 5 lines
tail -n 5 /etc/passwd  # display the last 5 lines
wc -l /etc/passwd      # count the number of lines
file /bin/ls           # identify the file type
```

### Writing to files

```bash
echo "hello" > hello.txt          # write to a file (overwrite if it exists)
echo "second line" >> hello.txt   # append to a file
nano hello.txt                    # open a file in the Nano text editor (Ctrl+O save, Ctrl+X exit)
```

- `>` Creates a file if it doesn't exist; overwrites its contents if it does.
- `>>` Creates a file if it doesn't exist; appends to it if it does.

## Important Directories

| Directory | What It Holds |
|---|---|
| `/` | Root of the entire filesystem |
| `/home` | Home directories of regular users |
| `/root` | Home directory of the root user |
| `/etc` | System configuration files (`passwd`, `shadow`, `hosts`, `ssh/`) |
| `/var/log` | System and application logs |
| `/tmp` | Temporary files (usually writable by all users) |
| `/usr/bin`, `/bin` | Executable programs and commands |
| `/opt` | Optional third-party software |
| `/proc` | Virtual filesystem containing live system and process information |
| `/dev` | Device files representing hardware and virtual devices |

- `/tmp`: files may be deleted automatically after some time or at reboot, depending on the system configuration.
- `/bin` and `/usr/bin`: on modern Kali and Ubuntu systems, `/bin` is often a symbolic link to `/usr/bin`.
- `/proc`: a virtual filesystem, not an ordinary directory of regular files.

```bash
ls /etc
cat /etc/passwd        # display user account information
sudo cat /etc/shadow   # display password hashes (root privileges required)
cat /etc/hosts         # display hostname-to-IP mappings
cat /etc/os-release    # display Linux distribution and version information
ls /var/log            # list log files and directories
```
> **⚠️ Warning:** `/etc/shadow` contains password hashes and sensitive password-related information. Never share its contents publicly or upload them to GitHub.

- `/etc/passwd` contains user account information, but it does not contain users' password hashes on modern Linux systems.
- `/etc/shadow` contains password hashes and other password-related information and is normally accessible only to privileged users.