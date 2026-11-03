# OverTheWire: Bandit Notes

Levels 0 to 20 completed: 2026-11-DD
Link: https://overthewire.org/wargames/bandit/

## Connection (same for every level)

`ssh banditN@bandit.labs.overthewire.org -p 2220`

Replace N with the level you are logging in to, for example bandit0, bandit1. Save each password in a private file on my computer, never in this repo.

## Summary table

| Level | Skill practiced | Commands used |
|---|---|---|
| 0 to 1 | SSH login, reading a file | ssh, cat |
| 1 to 2 | Files with unusual names | cat, ls, ./ |
| 2 to 3 | Filenames with spaces and leading dashes | quotes, ./, ls |
| 3 to 4 | Hidden files, case-sensitive names | ls -a, cd, cat |
| 4 to 5 | Identifying file types, binary vs text | file, cat, ./, wildcards |
| 5 to 6 | Finding files by properties | find, -type, -size, ! -executable |
| 6 to 7 | Searching the whole system by owner, group and size | find, -user, -group, -size, 2>/dev/null |
| 7 to 8 | Searching inside a file for text | grep, ls |
| 8 to 9 | Pipes, finding unique lines | sort, uniq -u, pipe (`\|`) |
| 9 to 10 | Extracting readable strings from binary data | strings, grep, pipe (`|`) |
| 10 to 11 | Decoding Base64-encoded data | cat, base64 -d |



## Level 0 → 1

**Goal:** log in with SSH and read a file in the home directory.

**Commands:**
- `ssh bandit0@bandit.labs.overthewire.org -p 2220`: remote login (`-p` sets the port)
- `ls`: list files to find `readme`
- `cat readme`: print the file

**Learned:** SSH uses a username, a host and a port. Always run `ls` before guessing a file name.

**Confusing:** There wasn't that much confusing in this level.

## Level 1 → 2

**Goal:** read a file whose name looks like a command option.

**Commands:**
- `ssh bandit1@bandit.labs.overthewire.org -p 2220`: log in as the new user (same port, new username and password)
- `ls`: list files to see the unusual file name
- `cat ./filename`: read the file; `./` means "in the current folder"

**Learned:** Linux commands treat names starting with `-` as options or special input, so a file with that name can't be read normally. Writing `./` before the name makes it a path instead of an option. `.` is the current folder and `/` separates path parts.

**Confusing:** I was stuck here for a long time. After researching it, I understood that the shell wasn't failing to find the file, it was reading the name as an option.

## Level 2 → 3

**Goal:** read a file whose name contains spaces and starts with dashes.

**Commands:**
- `ssh bandit2@bandit.labs.overthewire.org -p 2220`: log in as the new user
- `ls`: list files to see the unusual file name
- `cat "./file name"`: read the file; quotes keep the spaces together, and `./` stops the dashes being read as options

**Learned:** Two separate problems were in one file name. Spaces make the shell split a name into several arguments, which quotes fix. A name starting with `-` or `--` is read as a command option, which `./` before the name fixes. I needed both together.

**Confusing:** I knew quotes handle spaces, so I tried double quotes first, but `cat` still gave an "unexpected argument" error. I searched and found that the leading `--` was the real problem, since the quotes only fixed the spaces. Adding `./` made it work.

## Level 3 → 4

**Goal:** find and read a hidden file inside a folder.

**Commands:**
- `ssh bandit3@bandit.labs.overthewire.org -p 2220`: log in as the new user
- `ls`: showed one folder, `inhere`
- `cd inhere`: move into that folder
- `ls -a`: list all files, including hidden ones (names starting with `.`)
- `cat <filename>`: read the hidden file

**Learned:** Plain `ls` hides files whose names start with a dot, so `-a` is needed to see them. Hidden files are a normal Linux feature, not security, and attackers and admins both check for them.

**Confusing:** My first `cat` attempts failed with "No such file or directory" because I typed the name with the wrong capital letters. Linux file names are case-sensitive, so `Hiding` and `hiding` are different names. Now I copy the name from the `ls -a` output or press Tab to complete it.

## Level 4 → 5

**Goal:** find the one human-readable file among several files with names starting with a dash.

**Commands:**
- `ssh bandit4@bandit.labs.overthewire.org -p 2220`: log in as the new user
- `cd inhere` and `ls`: found several files with names starting with `-`
- `cat ./-fileNN`: read a file (`./` stops the dash being read as an option, as in Level 1 → 2)
- `file ./-file*`: shows what type of data each file holds (the better way, which I tried after solving it)

**Learned:** I ran `cat` on the files one by one, and most of them printed unreadable symbols because they hold binary data, not text. The `file` command checks all of them at once and labels the readable one as ASCII text. `*` is a wildcard that matches every file name starting with `-file`.

**Confusing:** Reading binary files with `cat` filled my terminal with garbage characters, and a few prompts looked broken. Typing `reset` fixes a messy terminal. Trying every file by hand worked here, but it would be slow with 100 files, so `file` is the better habit.

**Tried:** `cat ./-file*` printed every file at once. It worked, but binary files filled the screen with unreadable characters and the password was lost in the noise. `file ./-file*` shows each file's type, so I can pick the text file and read only that.

## Level 5 → 6

**Goal:** find one file among many folders using its properties (size, type, permissions) instead of opening each folder by hand.

**Commands:**
- `ssh bandit5@bandit.labs.overthewire.org -p 2220`: log in as the new user
- `cd inhere` and `ls`: about 20 folders, too many to check manually
- `find . -type f -size <N>c ! -executable`: search for files that match the level's clues
  - `.` starts the search in the current folder
  - `-type f` matches regular files only (not folders)
  - `-size <N>c` matches an exact size; `c` means bytes
  - `! -executable` excludes executable files (`!` means "not")
