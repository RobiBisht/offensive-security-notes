# Windows Basics

My notes on Windows for security work, practiced in my own Windows lab VM. For educational use only.

## Progress

| Topic | Status | Date |
|---|---|---|
| Command Prompt and PowerShell | | |
| Users, groups and UAC | | |
| NTFS permissions and shares | | |
| Registry | | |
| Processes, services and scheduled tasks | | |
| Networking | | |
| Event logs | | |
| Active Directory concepts | | |

<!-- Fill in Status (Done or In progress) and the date as I finish each topic -->

## Where I learned this

- Microsoft Learn (free documentation)
- TryHackMe Windows Fundamentals rooms (some are premium, so check for the lock icon first)
- My own practice in a Windows VM on a host-only network

## 1. Command Prompt and PowerShell

**Commands I practiced** (add what each one does, in my own words)

- [ ] `dir` / `Get-ChildItem`:
- [ ] `cd` / `Set-Location`:
- [ ] `type` / `Get-Content`:
- [ ] `Get-Help` and `Get-Command`:
- [ ] `Select-String`:
- [ ] A pipeline I wrote (for example `Get-Process | Sort-Object CPU -Descending`):

**What I learned**

<!-- 2 or 3 lines: the difference between Command Prompt and PowerShell, what an object is, how the pipeline works -->

**What confused me**

<!-- In my own words, or write "Nothing" -->

## 2. Users, groups and UAC

**Commands I practiced**

- [ ] `whoami /all`:
- [ ] `whoami /priv`:
- [ ] `net user` / `Get-LocalUser`:
- [ ] `net localgroup Administrators`:
- [ ] Creating and deleting a lab user:

**What I learned**

<!-- SIDs, the built-in accounts, what UAC does, standard vs elevated token -->

**Example from my VM**

<!-- A short, real result. Blur or remove usernames if needed -->

**What confused me**

<!-- In my own words -->

## 3. NTFS permissions and shares

**Commands I practiced**

- [ ] `icacls <file>`:
- [ ] `icacls <file> /grant <user>:R`:
- [ ] `Get-Acl`:
- [ ] `dir /r` (alternate data streams):
- [ ] `net share`:

**What I learned**

<!-- Allow vs deny, inheritance, share permissions vs NTFS permissions, why alternate data streams matter -->

**Example from my VM**

<!-- For example: what happened when a user with read-only access tried to write to a file -->

**What confused me**

<!-- In my own words -->

## 4. Registry

**Commands I practiced**

- [ ] `reg query` on a Run key:
- [ ] `reg add` and `reg delete` on a test key:
- [ ] `Get-ItemProperty`:

**What I learned**

<!-- Hives, keys and values, the value types, and why the Run keys matter to defenders -->

**What confused me**

<!-- In my own words -->

## 5. Processes, services and scheduled tasks

**Commands I practiced**

- [ ] `tasklist` / `Get-Process`:
- [ ] `taskkill` / `Stop-Process`:
- [ ] `sc query` / `Get-Service`:
- [ ] `schtasks /query` / `Get-ScheduledTask`:
- [ ] Creating and deleting a lab scheduled task:

**What I learned**

<!-- Normal Windows processes, what svchost.exe and lsass.exe do, how a service differs from a normal program, how malware can masquerade -->

**What confused me**

<!-- In my own words -->

## 6. Networking

**Commands I practiced**

- [ ] `ipconfig /all`:
- [ ] `netstat -ano`:
- [ ] `Get-NetTCPConnection -State Listen`:
- [ ] `Test-NetConnection`:
- [ ] `netsh advfirewall show allprofiles`:

**What I learned**

<!-- How the Windows tools match the Linux ones, which process owned port 445, what the firewall profiles are -->

See also my [networking notes](02-networking/README.md).

## 7. Event logs

**Commands I practiced**

- [ ] `Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4625}`:
- [ ] Event Viewer filters:

**Event IDs I found in my own logs**

| Event ID | What it meant | What I did to cause it |
|---|---|---|

<!-- Add one row for each event I really found, for example a failed logon (4625) after typing a wrong password -->

**What I learned**

<!-- Logon types, why the Security log needs admin rights, why log clearing events matter -->

## 8. Active Directory concepts

Write one line for each, in my own words:

- **Domain:**
- **Domain controller:**
- **Kerberos:**
- **NTLM:**
- **Group Policy:**
- **Workgroup vs domain:**

## 9. Windows vs Linux

| Task | Windows | Linux |
|---|---|---|
| Who am I | `whoami` | `whoami` |
| List files | `dir` / `Get-ChildItem` | `ls` |
| Show processes | `tasklist` / `Get-Process` | `ps aux` |

<!-- Add more rows from my own practice, for example permissions, services, logs and scheduled tasks -->

## Key takeaways

<!-- 3 to 5 lines in my own words after finishing the topics above -->

## Mistakes and confusing parts

<!-- Things that went wrong, and how I fixed them -->