# Linux Practice Exercises – Solutions

## 1. Using man pages for ls, cp, and grep

Use the `man` command to read manual pages.

```bash
man ls
man cp
man grep
```

Press `q` to exit the manual.

### Useful flags

**ls**
- `-l` – Display detailed information.
- `-a` – Show hidden files.
- `-h` – Display sizes in human-readable format when used with `-l`.

**cp**
- `-r` – Copy directories recursively.
- `-i` – Ask before overwriting.
- `-v` – Show copied files.

**grep**
- `-i` – Ignore case.
- `-n` – Display line numbers.
- `-r` – Search recursively inside directories.

---

## 2. Creating directories and managing files

Create the required directory structure:

```bash
mkdir -p ~/practice/projects/web
mkdir -p ~/practice/projects/network
mkdir -p ~/practice/notes
```

Enter the practice directory:

```bash
cd ~/practice
```

Create three text files:

```bash
touch file1.txt file2.txt file3.txt
```

Write some content:

```bash
echo "Web development notes" > file1.txt
echo "Network security notes" > file2.txt
echo "Linux notes" > file3.txt
```

Copy one file:

```bash
cp file1.txt file1-copy.txt
```

Rename one file:

```bash
mv file2.txt network-notes.txt
```

Delete one file:

```bash
rm file3.txt
```

Verify:

```bash
ls -l
tree ~/practice
```

---

## 3. Finding .conf files and SUID binaries

### Find all .conf files

```bash
sudo find / -type f -name "*.conf" 2>/dev/null
```

Explanation:
- `/` – Start searching from the root directory.
- `-type f` – Search only regular files.
- `-name "*.conf"` – Match names ending in `.conf`.
- `2>/dev/null` – Hide permission-denied messages.

### Find SUID binaries

```bash
sudo find / -type f -perm -4000 2>/dev/null
```

Display five SUID binaries:

```bash
sudo find / -type f -perm -4000 2>/dev/null | head -n 5
```

SUID files execute with the effective permissions of their file owner.

Examples commonly found on Linux include:

- `/usr/bin/passwd`
- `/usr/bin/su`
- `/usr/bin/sudo`
- `/usr/bin/chsh`
- `/usr/bin/chfn`

The actual list depends on the distribution and installed packages. Use your command output as the authoritative list.

---

## 4. Printing sorted usernames into users.txt

Extract usernames from `/etc/passwd`, sort them, and save them:

```bash
cut -d: -f1 /etc/passwd | sort > users.txt
```

Display the file:

```bash
cat users.txt
```

Explanation:
- `cut` – Extract selected fields.
- `-d:` – Use colon as the delimiter.
- `-f1` – Extract the first field.
- `sort` – Sort alphabetically.
- `>` – Save output into a file.

---

## 5. File permissions and testing another user

Create the file:

```bash
cd ~/practice
touch secret.txt
```

Set permissions so only the owner can read and write:

```bash
chmod 600 secret.txt
```

Verify:

```bash
ls -l secret.txt
```

Expected permissions:

```text
-rw------- 1 username username ... secret.txt
```

Create a second user:

```bash
sudo adduser testuser
```

Switch to that user:

```bash
su - testuser
```

Try reading the file:

```bash
cat /home/yourusername/practice/secret.txt
```

Expected result:

```text
Permission denied
```

Replace `yourusername` with your actual username.

Return to your original user:

```bash
exit
```

**Note:** If the parent directory permissions prevent traversal, the second user may receive permission denied even before reaching the file. This is also a valid access-control result.

---

## 6. Why /etc/shadow is restricted but /etc/passwd is readable

### /etc/passwd

The `/etc/passwd` file contains account information such as:

- Username
- User ID (UID)
- Group ID (GID)
- Home directory
- Login shell

It is readable by ordinary users because Linux applications need to look up usernames and account information.

### /etc/shadow

The `/etc/shadow` file contains password hashes and password-aging information.

It is restricted because password hashes must not be exposed to ordinary users. If attackers obtain these hashes, they may attempt offline password guessing.

Therefore:

- `/etc/passwd` is generally readable by everyone.
- `/etc/shadow` is normally readable only by root or authorized privileged processes.

Check permissions:

```bash
ls -l /etc/passwd /etc/shadow
```

---

## 7. SSH from Kali to Ubuntu and SCP file transfer

### On Ubuntu

Install SSH server:

```bash
sudo apt update
sudo apt install openssh-server
```

Start SSH:

```bash
sudo systemctl enable --now ssh
```

Find Ubuntu IP:

```bash
ip -br addr
```

### From Kali

Connect to Ubuntu:

```bash
ssh ubuntu_username@ubuntu_ip
```

Create a file on Ubuntu:

```bash
echo "File created on Ubuntu" > remote.txt
```

Exit SSH:

```bash
exit
```

