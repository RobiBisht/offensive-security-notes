**Only See This After Practicing Each Question By Own For At Least 10 Minutes*

Perform these exercises in your Kali and Ubuntu VMs rather than just copying the commands. They cover important Linux fundamentals used in system administration, networking, and cybersecurity labs.

## 1. Create the folder structure lab/web, lab/network, lab/notes with one .txt file in each.

### Command : 

```bash
mkdir -p lab/web lab/network lab/notes
```

### Create one file in each folder:

```bash
touch lab/web/web.txt
touch lab/network/network.txt
touch lab/notes/notes.txt
```

### Verify : 

```bash
tree lab
```

### Expected output:

```text
lab
├── network
│   └── network.txt
├── notes
│   └── notes.txt
└── web
    └── web.txt
```

- mkdir – Creates directories.
- -p – Creates parent directories if needed.
- touch – Creates an empty file.
- tree – Displays directory structure.


## 2. Find all files in /etc that are readable by everyone and end in .conf. Save the list to a file.

### Command : 

```bash
find /etc -type f -name "*.conf" -perm -004 2>/dev/null > readable_conf.txt
```

### Explanation:

- find /etc – Searches inside /etc.
- -type f – Finds regular files only.
- -name "*.conf" – Finds files ending in .conf.
- -perm -004 – Finds files with the read permission for others.
- 2>/dev/null – Hides permission error messages.
- > – Saves the output into a file.


### Verify :

```bash
cat readable_conf.txt
```

Note: -perm -004 checks whether the others-read permission bit is set, not whether the file is necessarily accessible through every parent directory.


## 3. List all users on the system with /bin/bash as their shell, sorted alphabetically.

### Command : 

```bash
awk -F: '$7 == "/bin/bash" {print $1}' /etc/passwd | sort
```

### Explanation:

- awk – Processes text data.
- -F: – Uses : as the field separator.
- $7 – Represents the seventh field (login shell).
- $1 – Represents the first field (username).
- sort – Sorts usernames alphabetically.
- | – Sends the output of one command to another.

### Example output:

```text
kali
root
student
```
Your actual output depends on the users on your system.


## 4. Create a user named guest1, make a file only you can read, and prove guest1 cannot read it.

### Step 1: Create the user

```bash
sudo adduser guest1
```

Follow the prompts to set a password.

### Step 2: Create a private file

```bash
echo "This is my private file" > private.txt
```

### Step 3: Give read/write permission only to the owner

```bash
chmod 600 private.txt
```

### Step 4: Verify permissions

```bash
ls -l private.txt
```

### Expected:

```text
-rw------- 1 kali kali ... private.txt
```

### Step 5: Switch to guest1

```bash
su - guest1
```

### Step 6: Try reading the file

```bash
cat /home/kali/private.txt
```

### Expected:

```text
Permission denied
```

### Step 7: Return to your original user

```bash
exit
```

Why does it fail?

chmod 600 means:

Permission                	 Value

Owner                    	  Read + Write (6)
Group	                        None (0)
Others                     	None (0)

Therefore, guest1 cannot read the file.


## 5. SSH from Kali to Ubuntu, create a file on Ubuntu, and download it with scp.

You need two running VMs.

Kali VM SSH client (sender/requester)

Ubuntu VM SSH server (remote machine)

### Step 1: Find Ubuntu's IP address

On Ubuntu:

```bash
ip addr
```

Look for the IP address associated with your network interface, such as : 192.168.1.105

### Step 2: Install and enable SSH server on Ubuntu

```bash
sudo apt update
sudo apt install openssh-server -y
sudo systemctl enable --now ssh
```

Check:

```bash
sudo systemctl status ssh
```

### Step 3: Connect from Kali to Ubuntu

On Kali:

```bash
ssh ubuntu_username@192.168.1.105
```

Replace :

- ubuntu_username with your Ubuntu username.
- 192.168.1.105 with Ubuntu's actual IP.

Enter the Ubuntu user's password when prompted.

### Step 4: Create a file on Ubuntu

After connecting:

```bash
echo "Hello from Ubuntu" > remote.txt
```

Verify:

```bash
ls
cat remote.txt
```

### Step 5: Exit SSH

```bash
exit
```

### Step 6: Download the file using SCP

On Kali:

```bash
scp ubuntu_username@192.168.1.105:~/remote.txt .
```

### Step 7: Verify

```bash
ls
cat remote.txt
```

You have now downloaded the file from Ubuntu to Kali.


