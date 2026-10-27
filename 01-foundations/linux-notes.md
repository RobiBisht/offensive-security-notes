My notes on learning Linux for ethical hacking. "Educational use only".

## Progress


                         Source                                                            Status                    Date

Linux Fundamentals [ My Own VM ]                                 Completed           2026-10-27

OverTheWire Bandit (levels 0 to 20)                                  In progress



##Linux Fundamentals [ My Own VM ]


#User Information

Command                                                      What it does                                                      Example 

`whoami`                                      Shows which user I am on the system                            `whoami`


#Navigation Commands

Command                                                      What it does                                                      Example 

`pwd`                                      Shows where I am (print working directory)                             `pwd` 

`ls`                                            Lists files and folders in the current folder                                `ls` 

`cd`                                                       Moves into another folder                                    `cd foldername` 


#Text And File Operations

Command                                                      What it does                                                      Example 

`echo`                                                      Prints the text I give it                                     `echo "hello world"`

`cat`                                                     Shows the contents of a file                                    `cat filename`


#Searching Commands

Command                                                      What it does                                                                         Example 

`find`                                   Searches for files by name on current directory                             `find -name passwords.txt`  

`grep`                                                 Searches inside a file for text                                    `grep "password123" passwords.txt` 


#Shell operators

Operator                                                                     Meaning                                                                                          Example

`&`                                     Runs a command in the background so I can keep working                                                `sleep 60 &`
`&&`                       Runs the second command only after the first one finishes and succeeds                            `mkdir lab && cd lab`
`>`                                       Sends output to a file and **overwrites** what was in it                                       `echo "hello" > notes.txt`  
`>>`                                          Sends output to a file and **appends** to the end                                          `echo "world" >> notes.txt`



## Things that confused me

1] When I Tried To When I Moved To A Folder Using `cd` I Didn't Get How To Move Out From That Folder, So I Researched It The I Got To Know The `cd ..` Let You Move Out From Current Directory.

2] When I First Used `cat` Command I Didn't Get What's Happening Because It Showed Nothing When I Did ctrl+c To Stop Whatever Making It Busy Then I Got To Know About First I Have To Move To the Folder/Directory Which Contains A File And Then Used It Like This `cat filename.extension` Then It Worked.


 
