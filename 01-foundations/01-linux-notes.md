# Linux Fundamental Notes.

Setup Your Practice Environment

What you need (all free):

•	VirtualBox or VMware Workstation Player
•	Kali Linux VM (your attacker machine)
•	Ubuntu Server or Desktop VM (your practice target; also use it to practice SSH)
•      You Can Use WSL And Install Kali & Ubuntu To Use Linux In Windows, Easy Setup Good For Commands.

Book :

•	The Linux Command Line by William Shotts.

Network setting: In VirtualBox, set both VMs to the same Host-only Adapter (or NAT Network) so they can reach each other. Take a snapshot of each VM before you start, so you can undo mistakes.

Create a safe practice folder (in Kali or Ubuntu):

```bash
mkdir ~/practice  # create a practice folder.

cd ~/practice     # do all the exercises inside this folder so you do not break anything.
```

The Man Pages (Manual Pages) are built-in Linux documentation that explains how to use Linux commands. on your VM (type man ls; press q to quit)


## 1. Getting Help and Flags (Switches)

Most commands accept flags that change their behavior. A flag usually starts with - (short options) or -- (long options).

```bash
ls         # list files
ls -l      # long format (permissions, owner, size, date)
ls -a      # show hidden files (names starting with .)
ls -lah    # combine flags: long, all, human-readable sizes
ls --help  # quick help
man ls     # full manual (press q to quit)
```


## 2. Navigating and Managing Files

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

