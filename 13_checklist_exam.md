# 13 — Checklist Exam

> À suivre dans l'ordre. Cocher au fur et à mesure.

---

## Avant de commencer

- [ ] Lire attentivement les consignes de l'exam
- [ ] Noter l'IP cible : `export IP=`
- [ ] Créer la structure de travail
```bash
mkdir -p ~/exam/{screenshots,loot,notes}
```
- [ ] Ouvrir le template de rapport
- [ ] Ouvrir un onglet Claude + un onglet GitHub (cheatsheets)

---

## Phase 1 — Recon & Énumération

- [ ] Scan Nmap rapide
```bash
nmap -sV -sC $IP -oN ~/exam/notes/scan.txt
```
- [ ] Lire les résultats — noter ports, services, versions
- [ ] Scan complet tous ports (en arrière-plan)
```bash
nmap -sV -p- $IP -oN ~/exam/notes/scan_full.txt &
```
- [ ] Identifier le type de cible (Linux / Windows / AD)
- [ ] Si port 80/443 → scanner les répertoires web
```bash
gobuster dir -u http://$IP -w /usr/share/wordlists/dirb/common.txt
```
- [ ] Si port 21 → tester FTP anonymous
- [ ] Si port 445 → énumérer SMB
```bash
enum4linux -a $IP
```
- [ ] Si port 22 → noter la version SSH
- [ ] Consulter `robots.txt` et `sitemap.xml` si web
- [ ] Remplir **Section 1** du rapport (périmètre)

---

## Phase 2 — Analyse & Identification des vulnérabilités

- [ ] Chercher les CVE pour chaque version de service trouvée
```bash
searchsploit [service] [version]
```
- [ ] Tester les credentials par défaut sur chaque service
- [ ] Si appli web → tester manuellement les champs (SQLi, XSS, LFI)
- [ ] Si appli web → analyser les requêtes avec Burp Suite
- [ ] Identifier les vulnérabilités → les noter dans `~/exam/notes/notes.txt`

---

## Phase 3 — Exploitation

- [ ] Sélectionner le vecteur d'attaque le plus prometteur
- [ ] Lancer l'exploit
- [ ] **Screenshot dès que la vuln est confirmée**
- [ ] Obtenir un shell → upgrade immédiatement
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```
- [ ] Vérifier les infos de base après accès
```bash
whoami && id && hostname && ip a
```
- [ ] Remplir le **bloc vuln** dans le rapport (pendant que c'est frais)

---

## Phase 4 — Post-Exploitation & Privesc

### Linux
- [ ] `sudo -l` → droits sudo sans mot de passe
- [ ] `find / -perm -4000 2>/dev/null` → binaires SUID
- [ ] `cat /etc/crontab` → cron jobs
- [ ] `getcap -r / 2>/dev/null` → capabilities
- [ ] Lancer LinPEAS
- [ ] Chercher les flags
```bash
find / -name "user.txt" -o -name "root.txt" 2>/dev/null
```

### Windows
- [ ] `whoami /priv` → privilèges disponibles
- [ ] `whoami /groups` → groupes
- [ ] Lancer WinPEAS
- [ ] Vérifier AlwaysInstallElevated
- [ ] Vérifier les services mal configurés
- [ ] Chercher les flags
```cmd
dir /s /b user.txt root.txt
```

- [ ] **Screenshot de chaque étape de privesc**
- [ ] Remplir le **bloc privesc** dans le rapport

---

## Phase 5 — Rapport

- [ ] Remplir le résumé exécutif (intro + tableau de synthèse)
- [ ] Compter les vulns → remplir distribution par sévérité
- [ ] Vérifier que chaque bloc vuln a au moins 1 screenshot PoC
- [ ] Remplir les recommandations
- [ ] Rédiger la conclusion
- [ ] Relire le rapport — vérifier que tous les `[placeholders]` sont remplis
- [ ] Exporter / sauvegarder le rapport

---

## Checklist screenshots

| Étape | Screenshot |
|---|---|
| Scan Nmap | Résultats avec ports ouverts |
| Vuln identifiée | Preuve que la faille existe |
| Exploitation | Commande + résultat |
| Accès obtenu | `whoami` / `id` sur la machine |
| Privesc | Commande + `whoami` → root/admin |
| Flag | Contenu du fichier flag |

---

## En cas de blocage

```
1. Relire les résultats Nmap — un service oublié ?
2. Gobuster avec une wordlist plus grande
3. Chercher des exploits pour chaque version de service
4. Tester les credentials par défaut sur tous les services
5. Consulter HackTricks pour la technique en cours
6. Demander à Claude avec le contexte exact
```

---

## Rappels importants

- Prendre les screenshots **pendant** l'exploit, pas après
- Remplir le rapport **au fil de l'eau**, pas à la fin
- Sauvegarder régulièrement le rapport
- Garder `~/exam/notes/notes.txt` à jour en permanence
