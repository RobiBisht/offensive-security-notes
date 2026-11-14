# Linux Notes: Users, Groups and Permissions

[Back to Linux index](README.md)

## Read permissions

```bash
ls -l file.txt
```

```text
-rwxr-xr-- 1 kali kali 120 Oct 27 10:00 file.txt
```

Breakdown: the first character is the type (`-` file, `d` directory), then three sets of `rwx` for **owner**, **group** and **others**.

- `r` = read (4), `w` = write (2), `x` = execute (1)
- So `-rwxr-xr--` reads as: type `-`, owner `rwx`, group `r-x`, others `r--`
- The owner has read, write and execute permission, the group has read and execute, and others have only read
- A `-` (hyphen) in a slot means that permission is not granted, so `r-x`, `r--`, `--x` and `---` are all possible patterns

Permission bits are necessary to understand for further study, so research them in more depth.

## Changing permissions

```bash
chmod 755 script.sh   # owner rwx, group r-x, others r-x (r=4, w=2, x=1)
chmod 644 notes.txt   # owner rw-, group r--, others r--
chmod +x script.sh    # add execute for everyone
chmod u+x,g-w file    # symbolic: user plus x (add execute for user), group minus w (remove write for group)
chmod -R 750 folder/  # recursively
```

## Ownership

```bash
chown user file.txt        # change owner (needs sudo)
chown user:group file.txt  # change owner and group
id                         # your user id and groups
whoami                     # current user
groups                     # your groups
```

## Users and sudo

```bash
sudo adduser testuser           # create a user (set a password when asked)
sudo usermod -aG sudo testuser  # add to the sudo group
su testuser                     # switch user
sudo -l                         # list what you can run with sudo
sudo passwd testuser            # change a password
sudo deluser testuser           # delete the user
```

## Special permissions (very important for privilege escalation)

- **SUID** (`s` in the owner position, like `-rwsr-xr-x`): the program runs with the owner's rights. Example: `/usr/bin/passwd`.
- **SGID:** runs with the group's rights.
- **Sticky bit** (`t`, like `drwxrwxrwt` on `/tmp`): users can only delete their own files in the folder.

```bash
ls -l /usr/bin/passwd
ls -ld /tmp
chmod u+s file  # set SUID (for learning only)
```
