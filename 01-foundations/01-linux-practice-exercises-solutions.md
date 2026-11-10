# Linux Fundamentals – Practical Questions and Solutions

> Solutions for the 24 practical exercises based on **linux-notes**.  
> Practice in Kali Linux or Ubuntu. Use your `~/practice` directory for file exercises.  
> Commands that modify users, services, packages, or system configuration may require `sudo`.

---

## 1. Getting Help and Flags (Switches)

**Question:** Use `man` and `--help` to find three useful flags for `ls`, `cp`, and `grep`. Explain what each flag does.

### Solution

```bash
man ls
ls --help
```

Useful `ls` flags:

- `-l` — long listing format.
- `-a` — include hidden files.
- `-h` — display sizes in human-readable units (usually with `-l`).

```bash
man cp
cp --help
```

Useful `cp` flags:

- `-r` — copy directories recursively.
- `-i` — prompt before overwriting a destination.
- `-v` — explain what is being copied.

```bash
man grep
grep --help
```

Useful `grep` flags:

- `-i` — ignore case distinctions.
- `-n` — show line numbers.
- `-r` — search recursively through files in a directory.

---

## 2. Navigating and Managing Files

**Question:** Inside `~/practice`, create `projects/web`, `projects/network`, and `notes`. Create three text files, copy one, rename one, and delete one.

### Solution

```bash
mkdir -p ~/practice/projects/web ~/practice/projects/network ~/practice/notes
cd ~/practice

touch file1.txt file2.txt file3.txt
cp file1.txt file1-copy.txt
mv file2.txt renamed-file.txt
rm file3.txt

ls -R ~/practice
```

---

## 3. Finding Things: `find`, `locate`, `grep`, `which`

**Question:** Find every `.txt` file inside your home directory, search for `root` in `/etc/passwd`, and locate Python 3.

### Solution

```bash
find ~ -type f -name "*.txt"
grep "root" /etc/passwd
which python3
```

Alternative for locating a command:

```bash
command -v python3
```

---

## 4. Pipes, Redirection and Text Processing

**Question:** Extract usernames from `/etc/passwd`, sort them alphabetically, save them in `users.txt`, and count them.

### Solution

```bash
cut -d ":" -f 1 /etc/passwd | sort > ~/practice/users.txt
wc -l ~/practice/users.txt
cat ~/practice/users.txt
```

Explanation:

- `cut -d ":" -f 1` extracts the first colon-separated field (username).
- `sort` sorts the usernames.
- `>` writes the output to a file, overwriting it if it already exists.
- `wc -l` counts lines.

---

## 5. Users, Groups and Permissions

**Question:** Create `script.sh`, give its owner read/write/execute and group/others read/execute permissions, then verify.

### Solution

```bash
cd ~/practice
touch script.sh
chmod 755 script.sh
ls -l script.sh
```

Expected permission pattern:

```text
-rwxr-xr-x
```

`755` means owner `rwx` (7), group `r-x` (5), and others `r-x` (5).

---

## 6. Important Directories

**Question:** Explore `/etc`, `/var/log`, `/tmp`, `/home`, and `/proc`. Identify their contents and display Linux distribution information.

### Solution

```bash
ls /etc
ls /var/log
ls /tmp
ls /home
ls /proc
cat /etc/os-release
```

Typical purposes:

- `/etc` — system configuration files.
- `/var/log` — system and application logs.
- `/tmp` — temporary files.
- `/home` — normal users' home directories.
- `/proc` — virtual filesystem containing live process and kernel information.
- `/etc/os-release` — distribution name and version information.

---

## 7. SSH: Logging In to a Remote Machine

**Question:** Configure SSH on Ubuntu, connect from Kali, and transfer a text file using `scp`.

### Solution

### On Ubuntu (the SSH server)

```bash
sudo apt update
sudo apt install openssh-server -y
sudo systemctl start ssh
sudo systemctl enable ssh
sudo systemctl status ssh
ip a
```

Note Ubuntu's IP address and your Ubuntu username.

### On Kali (the SSH client)

Replace `ubuntuusername` and `192.168.56.101` with your actual username and Ubuntu IP:

```bash
ssh ubuntuusername@192.168.56.101
```

Type `yes` if prompted to trust the host the first time, then enter the Ubuntu account password. Exit the remote session with:

```bash
exit
```

### Upload a file from Kali to Ubuntu

```bash
cd ~/practice
echo "SSH file transfer test" > file.txt
scp file.txt ubuntuusername@192.168.56.101:/home/ubuntuusername/
```

### Download a file from Ubuntu to Kali

```bash
scp ubuntuusername@192.168.56.101:/home/ubuntuusername/file.txt .
```

---

## 8. Basic Networking Commands

**Question:** Display your IP, routing table, listening ports, and DNS configuration. Test connectivity to Ubuntu.

