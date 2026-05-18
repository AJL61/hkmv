# 11 — Reconnaissance Passive & OSINT

> Objectif : collecter un maximum d'informations sans toucher la cible.

---

## Whois — infos sur le domaine

```bash
whois [domaine]
whois [IP]

# Infos utiles : registrar, dates, serveurs DNS, contacts
```

---

## DNS — énumération

```bash
# Résolution basique
nslookup [domaine]
dig [domaine]

# Enregistrements MX, TXT, NS
dig [domaine] MX
dig [domaine] TXT
dig [domaine] NS
dig [domaine] ANY

# Transfert de zone (si mal configuré)
dig axfr @[serveur_dns] [domaine]

# Brute-force sous-domaines
gobuster dns -d [domaine] -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

---

## theHarvester — OSINT automatique

```bash
# Emails, sous-domaines, IPs liés à un domaine
theHarvester -d [domaine] -b google
theHarvester -d [domaine] -b bing
theHarvester -d [domaine] -b all     # toutes les sources
```

---

## Google Dorks

```bash
# Fichiers sensibles exposés
site:[domaine] filetype:pdf
site:[domaine] filetype:xlsx
site:[domaine] filetype:env

# Pages de login
site:[domaine] inurl:login
site:[domaine] inurl:admin

# Fichiers de config
site:[domaine] ext:conf OR ext:config OR ext:ini

# Credentials exposés
site:[domaine] intext:password filetype:log
site:[domaine] "index of" passwords

# Cameras / devices
inurl:/view/index.shtml
intitle:"webcamXP 5"
```

---

## Shodan — reconnaissance sur internet

```bash
# Installer le CLI
pip install shodan --break-system-packages
shodan init [API_KEY]

# Chercher des services exposés
shodan search "hostname:[domaine]"
shodan search "net:[IP/CIDR]"
shodan host [IP]

# Filtres utiles
shodan search "org:[entreprise] port:22"
shodan search "product:Apache version:2.4.49"
```

> Interface web : https://www.shodan.io

---

## Certificats SSL — trouver des sous-domaines

```bash
# Via crt.sh (sans toucher la cible)
curl -s "https://crt.sh/?q=%25.[domaine]&output=json" | python3 -m json.tool | grep "name_value"
```

> Interface web : https://crt.sh

---

## Wayback Machine — pages archivées

```bash
# Trouver d'anciennes versions du site
curl "http://archive.org/wayback/available?url=[domaine]"
```

> Interface web : https://web.archive.org

---

## Recherche de credentials leakés

> https://haveibeenpwned.com
> https://dehashed.com
> https://leakcheck.io

---

## Résumé — ordre d'exécution

```
1. whois [domaine]                  → infos registrar, contacts
2. dig [domaine] ANY                → enregistrements DNS
3. theHarvester -d [domaine] -b all → emails, sous-domaines
4. crt.sh                           → sous-domaines via certificats
5. Google Dorks                     → fichiers et pages sensibles
6. Shodan                           → services exposés sur internet
7. Wayback Machine                  → anciennes pages / configs
```
