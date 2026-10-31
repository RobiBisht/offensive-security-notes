# Linux Fundamental Notes.

Setup: Your Practice Environment


What you need (all free):


•	VirtualBox or VMware Workstation Player
•	Kali Linux VM (your attacker machine)
•	Ubuntu Server or Desktop VM (your practice target; also use it to practice SSH)
•      You Can Use WSL And Install Kali \& Ubantu To Use Linux In Windows Easy Setup Good For Commands.

Book :

•	The Linux Command Line by William Shotts.

Network setting: In VirtualBox, set both VMs to the same Host-only Adapter (or NAT Network) so they can reach each other. Take a snapshot of each VM before you start, so you can undo mistakes.

Create a safe practice folder (in Kali or Ubuntu):

```bash
mkdir ~/practice                                                           # create a practice folder.

cd ~/practice                                                                 # do all the exercises inside this folder so you do not break anything.
```

The Man Pages (Manual Pages) are built-in Linux documentation that explains how to use Linux commands. on your VM (type man ls; press q to quit)


## 1. Getting Help and Flags (Switches)

Most commands accept flags that change their behavior. A flag usually starts with - (short options) or -- (long options).

```bash
ls                    # list files
ls -l                 # long format (permissions, owner, size, date)
ls -a                # show hidden files (names starting with .)
ls -lah            # combine flags: long, all, human-readable sizes
ls --help         # quick help
man ls           # full manual (press q to quit)
```


## 2. Navigating and Managing Files

```bash
pwd                                                                 # print current directory
cd /etc                                                             # go to an absolute path
cd ..                                                                 # go up one level
cd ~                                                                 # go to your home directory
cd -                                                                  # go back to the previous directory
```

```bash
mkdir notes                                                    # create a folder
mkdir -p a/b/c                                                # create nested folders in one go
touch file1.txt                                                # create an empty file
cp file1.txt file2.txt                                        # copy a file 
cp -r a/ backup/                                             # copy a folder 
mv file2.txt renamed.txt                               # move or rename
rm renamed.txt                                             # delete a file (no recycle bin!)
rm -r backup/                                                # delete a folder and its contents
rmdir notes                                                   # delete an empty folder
```