### Solution

```bash
ip a
ip route
ss -tuln
cat /etc/resolv.conf
ping -c 4 192.168.56.101
```

Replace `192.168.56.101` with the Ubuntu VM's actual IP address.

---

## 9. Processes and Services

**Question:** Find the SSH process, display running processes, check SSH status, and restart SSH.

### Solution

```bash
ps aux
ps aux | grep ssh
systemctl status ssh
sudo systemctl restart ssh
systemctl status ssh
```

The final status command verifies whether SSH is running.

---

## 10. Scheduled Tasks: Cron Jobs

**Question:** Create a cron job that writes the date and time to `/tmp/time.log` every five minutes. Verify the configuration.

### Solution

Open your user's crontab:

```bash
crontab -e
```

Add this line:

```cron
*/5 * * * * date >> /tmp/time.log
```

Save and exit the editor. Verify:

```bash
crontab -l
```

After the next five-minute interval, check the file:

```bash
cat /tmp/time.log
```

---

## 11. Process IDs and Signals in Depth

**Question:** Start `sleep 600` in the background, find its PID, pause it, resume it, and terminate it gracefully.

### Solution

```bash
sleep 600 &
pgrep sleep
```

Note the PID printed by `pgrep`. Replace `<PID>` below with that number (do not type the angle brackets):

```bash
kill -19 <PID>
ps aux | grep sleep
kill -18 <PID>
kill -15 <PID>
```

- `SIGSTOP` (19) pauses the process.
- `SIGCONT` (18) resumes it.
- `SIGTERM` (15) asks it to terminate gracefully.

You can also use `jobs` to view background jobs in the current shell.

---

## 12. Package Management

**Question:** Search for `tree`, install it, verify installation, and remove it.

### Solution

```bash
apt search tree
sudo apt install tree -y
tree --version
dpkg -l | grep tree
sudo apt remove tree
```

To remove the package and its system configuration files as well:

```bash
sudo apt purge tree
```

---

## 13. Logs

**Question:** Display the last 20 authentication log entries, find failed password attempts, and inspect SSH logs.

### Solution

On Ubuntu systems where `/var/log/auth.log` exists:

```bash
sudo tail -n 20 /var/log/auth.log
sudo grep "Failed password" /var/log/auth.log
sudo journalctl -u ssh -n 20
```

Follow authentication logs live:

```bash
sudo tail -f /var/log/auth.log
```

Press `Ctrl+C` to stop following the log.

If `auth.log` does not exist, use the system journal:

```bash
sudo journalctl -u ssh -n 20
sudo journalctl -xe
```

---

## 14. apt Repositories and Third-Party Software Sources

**Question:** Display configured apt repositories, explain GPG signing keys, and identify extra repository files.

### Solution

```bash
cat /etc/apt/sources.list
ls /etc/apt/sources.list.d/
```

On some newer Ubuntu installations, the main repository configuration may be in a `.sources` file under `/etc/apt/sources.list.d/`.

**Explanation:** A repository is a server/source from which apt obtains packages. A GPG signing key allows apt to verify package signatures and check that packages came from a trusted signing key. Only add repositories using the software vendor's official instructions.

---

## 15. Environment Variables and PATH

**Question:** Display `$HOME`, `$USER`, and `$PATH`. Create `MYVAR`, display it, and create an `ll` alias.

### Solution

```bash
echo "$HOME"
echo "$USER"
echo "$PATH"

export MYVAR="test"
echo "$MYVAR"

alias ll='ls -lah'
ll
```

These changes to `MYVAR` and the alias apply to the current shell session unless added to a startup file such as `~/.bashrc`.

---

## 16. Shell Fundamentals

**Question:** Identify your shell, distinguish a builtin from an executable, and show how single/double quotes affect `$HOME`.

### Solution

```bash
echo "$SHELL"
ps -p $$
echo "$0"
cat /etc/shells

type cd
type ls
command -v python3

echo "My home is $HOME"
echo 'My home is $HOME'
```

**Explanation:**

- `cd` is normally a shell builtin.
- `ls` is normally an external executable.
- Double quotes allow `$HOME` to expand to its value.
- Single quotes keep `$HOME` literal.

---

## 17. System Information and Identification

**Question:** Display Linux distribution, kernel version, CPU, RAM, disks, and uptime.

### Solution

```bash
cat /etc/os-release
uname -r
lscpu
free -h
lsblk
df -h
uptime
```

Additional system information:

```bash
hostnamectl
```

`/etc/os-release` identifies the distribution, while `uname -r` displays the running kernel release.

---

## 18. Compression and Archives

**Question:** Create a folder with three text files, compress it into `archive.tar.gz`, extract it elsewhere, and verify the files.

### Solution

