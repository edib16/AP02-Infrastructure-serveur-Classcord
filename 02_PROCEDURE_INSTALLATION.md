# Procédure d'Installation - Serveur Classcord

> **Auteur :** Edib Saoud
> **Date :** 06/2025
> **Version :** 1.0
> **Statut :** Validé

---

## 1. Pré-requis

- Fichier ISO de **Debian 12 (Bookworm) Netinst**.
- Environnement de virtualisation (VMware / VirtualBox ou Proxmox).
- Connexion internet active sur le réseau hôte.
- Au moins 40 Go d'espace disque et 4 Go de RAM disponibles.

---

## 2. Phase 1 : Installation de l'OS de base

### 2.1 Configuration de la Machine Virtuelle
1. Créer une nouvelle VM nommée `srv-classcord`.
2. Allouer **4096 Mo (4 Go)** de RAM et **2 vCPU**.
3. Allouer **40 Go** de stockage dynamiquement alloué.
4. Paramétrer la carte réseau en mode **Accès par pont (Bridge)** pour être directement joignable sur le LAN.
5. Lier l'ISO Debian 12 au lecteur optique virtuel et démarrer.

### 2.2 Déroulement de l'Installation Debian
1. Choisir **"Install"** (installation texte recommandée pour serveur).
2. Langue : **Français** | Pays : **France** | Clavier : **Français**.
3. Nom d'hôte (hostname) : `srv-classcord`.
4. Nom de domaine : (Laisser vide).
5. Mots de passe : 
   - Définir un mot de passe `root` complexe.
   - Créer un utilisateur d'administration : `sysadmin`.
6. Partitionnement : **Assisté - utiliser un disque entier**. Tout dans une seule partition.
7. Sélection des logiciels (`tasksel`) :
   - ❌ DÉCOCHER : Environnement de bureau Debian (GNOME, XFCE...).
   - ✅ COCHER : **Serveur SSH**.
   - ✅ COCHER : **Utilitaires usuels du système**.
8. Installer le chargeur d'amorçage GRUB sur le disque principal (`/dev/sda`).
9. Redémarrer une fois terminé.

---

## 3. Phase 2 : Configuration Système et Réseau

### 3.1 Adressage IP Statique

Afin de garantir un accès stable, le serveur doit avoir une adresse IP fixe (`192.168.10.150`).

1. Se connecter avec l'utilisateur `root`.
2. Éditer le fichier réseau :
```bash
nano /etc/network/interfaces
```
3. Modifier la configuration de l'interface principale (ex: `ens33` ou `enp0s3`) :
```text
# Configuration statique pour Classcord
auto enp0s3
iface enp0s3 inet static
    address 192.168.10.150
    netmask 255.255.255.0
    gateway 192.168.10.254
    dns-nameservers 8.8.8.8 8.8.4.4
```
4. Redémarrer le service réseau :
```bash
systemctl restart networking
```

### 3.2 Mise à jour du système

```bash
apt update
apt upgrade -y
```

---

## 4. Phase 3 : Durcissement et Pare-feu (UFW)

### 4.1 Installation et Configuration du Pare-feu
L'objectif est d'isoler le serveur en bloquant tous les ports sauf ceux nécessaires.

```bash
# 1. Installer UFW
apt install ufw -y

# 2. Règles par défaut (Tout refuser en entrée, tout accepter en sortie)
ufw default deny incoming
ufw default allow outgoing

# 3. Autoriser SSH (port 22)
ufw allow 22/tcp

# 4. Autoriser les flux Web (Classcord Backend/Frontend)
ufw allow 80/tcp
ufw allow 443/tcp

# 5. Activer le pare-feu
ufw enable
```

*Note : Le port de la base de données PostgreSQL (5432) ou Node.js (3000) n'a pas besoin d'être ouvert vers l'extérieur si Nginx fait office de reverse proxy local.*

### 4.2 Sécurisation SSH
Pour éviter les attaques par force brute sur le compte super-administrateur.

1. Éditer la configuration SSH :
```bash
nano /etc/ssh/sshd_config
```
2. Trouver et modifier la ligne suivante pour interdire l'accès root :
```text
PermitRootLogin no
```
3. Relancer SSH :
```bash
systemctl restart ssh
```

---

## 5. Phase 4 : Mise à disposition de l'équipe SLAM

À ce stade, le serveur `srv-classcord` est prêt et sécurisé au niveau réseau.

**Fourniture des accès aux développeurs :**
- L'équipe SLAM peut se connecter via `ssh sysadmin@192.168.10.150`.
- Ils peuvent procéder à l'installation de Node.js, PostgreSQL et nginx, tout en sachant que le trafic HTTP/HTTPS est déjà autorisé dans le pare-feu du serveur.
