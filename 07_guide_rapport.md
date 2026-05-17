# Guide — Comment remplir le rapport

---

## Page de garde

| Champ | Quoi mettre |
|---|---|
| Client | Nom de la cible / de l'organisation |
| Consultant | Ton prénom, nom et titre |
| Date | Date du jour de l'exam |
| Périmètre | Nom de la machine ou IP cible |
| Type de boîte | Noire (aucune info) / Grise (info partielle) / Blanche (accès total) |

---

## Section 1 — Périmètre & Objectifs

Remplis dès le début de l'exam, avant même de commencer.

```
Systèmes testés   : Nom de la machine
IP cible          : 10.10.10.10
Ports / Services  : à compléter après le scan Nmap
Période de test   : date et heure de début → fin
Type de boîte     : Noire / Grise / Blanche
Objectif          : ex. Obtenir un accès root sur la machine cible
```

---

## Section 2 — Méthodologie

**Ne pas modifier** — les 5 phases sont standard et valables pour tout pentest.
Tu peux ajouter une ligne si tu as utilisé une technique particulière.

---

## Section 3 — Résumé exécutif

### Texte d'introduction
3 à 5 lignes qui résument :
- Ce qui était testé
- Les failles principales trouvées
- L'impact global

**Exemple :**
> Le test d'intrusion réalisé sur la machine cible (10.10.10.10) a permis d'identifier
> plusieurs vulnérabilités critiques conduisant à une compromission complète du système.
> L'exploitation d'une injection SQL a permis l'extraction des credentials administrateur,
> suivie d'une élévation de privilèges via un binaire SUID mal configuré.

### Tableau de synthèse
Remplis une ligne par vulnérabilité trouvée :

| # | Vulnérabilité | Sévérité | CVSS | Statut |
|---|---|---|---|---|
| 1 | Injection SQL | Critique | 9.8 | À corriger |
| 2 | SUID bash | Élevée | 7.8 | À corriger |

### Distribution par sévérité
Compte tes vulns et remplis à la fin :

```
Critique : 1   ← nombre de vulns critiques trouvées
Élevée   : 1
Moyenne  : 0
Faible   : 0
```

---

## Section 4 — Détails techniques

Un bloc par vulnérabilité. Voici comment remplir chaque champ :

| Champ | Quoi mettre |
|---|---|
| **CVE** | Le numéro CVE si connu, sinon N/A |
| **Système** | Service ou composant affecté (ex: Apache 2.4.49, MySQL 5.5) |
| **Description** | Expliquer la faille en 2-3 lignes (quoi, pourquoi c'est vulnérable) |
| **Étapes de reproduction** | Les commandes exactes dans l'ordre pour reproduire |
| **Preuve (PoC)** | Tes screenshots (voir ci-dessous) |
| **Recommandation** | Ce qu'il faudrait faire pour corriger |

### Sévérité — comment choisir

| Sévérité | CVSS | Exemples |
|---|---|---|
| Critique | 9.0 – 10.0 | RCE sans auth, accès root direct |
| Élevée | 7.0 – 8.9 | SQLi avec extraction de données, privesc |
| Moyenne | 4.0 – 6.9 | XSS stocké, info disclosure |
| Faible | 0.1 – 3.9 | Headers manquants, version exposée |

> Si tu ne connais pas le CVSS exact, estime-le selon l'impact réel sur le système.

---

## Screenshots — PoC

### Combien par vulnérabilité ?

**Minimum (si manque de temps) :** 1 screenshot — la preuve finale.

**Idéal :** 2 à 3 screenshots selon cette logique :

| Screenshot | Ce qu'il montre | Exemple |
|---|---|---|
| **Avant** | La vulnérabilité existe | Champ injectable, service vulnérable |
| **Pendant** | L'exploitation en cours | Payload envoyé, commande lancée |
| **Après** | Le résultat obtenu | `whoami → root`, flag lu, données extraites |

### Exemples concrets

**SQLi**
1. Champ login avec le payload `' OR 1=1 --`
2. Réponse serveur confirmant l'injection
3. Données extraites (users/passwords)

**Privesc Linux via SUID**
1. Résultat de `find / -perm -4000` montrant le binaire
2. Commande d'exploitation
3. `whoami` → root

**Reverse shell**
1. Payload envoyé côté attaquant
2. Shell reçu côté Netcat avec `id` ou `whoami`

### Bonnes pratiques screenshot
- Inclure l'IP de la cible dans le terminal pour prouver que c'est bien la bonne machine
- Afficher la commande ET son résultat dans le même screenshot
- Capturer le flag complet si applicable

---

## Section 5 — Recommandations

Complète la ligne générique à la fin :
```
[Recommandation spécifique à la mission]
→ ex. Désactiver le compte anonymous FTP
→ ex. Mettre à jour OpenSSH vers la version 9.x
→ ex. Supprimer le bit SUID sur /usr/bin/vim
```

---

## Section 6 — Conclusion

3 à 5 lignes qui récapitulent :
- Le niveau de risque global (Critique / Élevé / Modéré / Faible)
- Les points les plus urgents à corriger
- Une recommandation sur la suite (nouveau pentest après correction, audit complémentaire, etc.)

**Exemple :**
> Le système audité présente un niveau de risque critique. Les vulnérabilités identifiées
> permettent une compromission complète sans nécessiter de compétences avancées.
> Il est fortement recommandé de corriger en priorité l'injection SQL et la mauvaise
> configuration SUID avant toute remise en production.

---

## Workflow le jour J

```
Début d'exam
    ↓
Remplir page de garde + Section 1 (périmètre)
    ↓
Lancer le scan — pendant ce temps rédiger l'objectif
    ↓
Au fil des découvertes → remplir un bloc vuln dès qu'elle est confirmée
    ↓
Prendre les screenshots immédiatement (ne pas attendre la fin)
    ↓
À la fin → compter les vulns → remplir résumé exécutif + conclusion
```

> **Conseil :** Ne laisse pas le rapport pour la fin. Remplis chaque bloc au moment où
> tu exploites la vuln — tu as encore tout en tête et les screenshots sont frais.