Download the file to Kali:

```bash
scp ubuntu_username@ubuntu_ip:~/remote.txt ~/remote.txt
```

Verify:

```bash
cat ~/remote.txt
```

### SSH key authentication

On Kali, generate a key:

```bash
ssh-keygen -t ed25519
```

Copy the public key to Ubuntu:

```bash
ssh-copy-id ubuntu_username@ubuntu_ip
```

Test key login:

```bash
ssh ubuntu_username@ubuntu_ip
```

### Disable password authentication on Ubuntu

First take a VirtualBox snapshot.

Edit SSH configuration:

```bash
sudo vim /etc/ssh/sshd_config
```

Set:

```text
PubkeyAuthentication yes
PasswordAuthentication no
```

Validate configuration:

```bash
sudo sshd -t
```

Restart SSH:

```bash
sudo systemctl restart ssh
```

Test key login from Kali.

**Important:** Keep your existing SSH session open until a new key-based session successfully connects. Otherwise, you could lock yourself out.

---

## 8. Background process and killing it

Start a background process:

```bash
sleep 600 &
```

Find its PID:

```bash
ps aux | grep sleep
```

A cleaner alternative:

```bash
pgrep -a sleep
```

Kill the process:

```bash
kill PID
```

Replace `PID` with the actual process ID.

Verify:

```bash
ps aux | grep sleep
```

---

## 9. Creating and removing a cron job

Open your user's crontab:

```bash
crontab -e
```

Add:

```cron
* * * * * date >> /tmp/cron-test.txt
```

This appends the current date and time every minute.

Wait approximately two minutes.

Check the file:

```bash
cat /tmp/cron-test.txt
```

Remove the job:

```bash
crontab -e
```

Delete the cron line and save.

Verify:

```bash
crontab -l
```

---

## 10. Pause one process and kill another using different signals

Start two processes:

```bash
sleep 600 &
sleep 600 &
```

Find their PIDs:

```bash
pgrep -a sleep
```

Assume the PIDs are `1111` and `2222`.

Pause the first process:

```bash
kill -STOP 1111
```

Check:

```bash
ps -o pid,stat,cmd -p 1111
```

The process state should contain `T`, meaning stopped.

Resume it:

```bash
kill -CONT 1111
```

Kill the second process with SIGTERM:

```bash
kill -TERM 2222
```

Verify:

```bash
ps aux | grep sleep
```

Signal meanings:

- `SIGSTOP` – Stop a process immediately.
- `SIGCONT` – Continue a stopped process.
- `SIGTERM` – Request graceful termination.
- `SIGKILL` – Force immediate termination.

---

## 11. Installing and removing tree

Install:

```bash
sudo apt update
sudo apt install tree
```

Display the practice structure:

```bash
tree ~/practice
```

Remove the package:

```bash
sudo apt remove tree
```

---

## 12. Failed SSH login attempts and log analysis

From Kali, attempt SSH login to Ubuntu using an incorrect password twice:

```bash
ssh ubuntu_username@ubuntu_ip
```

Enter an incorrect password two times.

### On Ubuntu

Search authentication logs:

```bash
sudo grep "Failed password" /var/log/auth.log
```

Display the latest failed attempts:

```bash
sudo grep "Failed password" /var/log/auth.log | tail
```

Filter by SSH:

```bash
sudo grep sshd /var/log/auth.log | grep "Failed password"
```

Typical log information includes:

- Timestamp
- SSH service name
- Username attempted
- Source IP address
- Authentication failure message

On systems using journald without `/var/log/auth.log`, use:

```bash
sudo journalctl -u ssh
```

---

## 13. Understanding APT sources

Read the main sources file:

```bash
cat /etc/apt/sources.list
```

List source files:

```bash
ls -l /etc/apt/sources.list.d/
```

Read source files:

```bash
cat /etc/apt/sources.list.d/*.list
```

For Deb822 source files:

```bash
cat /etc/apt/sources.list.d/*.sources
```

### Example traditional source

```text
deb http://archive.ubuntu.com/ubuntu noble main universe
```

Explanation:

- `deb` – Binary package repository.
- `http://archive.ubuntu.com/ubuntu` – Repository URL.
- `noble` – Ubuntu release codename.
- `main` – Officially supported open-source software.
- `universe` – Community-maintained open-source software.

Other common components:

- `restricted` – Software with restricted licensing or distribution conditions.
- `multiverse` – Software with licensing or other restrictions.

### Example Kali source

```text
deb http://http.kali.org/kali kali-rolling main contrib non-free non-free-firmware
```

Explanation:

- `deb` – Binary packages.
- `http://http.kali.org/kali` – Repository URL.
- `kali-rolling` – Kali rolling release.
- `main` – Main Kali repository.
- `contrib` – Contributions depending on packages outside main.
- `non-free` – Non-free software.
- `non-free-firmware` – Non-free firmware packages.

