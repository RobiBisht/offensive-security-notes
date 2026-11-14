# Linux Notes: Shell, Environment and Scripting

[Back to Linux index](README.md)

## Shell fundamentals

A **shell** is a program that accepts commands from the user, interprets them, and asks the operating system to carry them out. The terminal is the application or window where I interact with the shell.

Bash (Bourne Again Shell) is the default shell on many Linux distributions, including Kali and Ubuntu. Other shells include `sh`, `zsh` and `fish`.

### Checking my current shell

```bash
echo "$SHELL"    # your configured login shell
ps -p $$         # show the shell process running in this terminal
echo "$0"        # often shows the current shell name
cat /etc/shells  # list shells allowed on the system
```

`$SHELL` usually shows my configured login shell. `ps -p $$` checks the process for the shell currently running in this terminal. These values can differ in some situations.

### The shell prompt

A typical Bash prompt may look like this:

```bash
robi@kali:~/practice$
```

- `robi` is the current username.
- `kali` is the hostname (computer name).
- `~/practice` is the current working directory.
- `$` usually indicates a normal user, and `#` usually indicates a root shell.

### Command types

```bash
type cd             # identify whether cd is a shell builtin, alias, or executable
type ls             # identify the command type for ls
command -v python3  # show how the shell resolves python3
alias               # list aliases for this shell session
```

Some commands, such as `cd`, are shell builtins because they need to change the state of the current shell. Other commands, such as many programs in `/usr/bin`, are separate executable files.

### Quoting and special characters

Quotes control how the shell interprets spaces and special characters.

```bash
echo "My home is $HOME"         # double quotes allow variable expansion
echo 'My home is $HOME'         # single quotes keep $HOME literal
echo "A file name with spaces"  # double quotes keep the words together
```

Use quotes around paths or filenames that contain spaces:

```bash
cat "my notes.txt"      # read a file whose name contains a space
mkdir "Linux Practice"  # create a directory with a space in its name
```

### Shell startup files

Bash reads configuration files when it starts. These files can set aliases, environment variables and prompt settings.

```bash
ls -a ~           # show hidden files in your home directory
nano ~/.bashrc    # edit interactive Bash settings for your user
source ~/.bashrc  # reload ~/.bashrc in the current Bash session
```

Common files:

- `~/.bashrc`: settings for interactive non-login Bash shells.
- `~/.profile`: commonly used for login-session environment settings.
- `/etc/profile`: system-wide login-shell settings.
- `/etc/bash.bashrc`: system-wide Bash interactive settings on Debian-based systems.

Be careful when editing shell startup files: a syntax error can affect future terminal sessions. Keep a backup before making substantial changes.

### Shell history

```bash
history     # show commands entered previously
history 10  # show the last 10 history entries
!!          # repeat the previous command
!25         # run history entry number 25
```

History expansion such as `!!` and `!25` is interpreted by Bash. Check a command before repeating it, especially if it uses `sudo` or changes files.

## Environment variables and PATH

```bash
echo $HOME
echo $USER
echo $PATH          # folders the shell searches for commands
export MYVAR="test"
echo $MYVAR
env                 # list all variables
alias ll='ls -lah'  # create a shortcut for this session
```

PATH matters for security: if a privileged script runs a command without its full path and I can control PATH, I may be able to run my own program instead. Understanding it now will help me recognize it later in privilege escalation labs.

## Shell scripting basics

Create a file `hello.sh`:

```bash
#!/bin/bash
# A simple script
name="$1"
echo "Hello, $name"

for i in 1 2 3; do
  echo "Count: $i"
done

if [ -f /etc/passwd ]; then
  echo "passwd file exists"
fi
```

Run it:

```bash
chmod +x hello.sh
./hello.sh Robi
```

A small recon-style script (a ping sweep of my own lab network):

```bash
#!/bin/bash
# Ping sweep for a lab network. Use only on networks you own.
for i in $(seq 1 20); do
  ping -c 1 -W 1 192.168.56.$i &> /dev/null && echo "192.168.56.$i is up"
done
```

## Text editors: nano and Vim

I will edit config files and scripts on machines that have no graphical editor, so I learn both.

**nano (easy):**

```bash
nano notes.txt
```

| Keys | Action |
|---|---|
| `Ctrl + O`, then Enter | Save |
| `Ctrl + X` | Exit |
| `Ctrl + W` | Search |
| `Ctrl + K` / `Ctrl + U` | Cut a line / paste it |

**Vim (powerful, and on almost every server):**

```bash
vim notes.txt
```

Vim has **modes**, and this is what confuses beginners:

- **Normal mode** (default): keys are commands, not text. Press `Esc` to get back here from anywhere.
- **Insert mode:** I type text. Press `i` to enter it.
- **Command mode:** press `:` to type commands like save and quit.

| Keys | Action |
|---|---|
| `i` | Start typing (insert mode) |
| `Esc` | Return to normal mode |
| `:w` | Save |
| `:q` | Quit |
| `:wq` or `ZZ` | Save and quit |
| `:q!` | Quit without saving |
| `dd` | Delete (cut) the current line |
| `yy` then `p` | Copy a line, then paste it |
| `u` | Undo |
| `/word` then Enter | Search for a word (`n` for the next match) |
| `:set number` | Show line numbers |
| `gg` / `G` | Jump to the top / bottom of the file |

A safe way to learn: run `vimtutor` in the terminal. It is a 30-minute interactive lesson built into most Linux systems.

## Compression and archives

```bash
tar -cvf archive.tar folder/      # create a tar archive
tar -czvf archive.tar.gz folder/  # create a compressed archive
tar -xvf archive.tar              # extract
tar -xzvf archive.tar.gz          # extract a gzipped archive
zip -r archive.zip folder/
unzip archive.zip
```