## 6. Add a cron job that logs the date every minute, confirm it works, then remove it.

### Step 1: Open the cron editor

```bash
crontab -e
```

### Step 2: 

```bash
* * * * * date >> ~/date_log.txt
```

Explanation:

Field        	Meaning

First *       	Every minute
Second *      	Every hour
Third *      	Every day of the month
Fourth *   	Every month
Fifth *      	Every day of the week

date prints the current date and time.

>> appends the output to the file instead of overwriting it.

### Step 3: Verify the cron job

Wait 1–2 minutes.

```bash
cat ~/date_log.txt
```

You should see timestamps.

### Step 4: Remove the cron job

```bash
crontab -e
```

Delete this line : 

```bash
* * * * * date >> ~/date_log.txt
```

Save and exit.

### Step 5: Confirm removal

```bash
crontab -l
```

The date logging job should no longer appear.


## 7. Install tree, view your lab folder, and remove the package.


Step 1: Install tree

sudo apt update
sudo apt install tree -y


Step 2: View your lab folder

tree lab


Step 3: Remove tree

sudo apt remove tree -y


Step 4: Verify removal

tree --version

The command should no longer be available.

Note: apt remove removes the package but may leave configuration files behind.


8. Count the number of failed SSH login attempts in the logs after you deliberately fail twice.

First, ensure SSH is running on Ubuntu.


Step 1: On Kali, deliberately attempt two unsuccessful logins

ssh wronguser@UBUNTU_IP


Enter an incorrect password if prompted.

Repeat the failed attempt once more.


Step 2: On Ubuntu, inspect SSH authentication logs

sudo grep "Failed password" /var/log/auth.log


Step 3: Count failed password attempts

sudo grep -c "Failed password" /var/log/auth.log

Example output : 2

Important:

- This counts all matching failed-password entries in the log, not necessarily just your two attempts.
- If your system uses systemd journal logging instead, try:

sudo journalctl -u ssh --since "10 minutes ago" | grep -c "Failed password"

For a more precise count, use a time window that contains only your two test attempts.


9. Write a script that prints the names of all files larger than 1 MB in your home folder.


Step 1: Create a script

nano large_files.sh


Step 2: Write this script

#!/bin/bash

echo "Files larger than 1 MB:"

find "$HOME" -type f -size +1M -print


Step 3: Save and exit nano

- Ctrl + O – Save.
- Enter – Confirm filename.
- Ctrl + X – Exit.


Step 4: Make the script executable

chmod +x large_files.sh


Step 5: Run the script

./large_files.sh

Explanation:

- #!/bin/bash – Specifies Bash as the interpreter.
- $HOME – Represents your home directory.
- -type f – Searches regular files.
- -size +1M – Finds files larger than 1 MiB.
- -print – Prints the file paths.
- chmod +x – Gives execute permission.



10. Share a file from one VM to the other using python3 -m http.server and wget.


For this exercise:


- Ubuntu = Server (shares the file).
- Kali = Client (downloads the file).


Step 1: Create a file on Ubuntu

echo "Hello from Ubuntu VM" > shared.txt


Step 2: Start the Python HTTP server on Ubuntu

python3 -m http.server 8000

Expected output:

Serving HTTP on 0.0.0.0 port 8000

Keep this terminal open.


Step 3: Find Ubuntu's IP address

Open another Ubuntu terminal:

hostname -I

Example : 192.168.1.105


Step 4: Download the file from Kali

On Kali:

wget http://192.168.1.105:8000/shared.txt

Replace the IP with your Ubuntu IP.


Step 5: Verify the downloaded file

cat shared.txt

Expected:

Hello from Ubuntu VM


Step 6: Stop the HTTP server

Return to Ubuntu's server terminal and press:

Ctrl + C

The HTTP server is now stopped.

Note: Both VMs must have network connectivity. VirtualBox Host-only or Bridged networking can work; NAT may require port forwarding depending on your setup.




Quick revision

No.	                        Linux command	                                Purpose

1	                              mkdir -p	                          Create nested directories

2	                                find	                                              Search files

3	                            awk, sort	                             Process and sort text

4	                     adduser, chmod, su	          User and permission management

5	                             ssh, scp	                      Remote login and file transfer

6	                             crontab	                                 Schedule tasks

7	                  apt install, apt remove	           Package management

8	                       grep, journalctl	                           Log analysis

9	                           nano, find	                                  Shell scripting

10	             python3 -m http.server, wget         	       HTTP file sharing