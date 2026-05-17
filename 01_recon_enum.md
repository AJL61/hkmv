# 01 — Recon & Énumération

---

## Nmap

```bash
# Scan rapide — ports ouverts
nmap -sV 10.10.10.10

# Scan complet — tous les ports
nmap -sV -p- 10.10.10.10

# Scan + scripts par défaut
nmap -sV -sC 10.10.10.10

# Scan UDP (lent, cibler les ports clés)
nmap -sU -p 53,161 10.10.10.10

# Export des résultats
nmap -sV -oN scan.txt 10.10.10.10
```

---

## Web — Gobuster

```bash
# Brute-force de répertoires
gobuster dir -u http://10.10.10.10 -w /usr/share/wordlists/dirb/common.txt

# Avec extension de fichiers
gobuster dir -u http://10.10.10.10 -w /usr/share/wordlists/dirb/common.txt -x php,txt,html

# Sous-domaines
gobuster dns -d target.com -w /usr/share/wordlists/subdomains.txt
```

## Web — Feroxbuster (alternative rapide)

```bash
feroxbuster -u http://10.10.10.10 -w /usr/share/wordlists/dirb/common.txt
```

---

## FTP (port 21)

```bash
# Connexion anonymous
ftp 10.10.10.10
# user : anonymous | pass : (vide)

# Télécharger un fichier
get nom_fichier

# Lister récursivement
ls -la
```

---

## SSH (port 22)

```bash
# Connexion classique
ssh user@10.10.10.10

# Avec clé RSA
ssh -i id_rsa user@10.10.10.10

# Forcer la permission de la clé si refus
chmod 600 id_rsa
```

---

## SMB (port 445)

```bash
# Lister les partages
smbclient -L //10.10.10.10 -N

# Connexion à un partage
smbclient //10.10.10.10/share -N

# Enum automatique
enum4linux -a 10.10.10.10
```

---

## Wordlists utiles

| Cible        | Wordlist                                      |
|--------------|-----------------------------------------------|
| Répertoires  | `/usr/share/wordlists/dirb/common.txt`        |
| Mots de passe| `/usr/share/wordlists/rockyou.txt`            |
| Sous-domaines| `/usr/share/wordlists/amass/subdomains.txt`   |
