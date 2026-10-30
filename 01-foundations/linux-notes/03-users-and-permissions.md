5\. Users, Groups and Permissions





Read permissions:



ls -l file.txt

\-rwxr-xr-- 1 kali kali 120 Oct 27 10:00 file.txt



Breakdown: the first character is the type (- file, d directory), then three sets of rwx for owner, group, others. r=read \[4(numeric value)], w=write \[2], x=execute \[1], So its like this -(type)rwx(owner)r-x(groups)r--(others), owner have read,write\&execute permission, group have read \& execute permission, others have only read permission, permission bits written as this rwx so when you see r-x, r--, --x, ---, every - (hyphen) means that permission not granted. (For more information about Permission Bits research It, permission bits are necessary to understand for further study).





Changing permissions:



chmod 755 script.sh        # owner rwx, group r-x, others r-x (rwx have theie numeric values r=4 w=2 x=1, lets breakdown 755,  4+2+1 = 7 means owner, 4+1=5 means group, 4+1=5 means other numbers represents the permissions)



chmod 644 notes.txt        # owner rw-, group r--, others r--

chmod +x script.sh         # add execute for everyone

chmod u+x,g-w file         # symbolic: user plus x (add execute for user), group minus w (minus write for group)

chmod -R 750 folder/       # recursively





Ownership:



chown user file.txt        # change owner (needs sudo)

chown user:group file.txt  # change owner and group

id                         # your user id and groups

whoami                     # current user

groups                     # your groups





Users and sudo:



sudo adduser testuser      # create a user (set a password when asked)

sudo usermod -aG sudo testuser   # add to the sudo group

su testuser                # switch user

sudo -l                    # list what you can run with sudo

sudo passwd testuser       # change a password

sudo deluser testuser      # delete the user





Special permissions (very important for privilege escalation):



•	SUID (s in the owner position, like -rwsr-xr-x): the program runs with the owner's rights. Example: /usr/bin/passwd.

•	SGID: runs with the group's rights.

•	Sticky bit (t, like drwxrwxrwt on /tmp): users can only delete their own files in the folder.



ls -l /usr/bin/passwd

ls -ld /tmp

chmod u+s file             # set SUID (for learning only)





6\. Important Directories



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



ls /etc

cat /etc/passwd            # list of users

sudo cat /etc/shadow       # hashed passwords (root only)

cat /etc/hosts

cat /etc/os-release        # which distro and version

ls /var/log





Practice :



1] Create secret.txt and set it so only you can read and write it. Create a second user, switch to them with su, and confirm they cannot read the file.



2] Explain in your own words why /etc/shadow is only readable by root while /etc/passwd is readable by everyone.

