7\. SSH: Logging In to a Remote Machine



SSH lets you control another machine's terminal securely. Practice it between your Kali and Ubuntu VMs.



On the Ubuntu VM (the machine you connect to):



sudo apt update                                                         # refresh the terminal

sudo apt install openssh-server -y                            # install SSH

sudo systemctl start ssh                                            # start SSH

sudo systemctl enable ssh                                        # enable SSH

sudo systemctl status ssh                                         # see statusshould show active (running)

ip a                                                                             # note the IP address (for example 192.168.56.101)





On the Kali VM (the machine you connect from):



ssh ubuntuusername@192.168.56.101               # use you ubantu login username

Type yes the first time, then enter the password. Your prompt will change to the remote machine. Type exit to leave.



Copy files with scp:



scp file.txt user@192.168.56.101:/home/user/       # upload

scp user@192.168.56.101:/home/user/file.txt .      # download



Key-based login (more secure and a good thing to learn):



ssh-keygen -t ed25519

ssh-copy-id user@192.168.56.101

ssh user@192.168.56.101        # now works without a password





8\. Basic Networking Commands



ip a                    # your IP addresses and interfaces

ip route                # routing table and default gateway

ping -c 4 192.168.56.101   # test connectivity (4 packets)

ss -tuln                # open listening ports

sudo ss -tulnp          # include the program using each port

hostname                # machine name

cat /etc/resolv.conf    # DNS settings

curl http://example.com            # fetch a web page

wget http://example.com/file.txt   # download a file



Start a quick web server to share files between your VMs:



cd \~/practice

python3 -m http.server 8000

Then from the other VM run wget http://IP-OF-THIS-VM:8000/file.txt. You will use this exact trick in real CTFs and pentests to transfer files.





Practice :



1] Log in from Kali to Ubuntu, create a file there, and download it back with scp. Then disable password login in /etc/ssh/sshd\_config on Ubuntu to see how key-only login works (take a snapshot first).