(cp copy file1.txt to file2.txt, if file2.txt exists it overwrites it, if it doesn't exist it creates one and then copy)
(-r means recursive, means performing an operation on a directory and everything inside it, including its subdirectories and their contents.)

### Viewing file contents

```bash
cat file.txt                                                     # print the whole file
less /etc/passwd                                          # scroll through a file (q to quit)
head -n 5 /etc/passwd                                # print first 5 lines
tail -n 5 /etc/passwd                                   # print last 5 lines
wc -l /etc/passwd                                        # count lines
file /bin/ls                                                    # what type of file is this
```

### Writing to files

```bash
echo "hello" > hello.txt                              # `>` write 
echo "second line" >> hello.txt                 # `>>` appends 
nano hello.txt                                            # simple text editor (Ctrl+O save, Ctrl+X exit)
```

'>'(overwrites the file)
'>>' appends new content at the end of the file without deleting existing content.


## 3. Finding Things: find, locate, grep, which

```bash
find / -name "passwd" 2>/dev/null        # find a file by name
find /home -type f -name "*.txt"         # find all .txt files
find / -type f -size +10M 2>/dev/null    # files larger than 10 MB
find / -perm -4000 2>/dev/null           # find SUID files 
find / -user root -type f 2>/dev/null    # files owned by root

locate passwd                            # fast search (run: sudo updatedb first)
which python3                            # where is this command located

grep "root" /etc/passwd                  # lines containing root
grep -i "error" file.txt                 # case-insensitive
grep -r "password" ~/practice            # search inside all files in a folder
grep -n "bash" /etc/passwd               # show line numbers
grep -v "nologin" /etc/passwd            # show lines that do NOT match
2>/dev/null hides permission errors so the output stays readable.
```

find / -perm -4000 2>/dev/null           (important for privilege escalation) \[Research Permission Bits To Know More About]


## 4. Pipes, Redirection and Text Processing

A pipe (|) sends the output of one command into another. This is one of the most useful Linux skills for hacking.

```bash
cat /etc/passwd | grep "bash"            # users with a bash shell
ls -l /etc | wc -l                       # count items in /etc
cut -d ":" -f 1 /etc/passwd              # print only usernames (first field)
cut -d ":" -f 1,7 /etc/passwd            # username and shell
sort names.txt                           # sort lines
sort names.txt | uniq                    # remove duplicate lines
sort names.txt | uniq -c | sort -nr      # count and rank duplicates
awk -F: '{print $1}' /etc/passwd         # same as cut, using awk
sed 's/bash/zsh/' file.txt               # replace text in output (does not change the file)
tr 'a-z' 'A-Z' < file.txt                # convert to uppercase
```

### Redirection

```bash
command > file      # write output to a file (overwrite)
command >> file     # append new output at the end of the file without deleting existing content.
command 2> errors   # write only errors to a file
command &> all      # write output and errors
command < file      # read input from a file
```


## 5. Users, Groups and Permissions

### Read permissions

```bash
ls -l file.txt
-rwxr-xr-- 1 kali kali 120 Oct 27 10:00 file.txt
```



Breakdown: the first character is the type (- file, d directory), then three sets of rwx for owner, group, others. r=read [4(numeric value)], w=write [2], x=execute [1], So its like this -(type)rwx(owner)r-x(groups)r--(others), owner have read,write\&execute permission, group have read \& execute permission, others have only read permission, permission bits written as this rwx so when you see r-x, r--, --x, ---, every - (hyphen) means that permission not granted. (For more information about Permission Bits research It, permission bits are necessary to understand for further study).

### Changing permissions

```bash
chmod 755 script.sh        # owner rwx, group r-x, others r-x (rwx have theie numeric values r=4 w=2 x=1
chmod 644 notes.txt        # owner rw-, group r--, others r--
chmod +x script.sh         # add execute for everyone
chmod u+x,g-w file         # symbolic: user plus x (add execute for user), group minus w (minus write for group)
chmod -R 750 folder/       # recursively
```

### Ownership

```bash
chown user file.txt                        # change owner (needs sudo)
chown user:group file.txt           # change owner and group
id                                                  # your user id and groups
whoami                                      # current user
groups                                         # your groups
```

### Users and sudo

```bash
sudo adduser testuser      # create a user (set a password when asked)
sudo usermod -aG sudo testuser   # add to the sudo group
su testuser                # switch user
sudo -l                    # list what you can run with sudo
sudo passwd testuser       # change a password
sudo deluser testuser      # delete the user
```

### Special permissions (very important for privilege escalation)

•	SUID (s in the owner position, like -rwsr-xr-x): the program runs with the owner's rights. Example: /usr/bin/passwd.
•	SGID: runs with the group's rights.
•	Sticky bit (t, like drwxrwxrwt on /tmp): users can only delete their own files in the folder.

```bash
ls -l /usr/bin/passwd
ls -ld /tmp
chmod u+s file             # set SUID (for learning only)
```


## 6. Important Directories

Directory	                 What it holds

/                   	 # Root of the whole file system
/home	         # Normal users' personal folders
/root	         # Home folder of the root user
/etc	                 # System configuration files (passwd, shadow, hosts, ssh/)
/var/log	         # System and application logs
/tmp              	 # Temporary files (anyone can write here)
/usr/bin, /bin	       # Command programs
/opt	                        # Optional third-party software
/proc	                # Live system and process information
/dev	                        # Device files

```bash
ls /etc
cat /etc/passwd            # list of users
sudo cat /etc/shadow       # hashed passwords (root only)
cat /etc/hosts
cat /etc/os-release        # which distro and version
ls /var/log
```


## 7. SSH: Logging In to a Remote Machine

SSH lets you control another machine's terminal securely. Practice it between your Kali and Ubuntu VMs.

On the Ubuntu VM (the machine you connect to):

```bash
sudo apt update                                                         # refresh the terminal
sudo apt install openssh-server -y                            # install SSH
sudo systemctl start ssh                                            # start SSH
sudo systemctl enable ssh                                        # enable SSH
sudo systemctl status ssh                                         # status ## should show active (running)
ip a                                                                             # note the IP address (for example 192.168.56.101)
```

On the Kali VM (the machine you connect from):

```bash
ssh ubuntuusername@192.168.56.101               # use you ubantu login username
```

Type yes the first time, then enter the password. Your prompt will change to the remote machine. Type exit to leave.

Copy files with scp:

```bash
scp file.txt user@192.168.56.101:/home/user/       # upload
scp user@192.168.56.101:/home/user/file.txt .      # download
```

Key-based login (more secure and a good thing to learn):

```bash
ssh-keygen -t ed25519
ssh-copy-id user@192.168.56.101
ssh user@192.168.56.101        # now works without a password
```


## 8. Basic Networking Commands

```bash
ip a                                                      # your IP addresses and interfaces
ip route                                               # routing table and default gateway
ping -c 4 192.168.56.101                    # test connectivity (4 packets)
ss -tuln                                                 # open listening ports
sudo ss -tulnp                                       # include the program using each port
hostname                                            # machine name
cat /etc/resolv.conf                               # DNS settings
curl http://example.com                      # fetch a web page
wget http://example.com/file.txt         # download a file
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
ps                                # processes associated with your current terminal
ps aux                            # all processes on the system
ps aux | grep ssh         # find a specific process
top                                 # live view (q to quit)
htop                              # nicer version (sudo apt install htop)
```

### Controlling processes

```bash
sleep 300 &              # run a command in the background
jobs                     # list background jobs
fg %1                    # bring job 1 to the foreground
kill 1234                # stop the process with PID 1234
kill -9 1234             # force-kill
pkill sleep              # kill by name
```

### Services with systemctl

```bash
systemctl status ssh
sudo systemctl stop ssh
sudo systemctl start ssh
sudo systemctl restart ssh
sudo systemctl enable ssh     # start at boot
sudo systemctl disable ssh    # do not start at boot
systemctl list-units --type=service
```


## 10. Scheduled Tasks: Cron Jobs

Cron runs commands on a schedule. Attackers also abuse misconfigured cron jobs, so understanding them matters.

```bash
crontab -e               # edit your cron jobs
crontab -l               # list your cron jobs
ls /etc/cron.d           # system cron jobs
cat /etc/crontab
```

Format: minute hour day-of-month month day-of-week command

```bash
* * * * * echo "hi" >> /tmp/cron-test.txt        # every minute
0 2 * * * /home/user/backup.sh                   # every day at 2:00
*/5 * * * * date >> /tmp/time.log                # every 5 minutes
0 9 * * 1 /home/user/weekly.sh                   # Mondays at 9:00
```


## 11. Process IDs and Signals in Depth

Every running process has a PID (process ID) and a PPID (its parent's ID). The first process, PID 1 (systemd or init), starts all the others.

```bash
ps -ef                  # all processes with PID and PPID
ps aux | grep ssh       # find a process
pgrep ssh               # print the PID of processes named ssh
pstree                  # show the parent and child tree
echo $$                 # the PID of your current shell
ls /proc/$$             # the live data Linux keeps about that process
```

Signals are messages sent to processes. kill does not only kill; it sends any signal.

```bash
kill -l                 # list all signals
kill -15 1234           # SIGTERM: politely ask the process to stop (the default)
kill -9 1234            # SIGKILL: force it to stop immediately, no cleanup
kill -19 1234           # SIGSTOP: pause the process
kill -18 1234           # SIGCONT: resume a paused process
kill -1 1234            # SIGHUP: hang up; many services reload their config
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
sleep 600 &             # start a background process
pgrep sleep             # note its PID
kill -19 <PID>          # pause it
ps aux | grep sleep     # the state column shows T (stopped)
kill -18 <PID>          # resume it
kill -15 <PID>          # stop it politely
```

To keep a process running after you close the terminal, start it with nohup:

```bash
nohup python3 -m http.server 8000 &
```
Use kill -15 first and only use kill -9 when a process will not stop, because -9 skips cleanup such as closing files.


## 12. Package Management

On Debian-based systems (Kali, Ubuntu) use apt.

```bash
sudo apt update                  # refresh the package list
sudo apt upgrade -y              # upgrade installed packages
sudo apt install tree -y         # install a package
sudo apt remove tree             # remove it
sudo apt purge tree              # remove it and its config
apt search nmap                  # search packages
apt show nmap                    # package details
dpkg -l | grep nmap              # check whether it is installed
sudo dpkg -i package.deb         # install a downloaded .deb file
```

## 13. Logs

Logs show what happened on the system. As a hacker you read them to understand detection; as a defender you read them to find attacks.

```bash
ls /var/log
sudo tail -n 20 /var/log/syslog         # general system log (Ubuntu)
sudo tail -n 20 /var/log/auth.log       # logins and sudo use (Ubuntu with rsyslog)
sudo tail -f /var/log/auth.log          # follow live (Ctrl+C to stop)
sudo journalctl -u ssh -n 20            # logs for the ssh service
sudo journalctl -xe                     # recent logs with explanations
sudo grep "Failed password" /var/log/auth.log
last                                    # login history
```
If /var/log/auth.log does not exist, your system uses the systemd journal, so use journalctl instead.


## 14. apt Repositories and Third-Party Software Sources

apt installs software from repositories: servers that hold packages. Your system lists them in:

```bash
cat /etc/apt/sources.list            # main list of repositories
ls /etc/apt/sources.list.d/          # extra repository files
Kali's repositories are set up for security tools. On Ubuntu you may want software that is not in the default repositories, such as a vendor's own tools. Adding a third-party source has three parts:
```

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
echo $PATH              # folders the shell searches for commands
export MYVAR="test"
echo $MYVAR
env                     # list all variables
alias ll='ls -lah'      # create a shortcut for this session
```
PATH matters for security: if a privileged script runs a command without its full path and you can control PATH, you may be able to run your own program instead. Understand it now and you will recognize it later in privilege escalation labs.


## 16. Compression and Archives

```bash
tar -cvf archive.tar folder/        # create a tar archive
tar -czvf archive.tar.gz folder/    # create a compressed archive
tar -xvf archive.tar                # extract
tar -xzvf archive.tar.gz            # extract a gzipped archive
zip -r archive.zip folder/
unzip archive.zip
```


## 17. Shell Scripting Basics

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


## 18. Text Editors: nano and Vim

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


## 19. Chaining Commands: &&, ||, ; and &

Operators let you control when the next command runs.

### Operator	Meaning

&&	Run the second command only if the first one succeeds
||	Run the second command only if the first one fails
;	Run the second command no matter what happened to the first
&	Run the command in the background


```bash
mkdir testdir && cd testdir && echo "made it"     # each step runs only if the last one worked
mkdir testdir && echo "created"                     # second time: mkdir fails, so echo does NOT run
ls /nonexistent || echo "that folder is missing"    # echo runs because ls failed
cd ~/practice ; ls ; pwd                            # all three run, one after another
sleep 60 &                                          # runs in the background
```

### Combine them for useful patterns

```bash
ping -c 1 192.168.56.101 &> /dev/null && echo "host is up" || echo "host is down"
sudo apt update && sudo apt upgrade -y              # upgrade only if the update worked
```
To see whether the last command succeeded, check the exit status: 0 means success and anything else means failure.

```bash
ls /etc
echo $?      # prints 0
ls /nonexistent
echo $?      # prints a non-zero number (2)
```



