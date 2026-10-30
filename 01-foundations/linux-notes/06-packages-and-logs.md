12\. Package Management





On Debian-based systems (Kali, Ubuntu) use apt.



sudo apt update                  # refresh the package list

sudo apt upgrade -y              # upgrade installed packages

sudo apt install tree -y         # install a package

sudo apt remove tree             # remove it

sudo apt purge tree              # remove it and its config

apt search nmap                  # search packages

apt show nmap                    # package details

dpkg -l | grep nmap              # check whether it is installed

sudo dpkg -i package.deb         # install a downloaded .deb file





13\. Logs



Logs show what happened on the system. As a hacker you read them to understand detection; as a defender you read them to find attacks.



ls /var/log

sudo tail -n 20 /var/log/syslog         # general system log (Ubuntu)

sudo tail -n 20 /var/log/auth.log       # logins and sudo use (Ubuntu with rsyslog)

sudo tail -f /var/log/auth.log          # follow live (Ctrl+C to stop)

sudo journalctl -u ssh -n 20            # logs for the ssh service

sudo journalctl -xe                     # recent logs with explanations

sudo grep "Failed password" /var/log/auth.log

last                                    # login history

If /var/log/auth.log does not exist, your system uses the systemd journal, so use journalctl instead.





14 apt Repositories and Third-Party Software Sources



apt installs software from repositories: servers that hold packages. Your system lists them in:



cat /etc/apt/sources.list            # main list of repositories

ls /etc/apt/sources.list.d/          # extra repository files

Kali's repositories are set up for security tools. On Ubuntu you may want software that is not in the default repositories, such as a vendor's own tools. Adding a third-party source has three parts:



1\.	Download the vendor's GPG signing key. This lets apt check that packages really come from them.

2\.	Add a source entry that points to their repository.

3\.	Run sudo apt update, then install the package.



\# 1. Save the vendor's key (the URL and names below are placeholders; use the vendor's official instructions)



sudo mkdir -p /etc/apt/keyrings

curl -fsSL https://example.com/vendor-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/vendor.gpg



\# 2. Add the repository, tied to that key



echo "deb \[signed-by=/etc/apt/keyrings/vendor.gpg] https://example.com/apt stable main" | sudo tee /etc/apt/sources.list.d/vendor.list



\# 3. Refresh and install



sudo apt update

sudo apt install vendor-package

On Ubuntu, a PPA (Personal Package Archive) is a shortcut:



sudo add-apt-repository ppa:name/ppa

sudo apt update

To remove a source: delete its file in /etc/apt/sources.list.d/ (and its key in /etc/apt/keyrings/), then run sudo apt update.



Security notes:



•	Every extra repository can install code with root access, so only add sources you trust, from the vendor's official documentation.

•	Older tutorials use apt-key add, which is deprecated. Use the signed-by method above.

•	Do not add random Ubuntu repositories to Kali, because mixing them can break the system. Use a snapshot first.





Practice :



1] Install tree, run tree \~/practice to view your folder structure, then remove it.



2] Try to SSH into the Ubuntu VM with a wrong password twice from Kali. Then find those "Failed password" lines in the log on Ubuntu.



3] Read your /etc/apt/sources.list and the files in sources.list.d/, and write in your notes what each line means (type of source, URL, release name, components).

