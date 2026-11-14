“Solutions: practice solutions”

# Linux Notes: Practice Exercises

[Back to Linux index](README.md)

Try completing each question myself without looking at the commands in my notes. Each exercise matches a topic in the other files.

1. **Getting help and flags:** Use `man` and `--help` to find three useful flags for `ls`, `cp` and `grep`. Explain what each flag does.
2. **Navigating and managing files:** Inside `~/practice`, create `projects/web`, `projects/network` and `notes`. Create three text files, copy one, rename one and delete one.
3. **Finding things:** Find every `.txt` file inside my home directory, search for the word `root` in `/etc/passwd`, and locate the executable path of Python 3.
4. **Pipes, redirection and text processing:** Extract usernames from `/etc/passwd`, sort them alphabetically and save the output into `users.txt`. Then count how many usernames are present.
5. **Users, groups and permissions:** Create a file named `script.sh`, give its owner read, write and execute permissions while giving the group and others read and execute permissions. Verify its permissions using `ls -l`.
6. **Important directories:** Explore `/etc`, `/var/log`, `/tmp`, `/home` and `/proc`. Identify what each directory contains and display my Linux distribution information.
7. **SSH:** Configure SSH on my Ubuntu VM, connect to it from Kali using SSH, and transfer a text file between the two machines using `scp`.
8. **Basic networking commands:** Display my IP address, routing table, listening ports and DNS configuration. Then test connectivity to my Ubuntu VM using `ping`.
9. **Processes and services:** Find the SSH process, display all running processes, check the SSH service status, and restart the SSH service using `systemctl`.
10. **Cron jobs:** Create a cron job that writes the current date and time into `/tmp/time.log` every five minutes. Verify my cron configuration.
11. **Process IDs and signals:** Start `sleep 600` in the background, find its PID, pause it using SIGSTOP, resume it using SIGCONT, and terminate it gracefully using SIGTERM.
12. **Package management:** Search for the `tree` package, install it, verify its installation, and then remove it using `apt`.
13. **Logs:** Display the last 20 authentication log entries, find failed password login attempts, and inspect the SSH service logs using `journalctl`.
14. **apt repositories:** Display my configured apt repositories, explain the purpose of GPG signing keys, and identify where additional repository configuration files are stored.
15. **Environment variables and PATH:** Display `$HOME`, `$USER` and `$PATH`. Create a temporary environment variable named `MYVAR`, display its value, and create an alias named `ll` for `ls -lah`.
16. **Shell fundamentals:** Identify my current shell, distinguish between a shell builtin and an executable command, and demonstrate the difference between single and double quotes when displaying `$HOME`.
17. **System information:** Display my Linux distribution, kernel version, CPU information, RAM usage, disk information and system uptime using appropriate commands.
18. **Compression and archives:** Create a folder containing three text files, compress it into `archive.tar.gz`, extract the archive into another directory, and verify that all files were recovered.
19. **Shell scripting:** Create an executable Bash script that accepts a username as an argument, prints a greeting, counts from 1 to 5 using a loop, and checks whether `/etc/passwd` exists.
20. **Text editors:** Create a text file using nano, save it, reopen it using Vim, modify its contents, and save and exit Vim using the appropriate commands.
21. **Chaining commands:** Create a directory and enter it using `&&`. Use `||` to display an error message when a directory does not exist. Run `sleep 60` in the background using `&`.
22. **Linux boot process:** Explain the six main stages of the Linux boot process in order. Then identify PID 1, my default systemd target and the boot time using Linux commands.
23. **Disk management:** Display my disks, partitions, mounted filesystems, available storage space and active swap areas. Explain the difference between a disk, partition and mount point.
24. **Linux file systems:** Identify the filesystem type of my Linux root partition, display its UUID, find the inode number of a file, and explain the purpose of `/etc/fstab`.