(cp copy file1.txt to file2.txt, if file2.txt exists it overwrites it, if it doesn't exist it creates one and then copy)
(-r means recursive, means performing an operation on a directory and everything inside it, including its subdirectories and their contents.)

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
echo "hello" > hello.txt         # `>` write 
echo "second line" >> hello.txt  # `>>` appends 
nano hello.txt                   # simple text editor (Ctrl+O save, Ctrl+X exit)
```

'>'(overwrites the file)
'>>' appends new content at the end of the file without deleting existing content.


## 3. Finding Things: find, locate, grep, which

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
2>/dev/null hides permission errors so the output stays readable.

find / -perm -4000 2>/dev/null           (important for privilege escalation) [Research Permission Bits To Know More About]


## 4. Pipes, Redirection and Text Processing

A pipe (|) sends the output of one command into another. This is one of the most useful Linux skills for hacking.

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
command >> file    # append new output at the end of the file without deleting existing content.
command 2> errors  # write only errors to a file
command &> all     # write output and errors
command < file     # read input from a file
```


## 5. Users, Groups and Permissions

### Read permissions

```bash
ls -l file.txt
```

```text
-rwxr-xr-- 1 kali kali 120 Oct 27 10:00 file.txt
```

Breakdown: the first character is the type (- file, d directory), then three sets of rwx for owner, group, others. r=read [4(numeric value)], w=write [2], x=execute [1], So its like this -(type)rwx(owner)r-x(groups)r--(others), owner have read,write\&execute permission, group have read \& execute permission, others have only read permission, permission bits written as this rwx so when you see r-x, r--, --x, ---, every - (hyphen) means that permission not granted. (For more information about Permission Bits research It, permission bits are necessary to understand for further study).

### Changing permissions

```bash
chmod 755 script.sh   # owner rwx, group r-x, others r-x (rwx have theie numeric values r=4 w=2 x=1
chmod 644 notes.txt   # owner rw-, group r--, others r--
chmod +x script.sh    # add execute for everyone
chmod u+x,g-w file    # symbolic: user plus x (add execute for user), group minus w (minus write for group)
chmod -R 750 folder/  # recursively
```

### Ownership

```bash
chown user file.txt        # change owner (needs sudo)
chown user:group file.txt  # change owner and group
id                         # your user id and groups
whoami                     # current user
groups                     # your groups
```

### Users and sudo

```bash
sudo adduser testuser           # create a user (set a password when asked)
sudo usermod -aG sudo testuser  # add to the sudo group
su testuser                     # switch user
sudo -l                         # list what you can run with sudo
sudo passwd testuser            # change a password
sudo deluser testuser           # delete the user
```

### Special permissions (very important for privilege escalation)

•	SUID (s in the owner position, like -rwsr-xr-x): the program runs with the owner's rights. Example: /usr/bin/passwd.
•	SGID: runs with the group's rights.
•	Sticky bit (t, like drwxrwxrwt on /tmp): users can only delete their own files in the folder.

```bash
ls -l /usr/bin/passwd
ls -ld /tmp
chmod u+s file  # set SUID (for learning only)
```


## 6. Important Directories

Directory	                                       What it holds

/                               	 # Root of the whole file system
/home             	         # Normal users' personal folders
/root                 	         # Home folder of the root user
/etc                 	                 # System configuration files (passwd, shadow, hosts, ssh/)
/var/log            	         # System and application logs
/tmp                           	 # Temporary files (anyone can write here)
/usr/bin, /bin	                 # Command programs
/opt	                                 # Optional third-party software
/proc	                         # Live system and process information
/dev	                                 # Device files

```bash
ls /etc
cat /etc/passwd       # list of users
sudo cat /etc/shadow  # hashed passwords (root only)
cat /etc/hosts
cat /etc/os-release   # which distro and version
ls /var/log
```


## 7. SSH: Logging In to a Remote Machine

SSH lets you control another machine's terminal securely. Practice it between your Kali and Ubuntu VMs.

On the Ubuntu VM (the machine you connect to):

```bash
sudo apt update                     # refresh the terminal
sudo apt install openssh-server -y  # install SSH
sudo systemctl start ssh            # start SSH
sudo systemctl enable ssh           # enable SSH
sudo systemctl status ssh           # status ## should show active (running)
ip a                                # note the IP address (for example 192.168.56.101)
```

On the Kali VM (the machine you connect from):

```bash
ssh ubuntuusername@192.168.56.101  # use you ubantu login username
```

Type yes the first time, then enter the password. Your prompt will change to the remote machine. Type exit to leave.

Copy files with scp:

```bash
scp file.txt user@192.168.56.101:/home/user/   # upload
scp user@192.168.56.101:/home/user/file.txt .  # download
```

Key-based login (more secure and a good thing to learn):

```bash
ssh-keygen -t ed25519
ssh-copy-id user@192.168.56.101
ssh user@192.168.56.101  # now works without a password
```


## 8. Basic Networking Commands

```bash
ip a                              # your IP addresses and interfaces
ip route                          # routing table and default gateway
ping -c 4 192.168.56.101          # test connectivity (4 packets)
ss -tuln                          # open listening ports
sudo ss -tulnp                    # include the program using each port
hostname                          # machine name
cat /etc/resolv.conf              # DNS settings
curl http://example.com           # fetch a web page
wget http://example.com/file.txt  # download a file
```

Start a quick web server to share files between your VMs:

```bash
cd ~/practice
python3 -m http.server 8000
```

Then from the other VM run wget http://IP-OF-THIS-VM:8000/file.txt. You will use this exact trick in real CTFs and pentests to transfer files.


## 9. Processes and Services

### Viewing processes

```bash
ps                 # processes associated with your current terminal
ps aux             # all processes on the system
ps aux | grep ssh  # find a specific process
top                # live view (q to quit)
htop               # nicer version (sudo apt install htop)
```

### Controlling processes

```bash
sleep 300 &   # run a command in the background
jobs          # list background jobs
fg %1         # bring job 1 to the foreground
kill 1234     # stop the process with PID 1234
kill -9 1234  # force-kill
pkill sleep   # kill by name
```

### Services with systemctl

```bash
systemctl status ssh
sudo systemctl stop ssh
sudo systemctl start ssh
sudo systemctl restart ssh
sudo systemctl enable ssh   # start at boot
sudo systemctl disable ssh  # do not start at boot
systemctl list-units --type=service
```


## 10. Scheduled Tasks: Cron Jobs

Cron runs commands on a schedule. Attackers also abuse misconfigured cron jobs, so understanding them matters.

```bash
crontab -e      # edit your cron jobs
crontab -l      # list your cron jobs
ls /etc/cron.d  # system cron jobs
cat /etc/crontab
```

Format: minute hour day-of-month month day-of-week command

```bash
* * * * * echo "hi" >> /tmp/cron-test.txt  # every minute
0 2 * * * /home/user/backup.sh             # every day at 2:00
*/5 * * * * date >> /tmp/time.log          # every 5 minutes
0 9 * * 1 /home/user/weekly.sh             # Mondays at 9:00
```


## 11. Process IDs and Signals in Depth

Every running process has a PID (process ID) and a PPID (its parent's ID). The first process, PID 1 (systemd or init), starts all the others.

```bash
ps -ef             # all processes with PID and PPID
ps aux | grep ssh  # find a process
pgrep ssh          # print the PID of processes named ssh
pstree             # show the parent and child tree
echo $$            # the PID of your current shell
ls /proc/$$        # the live data Linux keeps about that process
```

Signals are messages sent to processes. kill does not only kill; it sends any signal.

```bash
kill -l        # list all signals
kill -15 1234  # SIGTERM: politely ask the process to stop (the default)
kill -9 1234   # SIGKILL: force it to stop immediately, no cleanup
kill -19 1234  # SIGSTOP: pause the process
kill -18 1234  # SIGCONT: resume a paused process
kill -1 1234   # SIGHUP: hang up; many services reload their config
```

| Signal | Number | Meaning |
|---------|--------|---------|
| SIGHUP | 1 | Terminal closed or reload configuration |
| SIGINT | 2 | Interrupt (Ctrl + C) |
| SIGTERM | 15 | Polite request to terminate|
| SIGKILL | 9 | Force kill |
| SIGSTOP | 19 | Pause a process |
| SIGCONT | 18 | Resume a process |

Keyboard shortcuts in a terminal:

•	Ctrl + C stops the foreground process (SIGINT)
•	Ctrl + Z pauses it and returns you to the shell
•	Then bg resumes it in the background, or fg brings it back to the front

Try this to see signals in action:

```bash
sleep 600 &          # start a background process
pgrep sleep          # note its PID
kill -19 <PID>       # pause it
ps aux | grep sleep  # the state column shows T (stopped)
kill -18 <PID>       # resume it
kill -15 <PID>       # stop it politely
```

To keep a process running after you close the terminal, start it with nohup:

```bash
nohup python3 -m http.server 8000 &
```
Use kill -15 first and only use kill -9 when a process will not stop, because -9 skips cleanup such as closing files.


## 12. Package Management

On Debian-based systems (Kali, Ubuntu) use apt.

```bash
sudo apt update           # refresh the package list
sudo apt upgrade -y       # upgrade installed packages
sudo apt install tree -y  # install a package
sudo apt remove tree      # remove it
sudo apt purge tree       # remove it and its config
apt search nmap           # search packages
apt show nmap             # package details
dpkg -l | grep nmap       # check whether it is installed
sudo dpkg -i package.deb  # install a downloaded .deb file
```

## 13. Logs

Logs show what happened on the system. As a hacker you read them to understand detection; as a defender you read them to find attacks.

```bash
ls /var/log
sudo tail -n 20 /var/log/syslog    # general system log (Ubuntu)
sudo tail -n 20 /var/log/auth.log  # logins and sudo use (Ubuntu with rsyslog)
sudo tail -f /var/log/auth.log     # follow live (Ctrl+C to stop)
sudo journalctl -u ssh -n 20       # logs for the ssh service
sudo journalctl -xe                # recent logs with explanations
sudo grep "Failed password" /var/log/auth.log
last                               # login history
```
If /var/log/auth.log does not exist, your system uses the systemd journal, so use journalctl instead.


## 14. apt Repositories and Third-Party Software Sources

apt installs software from repositories: servers that hold packages. Your system lists them in:

```bash
cat /etc/apt/sources.list    # main list of repositories
ls /etc/apt/sources.list.d/  # extra repository files
```
Kali's repositories are set up for security tools. On Ubuntu you may want software that is not in the default repositories, such as a vendor's own tools. Adding a third-party source has three parts:

1.	Download the vendor's GPG signing key. This lets apt check that packages really come from them.
2.	Add a source entry that points to their repository.
3.	Run sudo apt update, then install the package.

### 1. Save the vendor's key (the URL and names below are placeholders; use the vendor's official instructions)

```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://example.com/vendor-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/vendor.gpg
```

### 2. Add the repository, tied to that key

```bash
echo "deb [signed-by=/etc/apt/keyrings/vendor.gpg] https://example.com/apt stable main" | sudo tee /etc/apt/sources.list.d/vendor.list
```

### 3. Refresh and install

```bash
sudo apt update
sudo apt install vendor-package
```
On Ubuntu, a PPA (Personal Package Archive) is a shortcut:

```bash
sudo add-apt-repository ppa:name/ppa
sudo apt update
```
To remove a source: delete its file in /etc/apt/sources.list.d/ (and its key in /etc/apt/keyrings/), then run sudo apt update.

Security notes:

•	Every extra repository can install code with root access, so only add sources you trust, from the vendor's official documentation.
•	Older tutorials use apt-key add, which is deprecated. Use the signed-by method above.
•	Do not add random Ubuntu repositories to Kali, because mixing them can break the system. Use a snapshot first.


## 15. Environment Variables and PATH

```bash
echo $HOME
echo $USER
echo $PATH          # folders the shell searches for commands
export MYVAR="test"
echo $MYVAR
env                 # list all variables
alias ll='ls -lah'  # create a shortcut for this session
```
PATH matters for security: if a privileged script runs a command without its full path and you can control PATH, you may be able to run your own program instead. Understand it now and you will recognize it later in privilege escalation labs.


## 16. Shell Fundamentals	

A **shell** is a program that accepts commands from the user, interprets them, and asks the operating system to carry them out. The terminal is the application/window where you interact with the shell.

Bash (Bourne Again Shell) is the default shell on many Linux distributions, including Kali and Ubuntu. Other shells include `sh`, `zsh`, and `fish`.

### Checking your current shell

```bash
echo "$SHELL"    # your configured login shell
ps -p $$         # show the shell process running in this terminal
echo "$0"        # often shows the current shell name
cat /etc/shells  # list shells allowed on the system
```

`$SHELL` usually shows your configured login shell. `ps -p $$` checks the process for the shell currently running in this terminal; these values can differ in some situations.

### The shell prompt

A typical Bash prompt may look like this:

```bash
robi@kali:~/practice$
```

- `robi` is the current username.
- `kali` is the hostname (computer name).
- `~/practice` is the current working directory.
- `$` usually indicates a normal user; `#` usually indicates a root shell.

### Command types

```bash
type cd             # identify whether cd is a shell builtin, alias, or executable
type ls             # identify the command type for ls
command -v python3  # show how the shell resolves python3
alias               # list aliases for this shell session
```

Some commands, such as `cd`, are shell builtins because they need to change the state of the current shell. Other commands, such as many programs in `/usr/bin`, are separate executable files.

### Quoting and special characters

Quotes control how the shell interprets spaces and special characters.

```bash
echo "My home is $HOME"         # double quotes allow variable expansion
echo 'My home is $HOME'         # single quotes keep $HOME literal
echo "A file name with spaces"  # double quotes keep the words together
```

Use quotes around paths or filenames that contain spaces:

```bash
cat "my notes.txt"      # read a file whose name contains a space
mkdir "Linux Practice"  # create a directory with a space in its name
```

### Shell startup files

Bash reads configuration files when it starts. These files can set aliases, environment variables, and prompt settings.

```bash
ls -a ~           # show hidden files in your home directory
nano ~/.bashrc    # edit interactive Bash settings for your user
source ~/.bashrc  # reload ~/.bashrc in the current Bash session
```

Common files:

- `~/.bashrc` — settings for interactive non-login Bash shells.
- `~/.profile` — commonly used for login-session environment settings.
- `/etc/profile` — system-wide login-shell settings.
- `/etc/bash.bashrc` — system-wide Bash interactive settings on Debian-based systems.

Be careful when editing shell startup files: a syntax error can affect future terminal sessions. Keep a backup before making substantial changes.

### Shell history

```bash
history     # show commands entered previously
history 10  # show the last 10 history entries
!!          # repeat the previous command
!25         # run history entry number 25
```

History expansion such as `!!` and `!25` is interpreted by Bash. Check a command before repeating it, especially if it uses `sudo` or changes files.

---

## 17. System Information and Identification

System information commands help you identify the Linux distribution, kernel, hardware, memory, processor, and system uptime. This is useful when troubleshooting, checking VM resources, and understanding the machine you are working on.

### Operating system and kernel

```bash
uname                # kernel name
uname -r             # kernel release/version
uname -a             # all available kernel/system information
cat /etc/os-release  # Linux distribution name and version
hostname             # display the computer's hostname
hostnamectl          # show hostname and system information
```

`uname` reports kernel information, while `/etc/os-release` identifies the Linux distribution. They answer different questions.

### CPU and memory

```bash
lscpu              # CPU architecture, cores, threads, and model
free -h            # RAM and swap usage in human-readable units
cat /proc/cpuinfo  # detailed processor information
cat /proc/meminfo  # detailed memory information
```

### Storage and connected devices

```bash
lsblk     # list disks and partitions
lsblk -f  # show filesystem types, labels, and mount points
lspci     # list PCI devices, such as network and graphics adapters
lsusb     # list USB devices
```

### Uptime and logged-in users

```bash
uptime       # how long the system has been running and load averages
who          # users currently logged in
w            # logged-in users and what they are doing
date         # current system date and time
timedatectl  # time, date, and time-synchronization information
```

### A quick system-information check

```bash
hostnamectl
uname -r
lscpu
free -h
lsblk
df -h
uptime
```

These commands only display information; they do not change system configuration.

## 18. Compression and Archives

```bash
tar -cvf archive.tar folder/      # create a tar archive
tar -czvf archive.tar.gz folder/  # create a compressed archive
tar -xvf archive.tar              # extract
tar -xzvf archive.tar.gz          # extract a gzipped archive
zip -r archive.zip folder/
unzip archive.zip
```


## 19. Shell Scripting Basics

Create a file hello.sh:

```bash
#!/bin/bash
# A simple script
name="$1"
echo "Hello, $name"

for i in 1 2 3; do
  echo "Count: $i"
done

if [ -f /etc/passwd ]; then
  echo "passwd file exists"
fi
```

Run it:

```bash
chmod +x hello.sh
./hello.sh Robi
```

A small recon-style script (a ping sweep of your own lab network):

```bash
#!/bin/bash
# Ping sweep for a lab network. Use only on networks you own.
for i in $(seq 1 20); do
  ping -c 1 -W 1 192.168.56.$i &> /dev/null && echo "192.168.56.$i is up"
done
```


## 20. Text Editors: nano and Vim

You will edit config files and scripts on machines that have no graphical editor, so learn both.

nano (easy):

nano notes.txt

Keys                                                                            	Action

Ctrl + O, then Enter	                                                 Save
Ctrl + X	                                                                         Exit
Ctrl + W	                                                                       Search
Ctrl + K / Ctrl + U	                                            Cut a line / paste it

Vim (powerful, and on almost every server):

vim notes.txt
Vim has modes, and this is what confuses beginners:

•	Normal mode (default): keys are commands, not text. Press Esc to get back here from anywhere.
•	Insert mode: you type text. Press i to enter it.
•	Command mode: press : to type commands like save and quit.

Keys                                 	Action

i                          	Start typing (insert mode)
Esc                   	Return to normal mode
:w                     	Save
:q                         	Quit
:wq or ZZ         	Save and quit
:q!                     	Quit without saving
dd            	        Delete (cut) the current line
yy then p          	Copy a line, then paste it
u                      	Undo
/word then Enter	Search for a word (n for the next match)
:set number       	Show line numbers
gg / G                	Jump to the top / bottom of the file

A safe way to learn: run vimtutor in the terminal. It is a 30-minute interactive lesson built into most Linux systems.


## 21. Chaining Commands: &&, ||, ; and &

Operators let you control when the next command runs.

### Operator	Meaning

&&	Run the second command only if the first one succeeds
||	Run the second command only if the first one fails
;	Run the second command no matter what happened to the first
&	Run the command in the background


```bash
mkdir testdir && cd testdir && echo "made it"     # each step runs only if the last one worked
mkdir testdir && echo "created"                   # second time: mkdir fails, so echo does NOT run
ls /nonexistent || echo "that folder is missing"  # echo runs because ls failed
cd ~/practice ; ls ; pwd                          # all three run, one after another
sleep 60 &                                        # runs in the background
```

### Combine them for useful patterns

```bash
ping -c 1 192.168.56.101 &> /dev/null && echo "host is up" || echo "host is down"
sudo apt update && sudo apt upgrade -y  # upgrade only if the update worked
```
To see whether the last command succeeded, check the exit status: 0 means success and anything else means failure.

```bash
ls /etc
echo $?  # prints 0
ls /nonexistent
echo $?  # prints a non-zero number (2)
```

## 22. Linux Boot Process

The **boot process** is the sequence of steps Linux follows when a computer or virtual machine is switched on. Understanding it helps with troubleshooting startup problems and understanding how Linux services begin running.

### Main stages of the boot process

1. **BIOS/UEFI:** Firmware starts first, performs basic hardware checks, and selects a boot device.
2. **Bootloader (usually GRUB):** The bootloader presents or selects a Linux boot entry and loads the kernel and initial RAM filesystem.
3. **Kernel:** The Linux kernel initializes hardware, memory, drivers, and core operating-system functions.
4. **Initramfs:** A temporary filesystem used during early startup. It provides tools and drivers needed to locate and mount the real root filesystem.
5. **Init system (usually systemd):** The init system starts and manages system services and brings the machine to its configured target.
6. **Login:** A login screen or text login prompt becomes available.

### Checking boot information

```bash
uptime             # show how long the system has been running
who -b             # show the last system boot time
last reboot        # show recorded reboot history
uname -r           # show the running kernel version
ls /boot           # list available kernel and boot files
cat /proc/cmdline  # show parameters passed to the running kernel
```

### Understanding systemd startup

On most modern Kali and Ubuntu installations, `systemd` is PID 1 and manages services and system startup.

```bash
ps -p 1 -o pid,comm,args             # identify process 1
systemctl get-default                # show the default systemd target
systemctl list-units --type=target   # list currently loaded targets
systemctl list-units --type=service  # list loaded services
```

Common systemd targets:

| Target | Purpose |
|---|---|
| `poweroff.target` | Shut down the system |
| `rescue.target` | Minimal rescue environment |
| `multi-user.target` | Multi-user text-based system |
| `graphical.target` | Graphical login and desktop environment |
| `reboot.target` | Reboot the system |

### Viewing boot logs and timing

```bash
journalctl -b          # logs from the current boot
journalctl -b -1       # logs from the previous boot, if available
journalctl -b -p err   # error-priority messages from the current boot
systemd-analyze        # summary of startup time
systemd-analyze blame  # services ordered by time spent during startup
```

`journalctl -b` is especially useful when investigating startup or service problems. Some commands may show limited information depending on permissions and the system's log retention settings.

---

## 23. Disk Management

Disk management means checking storage devices, partitions, available space, and mounted filesystems. A disk is a physical or virtual storage device; a partition is a defined region of that disk.

**Important:** Commands that create, delete, resize, or format partitions can permanently destroy data or make a system unbootable. The examples below focus on inspecting storage. Do not experiment with partition changes on your main system or an important VM disk.

### Viewing disks and partitions

```bash
lsblk          # list disks, partitions, and mount points
lsblk -f       # include filesystem type, UUID, and labels
sudo fdisk -l  # list partition tables on available disks
sudo blkid     # show filesystem types and UUIDs
```

Example `lsblk` output (illustrative):

```text
NAME   SIZE TYPE MOUNTPOINTS
sda     40G disk
├─sda1   1G part /boot
└─sda2  39G part /
```

Here, `sda` is the disk, while `sda1` and `sda2` are partitions. Your actual device names and layout will differ.

### Checking free and used space

```bash
df -h                  # free and used space on mounted filesystems
df -T                  # show filesystem type as well
du -sh ~/practice      # total space used by one directory
du -h --max-depth=1 ~  # show space used by each immediate home subdirectory
```

- `df` reports space available on filesystems.
- `du` estimates the space used by files and directories.

### Mount points and mounted filesystems

A **mount point** is a directory where a filesystem becomes accessible.

```bash
findmnt           # show mounted filesystems in a tree
findmnt /         # show the filesystem mounted at /
mount             # list currently mounted filesystems
cat /proc/mounts  # show kernel mount information
```

For example, a separate storage device may be made accessible through a directory such as `/mnt/data`. The device must have a suitable filesystem and be mounted before its files can be accessed through that directory.

### Removable storage: mount and unmount

First identify the correct device:

```bash
lsblk -f  # identify the device and its filesystem
```

If you have a suitable, already-formatted practice device and know its correct partition, an administrator can mount it at an existing directory:

```bash
sudo mkdir -p /mnt/usb
sudo mount /dev/sdX1 /mnt/usb  # example placeholder: replace with the verified partition
findmnt /mnt/usb               # verify the mount
ls /mnt/usb                    # view its contents
sudo umount /mnt/usb           # unmount before disconnecting the device
```

`/dev/sdX1` is only a placeholder, not a device name to copy blindly. Confirm the device with `lsblk -f` first. Never unplug a mounted storage device without safely unmounting it.

### Swap space

Swap is disk space Linux can use as a backing store for memory pages when needed. It is not a replacement for sufficient RAM.

```bash
swapon --show  # show active swap areas
free -h        # show RAM and swap usage
```

### Disk-management terms

- **Disk:** The complete storage device, such as a virtual disk in VirtualBox.
- **Partition:** A defined section of a disk.
- **Filesystem:** The structure used to organize files and directories on a partition or other storage.
- **Mount point:** The directory through which a filesystem is accessed.
- **UUID:** A filesystem identifier that can be used to refer to a filesystem consistently.


## 24. Linux File Systems

A **filesystem** is the method Linux uses to organize, store, name, and retrieve files and directories on storage devices. Linux supports several filesystem types.

### Common Linux filesystem types

| Filesystem | Description |
|---|---|
| ext4 | Common general-purpose Linux filesystem; widely used on Ubuntu and Kali installations |
| XFS | Journaling filesystem often used for larger storage workloads |
| Btrfs | Supports features such as snapshots and checksums |
| vfat/FAT32 | Commonly used for removable media and compatibility |
| exFAT | Common on removable storage devices, especially larger flash drives |
| NTFS | Common Windows filesystem; Linux can access it with appropriate support |
| swap | Dedicated swap area; it is not a normal file-storage filesystem |

### Identify a filesystem

```bash
lsblk -f           # show filesystem type and mount point for each device
sudo blkid         # show filesystem type, UUID, and labels
df -T              # show filesystem types for mounted filesystems
file -s /dev/sda1  # inspect filesystem/device signature (replace with a verified device)
```

Use the actual device name shown by `lsblk`; `/dev/sda1` above is only an example.

### Files, directories, and inodes

An **inode** stores metadata about a filesystem object, such as its type, owner, permissions, timestamps, and the locations of its data blocks. A filename is stored as a directory entry that refers to an inode.

```bash
ls -i file.txt  # show the inode number of a file
stat file.txt   # show file metadata, including inode and permissions
df -i           # show inode usage on mounted filesystems
```

A filesystem can run out of available inodes even when it still has free storage space. `df -i` helps check inode availability.

### Linux directory structure and filesystems

Linux presents filesystems through one directory tree beginning at `/` (root). A separate partition or storage device can be mounted at a directory within that tree.

Examples:

- `/` — root of the directory tree.
- `/home` — commonly contains users' home directories.
- `/boot` — boot-related files.
- `/mnt` — commonly used as a temporary mount point for filesystems.
- `/media` — commonly used for removable media.
- `/etc/fstab` — configuration file describing filesystems that should be mounted.

### Viewing filesystem information

```bash
findmnt          # display mounted filesystems
findmnt -t ext4  # display mounted ext4 filesystems
df -hT           # show space and filesystem type
cat /etc/fstab   # view configured filesystem mount entries
```

### Understanding /etc/fstab

`/etc/fstab` is a configuration file that describes filesystems and how they should be mounted. Entries commonly contain:

1. Device or filesystem identifier (often a UUID).
2. Mount point.
3. Filesystem type.
4. Mount options.
5. Dump setting.
6. Filesystem-check order.

Example format (illustrative only):

```text
UUID=xxxx-xxxx  /data  ext4  defaults  0  2
```

Do not copy this placeholder entry into your system. Incorrect `/etc/fstab` entries can cause boot or mounting problems. Read the file first and make a backup before editing it.

### Filesystem checks

```bash
findmnt  # identify where a filesystem is mounted
df -h    # check available space
df -i    # check inode availability
```

Filesystem repair tools can modify filesystem metadata. They should only be used with the correct device identified and according to the filesystem's documentation; do not run repair operations on a mounted filesystem unless the tool explicitly supports that operation.


## 25. Linux Practice Exercises

### Practical exercises

Try completing each question yourself without looking at the commands in your notes.

1. Getting Help and Flags (Switches)
Use man and --help to find three useful flags for ls, cp, and grep. Explain what each flag does.

2. Navigating and Managing Files
Inside ~/practice, create projects/web, projects/network, and notes. Create three text files, copy one, rename one, and delete one.

3. Finding Things: find, locate, grep, which
Find every .txt file inside your home directory, search for the word root in /etc/passwd, and locate the executable path of Python 3.

4. Pipes, Redirection and Text Processing
Extract usernames from /etc/passwd, sort them alphabetically, and save the output into users.txt. Then count how many usernames are present.

5. Users, Groups and Permissions
Create a file named script.sh, give its owner read, write, and execute permissions while giving the group and others read and execute permissions. Verify its permissions using ls -l.

6. Important Directories
Explore /etc, /var/log, /tmp, /home, and /proc. Identify what each directory contains and display your Linux distribution information.

7. SSH: Logging In to a Remote Machine
Configure SSH on your Ubuntu VM, connect to it from Kali using SSH, and transfer a text file between the two machines using scp.

8. Basic Networking Commands
Display your IP address, routing table, listening ports, and DNS configuration. Then test connectivity to your Ubuntu VM using ping.

9. Processes and Services
Find the SSH process, display all running processes, check the SSH service status, and restart the SSH service using systemctl.

10. Scheduled Tasks: Cron Jobs
Create a cron job that writes the current date and time into /tmp/time.log every five minutes. Verify your cron configuration.

11. Process IDs and Signals in Depth
Start sleep 600 in the background, find its PID, pause it using SIGSTOP, resume it using SIGCONT, and terminate it gracefully using SIGTERM.

12. Package Management
Search for the tree package, install it, verify its installation, and then remove it using apt.

13. Logs
Display the last 20 authentication log entries, find failed password login attempts, and inspect the SSH service logs using journalctl.

14. apt Repositories and Third-Party Software Sources
Display your configured apt repositories, explain the purpose of GPG signing keys, and identify where additional repository configuration files are stored.

15. Environment Variables and PATH
Display $HOME, $USER, and $PATH. Create a temporary environment variable named MYVAR, display its value, and create an alias named ll for ls -lah.

16. Shell Fundamentals
Identify your current shell, distinguish between a shell builtin and an executable command, and demonstrate the difference between single and double quotes when displaying $HOME.

17. System Information and Identification
Display your Linux distribution, kernel version, CPU information, RAM usage, disk information, and system uptime using appropriate commands.

18. Compression and Archives
Create a folder containing three text files, compress it into archive.tar.gz, extract the archive into another directory, and verify that all files were recovered.

19. Shell Scripting Basics
Create an executable Bash script that accepts a username as an argument, prints a greeting, counts from 1 to 5 using a loop, and checks whether /etc/passwd exists.

20. Text Editors: nano and Vim
Create a text file using nano, save it, reopen it using Vim, modify its contents, and save and exit Vim using the appropriate commands.

21. Chaining Commands: &&, ||, ; and &
Create a directory and enter it using &&. Use || to display an error message when a directory does not exist. Run sleep 60 in the background using &.

22. Linux Boot Process
Explain the six main stages of the Linux boot process in order. Then identify PID 1, your default systemd target, and the boot time using Linux commands.

23. Disk Management
Display your disks, partitions, mounted filesystems, available storage space, and active swap areas. Explain the difference between a disk, partition, and mount point.

24. Linux File Systems
Identify the filesystem type of your Linux root partition, display its UUID, find the inode number of a file, and explain the purpose of /etc/fstab.