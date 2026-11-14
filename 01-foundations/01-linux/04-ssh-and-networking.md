# Linux Notes: SSH and Basic Networking

[Back to Linux index](README.md)

## SSH: logging in to a remote machine

SSH lets me control another machine's terminal securely. I practice it between my Kali and Ubuntu VMs.

On the Ubuntu VM (the machine I connect to):

```bash
sudo apt update                     # refresh the package list
sudo apt install openssh-server -y  # install SSH
sudo systemctl start ssh            # start SSH
sudo systemctl enable ssh           # start SSH at boot
sudo systemctl status ssh           # should show active (running)
ip a                                # note the IP address (for example 192.168.56.101)
```

On the Kali VM (the machine I connect from):

```bash
ssh ubuntuusername@192.168.56.101  # use my Ubuntu login username
```

Type `yes` the first time, then enter the password. The prompt will change to the remote machine. Type `exit` to leave.

Copy files with `scp`:

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

## Basic networking commands

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

Start a quick web server to share files between my VMs:

```bash
cd ~/practice
python3 -m http.server 8000
```

Then, from the other VM, run `wget http://IP-OF-THIS-VM:8000/file.txt`. This exact trick is used in real CTFs and pentests to transfer files.