- `cat <path>`: read the file that `find` returned

**Learned:** `find` can filter by several properties at once, so I can search by clues instead of looking through folders. Size suffixes matter: `c` is bytes, `k` is kilobytes, and without the right suffix the size is counted in 512-byte blocks. The file was also hidden (its name started with a dot), and `find` found it anyway because it searches hidden files by default, unlike `ls`.

**Confusing:** After `find` gave me the path, I went into the folder and ran `cat ./-file2`. It printed about 1,000 words of readable text, so I was sure I had the right file. I was confused until I searched and found the problem: the file I wanted was `.file2` (a dot, a hidden file), and I had typed `-file2` (a dash), a different file in the same folder.

**Lesson:** Copy the exact path that `find` prints instead of retyping it. A dot and a dash look similar but are different characters, and similar file names can hide a decoy. Running `cat` straight on the path from `find` (`cat ./folder/.file2`) skips the typing error.

## Level 6 → 7

**Goal:** find a file somewhere on the whole server using its owner, group and size.

**Commands:**
- `ssh bandit6@bandit.labs.overthewire.org -p 2220`: log in as the new user
- `find / -type f -user <owner> -group <group> -size <N>c 2>/dev/null`: search from the root folder
  - `/` starts the search at the top of the file system
  - `-type f` matches regular files only
  - `-user` and `-group` match the file's owner and group
  - `-size <N>c` matches an exact size in bytes
  - `2>/dev/null` hides the "Permission denied" errors by sending error messages to a black hole
- `cat <path>`: read the file that `find` returned

**Learned:** Searching `/` with only a few filters prints thousands of lines, so each extra filter cuts the output. Combining owner, group and size narrows it to exactly one file. `2>/dev/null` matters because without it, every folder I can't read prints an error and buries the real result. `2` is the error stream, and `>` redirects it.

**Confusing:** My first search printed a huge list that scrolled for a long time, because I hadn't narrowed it enough. Adding the owner, group and size filters fixed it.

## Level 7 → 8

**Goal:** find one line in a very large text file, using a keyword given in the level.

**Commands:**
- `ssh bandit7@bandit.labs.overthewire.org -p 2220`: log in as the new user
- `ls`: found a single large text file
- `grep "<keyword>" data.txt`: print only the lines that contain the keyword

**Learned:** `grep` searches inside a file and prints the matching lines, so I don't have to read thousands of lines by hand. This is the same idea as using `find`, but for the contents of a file instead of file names. Quotes around the keyword are good practice, and they're needed when the text has spaces.

**Confusing:** Nothing much in this level, since `grep` was clear once I knew what to search for.

**Next time:** grep flags I want to practice: `-i` ignores case, `-n` shows line numbers, `-c` counts matching lines, `-v` shows lines that don't match.

## Level 8 → 9

**Goal:** find the one line in a text file that appears only once, when every other line is repeated.

**Commands:**
- `ssh bandit8@bandit.labs.overthewire.org -p 2220`: log in as the new user
- `ls`: found `data.txt`
- `sort data.txt | uniq -u`: sort the lines, then print only the lines that appear exactly once
  - `sort` puts identical lines next to each other
  - `|` (pipe) sends the sorted output into the next command
  - `uniq -u` prints only the unique lines (lines that appear once)

**Learned:** `uniq` only compares lines that are right next to each other, so it must be used after `sort`. Without `sort`, the duplicates are scattered and `uniq` can't remove them. This was my first real use of a pipe, where the output of one command becomes the input of the next.

**Confusing:** I didn't know why `uniq` alone didn't work until I read that it only checks neighbouring lines.

**Extra flags to try:** `uniq -c` counts how many times each line appears, and `uniq -d` prints only the duplicated lines. `sort data.txt | uniq -c | sort -nr` ranks lines by how often they appear.


## Level 9 → 10

**Goal:** find the password hidden inside a file containing mostly binary data. The password is one of the few human-readable strings preceded by several `=` characters.

**Commands:**
- `ssh bandit9@bandit.labs.overthewire.org -p 2220`: log in as the new user
- `ls`: found `data.txt`
- `strings data.txt`: extract and print human-readable text from the binary file
- `strings data.txt | grep "===="`: extract readable strings and filter for lines containing several `=` characters
  - `strings`: finds sequences of printable characters inside binary files
  - `data.txt`: the file being searched
  - `|` (pipe): sends the output of `strings` into `grep`
  - `grep "===="`: prints only lines containing four consecutive `=` characters

**Learned:** Binary files can contain readable text mixed with non-readable data. `cat` displays the raw contents, which can produce unreadable output. The `strings` command extracts readable character sequences, making it easier to search for useful information. Combining `strings` with `grep` allows me to filter the extracted text and find the password without reading everything.

**Confusing:** When I first ran `cat data.txt`, it printed unreadable symbols because the file contained binary data. I also tried searching directly with `grep`, but the password was difficult to find. After learning about `strings`, I could extract readable text. Using `grep "===="` narrowed the output to the relevant line.


## Level 10 → 11

**Goal:** decode a Base64-encoded file to reveal the next password.

**Commands:**
- `ssh bandit10@bandit.labs.overthewire.org -p 2220`: log in as the new user
- `ls`: found `data.txt`
- `cat data.txt`: display the encoded contents
- `base64 -d data.txt`: decode the Base64 contents and print the original text

**Learned:** Base64 is an encoding format that represents data using printable characters. It is not encryption, because it does not require a secret key. The `base64` command can encode and decode data. The `-d` option means decode.

**Confusing:** The contents of `data.txt` looked like random letters, numbers and symbols. I initially thought it was encrypted, but it was Base64-encoded text. Using `base64 -d` decoded it and revealed the password.