```bash
cd ~/practice
mkdir -p archive-folder
echo "File one" > archive-folder/file1.txt
echo "File two" > archive-folder/file2.txt
echo "File three" > archive-folder/file3.txt

tar -czvf archive.tar.gz archive-folder/
mkdir extracted
tar -xzvf archive.tar.gz -C extracted

ls -R extracted
```

The recovered files should be under `extracted/archive-folder/`.

---

## 19. Shell Scripting Basics

**Question:** Write an executable Bash script that accepts a username, prints a greeting, counts 1–5, and checks `/etc/passwd`.

### Solution

Create the file:

```bash
cd ~/practice
nano hello.sh
```

Write this script:

```bash
#!/bin/bash

name="$1"
echo "Hello, $name"

for i in 1 2 3 4 5; do
    echo "Count: $i"
done

if [ -f /etc/passwd ]; then
    echo "passwd file exists"
fi
```

Save and exit nano. Make it executable and run it:

```bash
chmod +x hello.sh
./hello.sh Robi
```

`$1` represents the first argument supplied to the script.

---

## 20. Text Editors: nano and Vim

**Question:** Create a file in nano, then modify and save it in Vim.

### Solution

Create and edit using nano:

```bash
cd ~/practice
nano notes.txt
```

- Type some text.
- Press `Ctrl+O`, then Enter to save.
- Press `Ctrl+X` to exit.

Open the same file in Vim:

```bash
vim notes.txt
```

- Press `i` to enter Insert mode.
- Add or modify text.
- Press `Esc` to return to Normal mode.
- Type `:wq` and press Enter to save and quit.

---

## 21. Chaining Commands: `&&`, `||`, `;` and `&`

**Question:** Create and enter a directory with `&&`, use `||` for an error message, run commands with `;`, and start a background process.

### Solution

```bash
mkdir testdir && cd testdir && echo "made it"
```

Try a command that fails:

```bash
ls /nonexistent || echo "that folder is missing"
```

Run commands regardless of the previous command's result:

```bash
cd ~/practice ; ls ; pwd
```

Start a background process:

```bash
sleep 60 &
jobs
```

- `&&` runs the next command only after success.
- `||` runs the next command after failure.
- `;` runs the next command regardless of success or failure.
- `&` starts a command in the background.

---

## 22. Linux Boot Process

**Question:** Explain the six main boot stages and identify PID 1, default systemd target, and boot time.

### Solution

### Main stages

1. **BIOS/UEFI:** Firmware initializes basic hardware and selects a boot device.
2. **Bootloader (GRUB):** Loads the Linux kernel and initial RAM filesystem.
3. **Kernel:** Initializes hardware, memory, drivers, and core operating-system functions.
4. **Initramfs:** Helps locate and mount the real root filesystem.
5. **Init system (systemd):** Starts services and reaches the configured system target.
6. **Login:** A graphical login screen or text login prompt becomes available.

### Commands

```bash
ps -p 1 -o pid,comm,args
systemctl get-default
systemctl list-units --type=target
who -b
last reboot
uname -r
ls /boot
```

View current boot logs and startup timing:

```bash
journalctl -b
systemd-analyze
systemd-analyze blame
```

---

## 23. Disk Management

**Question:** Display disks, partitions, mounted filesystems, free storage, and swap. Explain disk, partition, and mount point.

### Solution

```bash
lsblk
lsblk -f
sudo fdisk -l
sudo blkid
df -h
findmnt
swapon --show
free -h
```

Check the space used by your practice directory:

```bash
du -sh ~/practice
```

**Terms:**

- **Disk:** A complete storage device, such as a VirtualBox virtual disk.
- **Partition:** A defined section of a disk.
- **Mount point:** A directory through which a filesystem is accessed.
- **Filesystem:** The structure used to organize and store files.
- **Swap:** Disk space Linux can use as backing store for memory pages.

---

## 24. Linux File Systems

**Question:** Identify your root filesystem type, display its UUID, find a file's inode, and explain `/etc/fstab`.

### Solution

```bash
findmnt /
df -hT /
lsblk -f
sudo blkid
```

Find the inode number and metadata of a practice file:

```bash
cd ~/practice
touch inode-test.txt
ls -i inode-test.txt
stat inode-test.txt
df -i
```

View the filesystem configuration:

```bash
cat /etc/fstab
```

**Explanation:**

- A **filesystem** organizes and stores files and directories.
- An **inode** stores metadata about a filesystem object, such as permissions, ownership, timestamps, and data-block locations.
- A **UUID** identifies a filesystem.
- `/etc/fstab` describes filesystems and how they should be mounted, including mount points, filesystem types, and mount options.

Do not copy example UUIDs or placeholder mount entries into `/etc/fstab`. Incorrect entries can cause boot or mounting problems.

---