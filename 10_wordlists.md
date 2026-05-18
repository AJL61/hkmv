# 10 — Wordlists

> Référence : https://github.com/danielmiessler/SecLists

---

## Emplacement sur Kali

```bash
# SecLists (à installer si absent)
sudo apt install seclists
ls /usr/share/seclists/

# Wordlists natives Kali
ls /usr/share/wordlists/
```

---

## Par cas d'usage

### Brute-force mots de passe

```bash
# Généraliste — le plus utilisé
/usr/share/wordlists/rockyou.txt

# Petite liste rapide pour tester
/usr/share/seclists/Passwords/Common-Credentials/10-million-password-list-top-1000.txt

# Mots de passe par défaut constructeurs
/usr/share/seclists/Passwords/Default-Credentials/default-passwords.txt
```

### Brute-force web — répertoires & fichiers

```bash
# Rapide — scan initial
/usr/share/wordlists/dirb/common.txt

# Plus complet
/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt

# Extensions spécifiques (PHP, ASP, config)
/usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt
```

### Sous-domaines

```bash
/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
/usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt
```

### Usernames

```bash
/usr/share/seclists/Usernames/top-usernames-shortlist.txt
/usr/share/seclists/Usernames/Names/names.txt
```

### Active Directory

```bash
# Passwords spray AD (éviter le lockout)
/usr/share/seclists/Passwords/Common-Credentials/best110.txt

# Usernames format AD (prenom.nom)
/usr/share/seclists/Usernames/Names/names.txt
```

### Hash cracking

```bash
# Toujours rockyou en premier
/usr/share/wordlists/rockyou.txt

# Si rockyou échoue
/usr/share/seclists/Passwords/Leaked-Databases/rockyou-75.txt
```

---

## Générer une wordlist custom

```bash
# Avec crunch — ex. 8 caractères, minuscules + chiffres
crunch 8 8 abcdefghijklmnopqrstuvwxyz0123456789 -o custom.txt

# Avec cewl — extraire les mots d'un site web
cewl http://[IP] -d 2 -m 5 -w custom.txt
# -d : profondeur | -m : longueur minimum

# Avec cupp — wordlist basée sur une personne (OSINT)
cupp -i
```

---

## Référence rapide

| Cas | Wordlist |
|---|---|
| SSH / FTP brute-force | `rockyou.txt` |
| Web répertoires (rapide) | `dirb/common.txt` |
| Web répertoires (complet) | `directory-list-2.3-medium.txt` |
| Sous-domaines | `subdomains-top1million-5000.txt` |
| Usernames | `top-usernames-shortlist.txt` |
| Hash cracking | `rockyou.txt` |
| AD password spray | `best110.txt` |
| Cible spécifique | `cewl` sur le site |
