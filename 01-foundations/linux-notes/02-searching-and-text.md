3.Finding Things: find, locate, grep, which



find / -name "passwd" 2>/dev/null        # find a file by name

find /home -type f -name "\*.txt"         # find all .txt files

find / -type f -size +10M 2>/dev/null    # files larger than 10 MB

find / -perm -4000 2>/dev/null           # find SUID files (important for privilege escalation) \[Research Permission Bits To Know More About]

find / -user root -type f 2>/dev/null    # files owned by root



locate passwd                            # fast search (run: sudo updatedb first)

which python3                            # where is this command located



grep "root" /etc/passwd                  # lines containing root

grep -i "error" file.txt                 # case-insensitive

grep -r "password" \~/practice            # search inside all files in a folder

grep -n "bash" /etc/passwd               # show line numbers

grep -v "nologin" /etc/passwd            # show lines that do NOT match

2>/dev/null hides permission errors so the output stays readable.





4\. Pipes, Redirection and Text Processing



A pipe (|) sends the output of one command into another. This is one of the most useful Linux skills for hacking.



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



Redirection:



command > file      # write output to a file (overwrite)

command >> file     # append new output at the end of the file without deleting existing content.

command 2> errors   # write only errors to a file

command \&> all      # write output and errors

command < file      # read input from a file





Practice :



1] Find every file on your system whose name ends in .conf. Then find the SUID binaries and write down five of them.



2] Print a sorted list of all usernames on your system, one per line, and save it to users.txt.

