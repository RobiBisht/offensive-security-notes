9. Processes and Services

Viewing processes:

ps                       # your current processes
ps aux                   # all processes on the system
ps aux | grep ssh        # find a specific process
top                      # live view (q to quit)
htop                     # nicer version (sudo apt install htop)

Controlling processes:

sleep 300 &              # run a command in the background
jobs                     # list background jobs
fg %1                    # bring job 1 to the foreground
kill 1234                # stop the process with PID 1234
kill -9 1234             # force-kill
pkill sleep              # kill by name

Services with systemctl:

systemctl status ssh
sudo systemctl stop ssh
sudo systemctl start ssh
sudo systemctl restart ssh
sudo systemctl enable ssh     # start at boot
sudo systemctl disable ssh    # do not start at boot
systemctl list-units --type=service


10. Scheduled Tasks: Cron Jobs

Cron runs commands on a schedule. Attackers also abuse misconfigured cron jobs, so understanding them matters.

crontab -e               # edit your cron jobs
crontab -l               # list your cron jobs
ls /etc/cron.d           # system cron jobs
cat /etc/crontab

Format: minute hour day-of-month month day-of-week command

* * * * * echo "hi" >> /tmp/cron-test.txt        # every minute
0 2 * * * /home/user/backup.sh                   # every day at 2:00
*/5 * * * * date >> /tmp/time.log                # every 5 minutes
0 9 * * 1 /home/user/weekly.sh                   # Mondays at 9:00




11. Process IDs and Signals in Depth

Every running process has a PID (process ID) and a PPID (its parent's ID). The first process, PID 1 (systemd or init), starts all the others.

ps -ef                  # all processes with PID and PPID
ps aux | grep ssh       # find a process
pgrep ssh               # print the PID of processes named ssh
pstree                  # show the parent and child tree
echo $$                 # the PID of your current shell
ls /proc/$$             # the live data Linux keeps about that process

Signals are messages sent to processes. kill does not only kill; it sends any signal.

kill -l                 # list all signals
kill -15 1234           # SIGTERM: politely ask the process to stop (the default)
kill -9 1234            # SIGKILL: force it to stop immediately, no cleanup
kill -19 1234           # SIGSTOP: pause the process
kill -18 1234           # SIGCONT: resume a paused process
kill -1 1234            # SIGHUP: hang up; many services reload their config

Signal	                                                          Number	                                                                                   Meaning

SIGHUP	                                                                1	                                                           Terminal closed or reload configuration
SIGINT	                                                                2	                                                                  Interrupt (what Ctrl + C sends)
SIGTERM	                                                               15	                                                        Polite request to terminate (default for kill)
SIGKILL	                                                                9	                                                                   Force kill, cannot be ignored
SIGSTOP	                                                               19	                                                                      Pause, cannot be ignored
SIGCONT	                                                               18	                                                                      Resume a paused process

Keyboard shortcuts in a terminal:

•	Ctrl + C stops the foreground process (SIGINT)
•	Ctrl + Z pauses it and returns you to the shell
•	Then bg resumes it in the background, or fg brings it back to the front

Try this to see signals in action:

sleep 600 &             # start a background process
pgrep sleep             # note its PID
kill -19 <PID>          # pause it
ps aux | grep sleep     # the state column shows T (stopped)
kill -18 <PID>          # resume it
kill -15 <PID>          # stop it politely

To keep a process running after you close the terminal, start it with nohup:

nohup python3 -m http.server 8000 &
Use kill -15 first and only use kill -9 when a process will not stop, because -9 skips cleanup such as closing files.



Practice :

1] Start a sleep 600 process in the background, find its PID with ps aux | grep sleep, and kill it.

2] Add a cron job that writes the date to /tmp/cron-test.txt every minute. Wait 2 minutes, check the file, then remove the job.

3] Start two sleep processes, pause one and kill the other with different signals, and confirm each result with ps aux.
