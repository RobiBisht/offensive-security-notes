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

mkdir ~/practice                                                           # create a practice folder.

cd ~/practice                                                                 # do all the exercises inside this folder so you do not break anything.

The Man Pages (Manual Pages) are built-in Linux documentation that explains how to use Linux commands. on your VM (type man ls; press q to quit)


1. Getting Help and Flags (Switches)

Most commands accept flags that change their behavior. A flag usually starts with - (short options) or -- (long options).

ls                    # list files
ls -l                 # long format (permissions, owner, size, date)
ls -a                # show hidden files (names starting with .)
ls -lah            # combine flags: long, all, human-readable sizes
ls --help         # quick help
man ls           # full manual (press q to quit)


2. Navigating and Managing Files

pwd                                                                 # print current directory
cd /etc                                                             # go to an absolute path
cd ..                                                                 # go up one level
cd ~                                                                 # go to your home directory
cd -                                                                  # go back to the previous directory

mkdir notes                                                    # create a folder
mkdir -p a/b/c                                                # create nested folders in one go
touch file1.txt                                                # create an empty file
cp file1.txt file2.txt                                        # copy a file (cp copy file1.txt to file2.txt, if file2.txt exists it overwrites it, if it doesn't exist it creates one and then copy)
cp -r a/ backup/                                             # copy a folder (-r means recursive, means performing an operation on a directory and everything inside it, including its subdirectories and their contents.)
mv file2.txt renamed.txt                               # move or rename
rm renamed.txt                                             # delete a file (no recycle bin!)
rm -r backup/                                                # delete a folder and its contents
rmdir notes                                                   # delete an empty folder

Viewing file contents:

cat file.txt                                                     # print the whole file
less /etc/passwd                                          # scroll through a file (q to quit)
head -n 5 /etc/passwd                                # print first 5 lines
tail -n 5 /etc/passwd                                   # print last 5 lines
wc -l /etc/passwd                                        # count lines
file /bin/ls                                                    # what type of file is this

Writing to files:

echo "hello" > hello.txt                              # `>` write (overwrites the file)
echo "second line" >> hello.txt                 # `>>` appends new content at the end of the file without deleting existing content.
nano hello.txt                                            # simple text editor (Ctrl+O save, Ctrl+X exit)

Practice :

1] Run man on ls, cp and grep and write down three useful flags for each.

2] In ~/practice, create the structure projects/web, projects/network, and notes. Create three text files, copy one, rename one and delete one.