Update package information:

```bash
sudo apt update
```

---

## 14. Bash script to check file existence, size, and permissions

Create the script:

```bash
nano file_info.sh
```

Write:

```bash
#!/bin/bash

file="$1"

if [ -z "$file" ]; then
    echo "Usage: $0 filename"
    exit 1
fi

if [ -e "$file" ]; then
    echo "File exists: $file"
    echo "Size: $(stat -c %s "$file") bytes"
    echo "Permissions: $(stat -c %A "$file")"
else
    echo "File does not exist: $file"
fi
```

Save the file.

Make executable:

```bash
chmod +x file_info.sh
```

Run:

```bash
./file_info.sh secret.txt
```

Example output:

```text
File exists: secret.txt
Size: 0 bytes
Permissions: -rw-------
```

Test a nonexistent file:

```bash
./file_info.sh missing.txt
```

---

## 15. Creating and executing a shell script using Vim

Open Vim:

```bash
vim hello.sh
```

Press `i` to enter insert mode.

Write:

```bash
#!/bin/bash
echo "Hello, Linux!"
echo "This is my first script."
```

Press `Esc`.

Save and exit:

```vim
:wq
```

Make executable:

```bash
chmod +x hello.sh
```

Run:

```bash
./hello.sh
```

### Safely opening /etc/passwd

```bash
vim /etc/passwd
```

Quit without saving:

```vim
:q!
```

---

## 16. Creating a directory, entering it, and creating a file in one line

Use:

```bash
mkdir myfolder && cd myfolder && touch myfile.txt
```

Explanation:

- `mkdir myfolder` – Create directory.
- `&&` – Execute the next command only if the previous command succeeds.
- `cd myfolder` – Enter the directory.
- `touch myfile.txt` – Create a file.

This stops cleanly if any command fails.

---

## 17. Identifying the current shell

Display the current shell process:

```bash
ps -p $$ -o comm=
```

Display the SHELL environment variable:

```bash
echo "$SHELL"
```

Example:

```text
bash
/bin/bash
```

Note that `$SHELL` normally shows your configured login shell, while `ps` shows the shell process currently running.

---

## 18. Displaying system information

### Linux distribution

```bash
cat /etc/os-release
```

Short version:

```bash
lsb_release -a
```

### Kernel version

```bash
uname -r
```

Complete kernel information:

```bash
uname -a
```

### CPU information

```bash
lscpu
```

Detailed CPU information:

```bash
cat /proc/cpuinfo
```

### RAM usage

```bash
free -h
```

### System uptime

```bash
uptime
```

Human-readable uptime:

```bash
uptime -p
```

---

## 19. Checking boot messages and identifying the service manager

Display boot messages:

```bash
sudo dmesg
```

Display the latest messages:

```bash
sudo dmesg | tail -n 30
```

Display boot logs:

```bash
journalctl -b
```

Identify the service manager:

```bash
ps -p 1 -o comm=
```

Check systemd:

```bash
systemctl --version
```

On Kali Linux and Ubuntu, the service manager is normally:

```text
systemd
```

Systemd manages services, startup processes, and system initialization.

---

## 20. Inspecting disks, partitions, and mounted filesystems

### lsblk

```bash
lsblk
```

Detailed output:

```bash
lsblk -f
```

Shows:

- Disk devices
- Partitions
- Filesystem types
- UUIDs
- Mount points

### df -h

```bash
df -h
```

Shows filesystem disk usage in human-readable units.

### findmnt

```bash
findmnt
```

Display the root filesystem:

```bash
findmnt /
```

Display filesystem type and mount options:

```bash
findmnt -o TARGET,SOURCE,FSTYPE,OPTIONS
```

---

## 21. Identifying the root filesystem and explaining mount points

### Identify root filesystem type

Run:

```bash
findmnt -no SOURCE,FSTYPE /
```

Example output:

```text
/dev/sda2 ext4
```

This means:

- `/dev/sda2` – Device containing the root filesystem.
- `ext4` – Filesystem type.

Your system might use ext4, btrfs, xfs, or another filesystem.

### What is a mount point?

A mount point is a directory where a filesystem is attached and made accessible within the Linux directory hierarchy.

Examples:

- `/` – Root filesystem.
- `/home` – User files, if separately mounted.
- `/boot` – Boot files, if separately mounted.
- `/mnt` – Common temporary mount location.
- `/media` – Common location for removable media.

Linux combines multiple filesystems into a single directory tree rather than assigning a separate drive letter to each filesystem.

Example:

```text
/
├── home
├── boot
├── etc
├── usr
├── var
└── mnt
```

A filesystem mounted at `/home` becomes accessible through the `/home` directory.

Check mount points:

```bash
findmnt
```

Check the root filesystem:

```bash
df -h /
```

---