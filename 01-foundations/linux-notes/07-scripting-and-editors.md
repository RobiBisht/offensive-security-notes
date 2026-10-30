15\. Environment Variables and PATH



echo $HOME

echo $USER

echo $PATH              # folders the shell searches for commands

export MYVAR="test"

echo $MYVAR

env                     # list all variables

alias ll='ls -lah'      # create a shortcut for this session



PATH matters for security: if a privileged script runs a command without its full path and you can control PATH, you may be able to run your own program instead. Understand it now and you will recognize it later in privilege escalation labs.





16\. Compression and Archives



tar -cvf archive.tar folder/        # create a tar archive

tar -czvf archive.tar.gz folder/    # create a compressed archive

tar -xvf archive.tar                # extract

tar -xzvf archive.tar.gz            # extract a gzipped archive

zip -r archive.zip folder/

unzip archive.zip





17\. Shell Scripting Basics



Create a file hello.sh:



\#!/bin/bash

\# A simple script

name="$1"

echo "Hello, $name"



for i in 1 2 3; do

&#x20; echo "Count: $i"

done



if \[ -f /etc/passwd ]; then

&#x20; echo "passwd file exists"

fi



Run it:



chmod +x hello.sh

./hello.sh Robi



A small recon-style script (a ping sweep of your own lab network):





\#!/bin/bash

\# Ping sweep for a lab network. Use only on networks you own.

for i in $(seq 1 20); do

&#x20; ping -c 1 -W 1 192.168.56.$i \&> /dev/null \&\& echo "192.168.56.$i is up"

done





18\. Text Editors: nano and Vim



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





19\. Chaining Commands: \&\&, ||, ; and \&



Operators let you control when the next command runs.



Operator	Meaning



\&\&	Run the second command only if the first one succeeds

||	Run the second command only if the first one fails

;	Run the second command no matter what happened to the first

\&	Run the command in the background





mkdir testdir \&\& cd testdir \&\& echo "made it"     # each step runs only if the last one worked

mkdir testdir \&\& echo "created"                     # second time: mkdir fails, so echo does NOT run

ls /nonexistent || echo "that folder is missing"    # echo runs because ls failed

cd \~/practice ; ls ; pwd                            # all three run, one after another

sleep 60 \&                                          # runs in the background



Combine them for useful patterns:



ping -c 1 192.168.56.101 \&> /dev/null \&\& echo "host is up" || echo "host is down"

sudo apt update \&\& sudo apt upgrade -y              # upgrade only if the update worked

To see whether the last command succeeded, check the exit status: 0 means success and anything else means failure.



ls /etc

echo $?      # prints 0

ls /nonexistent

echo $?      # prints a non-zero number (2)





Practice:

1] Write a script that takes a file name as an argument and tells you whether it exists, its size and its permissions.



2] Use Vim to create hello.sh, write a three-line script, save it, make it executable and run it. Then open /etc/passwd with vim and quit with :q! so you cannot damage anything.



3] Write one line that creates a folder, enters it and creates a file inside it, so that it stops cleanly if any step fails.

