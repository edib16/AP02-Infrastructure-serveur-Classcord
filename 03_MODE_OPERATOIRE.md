# Mode Opératoire d'Exploitation - Serveur Classcord

> **Auteur :** Edib Saoud
> **Date :** 06/2025
> **Version :** 1.0
> **Cible :** Équipe SISR / Administrateurs Système

---

## 1. Description du Serveur

Le serveur `srv-classcord` est une machine Debian 12 (CLI uniquement) hébergeant les applicatifs de la messagerie interne Classcord. Il est essentiel pour le fonctionnement de l'application sur le LAN.

- **IP :** `192.168.10.150`
- **Utilisateur admin :** `sysadmin`
- **Rôle :** Hébergement Web (Port 80/443) et Base de Données

---

## 2. Procédures de Supervision Basique

L'infrastructure étant allégée, la supervision se fait directement en ligne de commande (CLI Linux).

### 2.1 Vérification de l'état du serveur (Charge globale)

**Commande :** `htop` (ou `top`)
- *Objectif :* Vérifier si le CPU ou la RAM sont saturés.
- *Indicateur d'alerte :* Barre RAM dans le rouge (> 3.5 Go utilisés) ou Load Average > 2.00 en constant.
- *Action :* Quitter avec `F10` ou `q`.

### 2.2 Vérification de l'espace disque

**Commande :** `df -h`
- *Objectif :* S'assurer que le stockage (40 Go) n'est pas plein, ce qui bloquerait la base de données.
- *Indicateur d'alerte :* Utilisation de la partition `/` supérieure à 85%.

### 2.3 Vérification des règles de Pare-Feu

**Commande :** `sudo ufw status verbose`
- *Résultat attendu :* Status `active` avec `22/tcp`, `80/tcp` et `443/tcp` en `ALLOW IN`.

---

## 3. Gestion des Services (Dépannage)

Si l'équipe SLAM signale que l'application est inaccessible (Erreur 502 Bad Gateway ou refus de connexion).

### 3.1 Vérifier l'état du service Web (Reverse Proxy)
```bash
sudo systemctl status nginx
```
*Si inactif ou en erreur :* `sudo systemctl restart nginx`

### 3.2 Vérifier l'état de la base de données
```bash
sudo systemctl status postgresql
```
*Si inactif :* `sudo systemctl restart postgresql`

### 3.3 Consulter les journaux système (Logs) en cas d'erreur
Si un service refuse de démarrer, il faut lire les logs pour comprendre l'erreur :
```bash
# Voir les 50 dernières lignes de logs système
sudo journalctl -n 50 -e

# Voir spécifiquement les erreurs critiques
sudo journalctl -p err
```

---

## 4. Procédure de Redémarrage Complet

En cas d'instabilité globale ou après une mise à jour critique du kernel.

**Étape 1 : Prévenir les utilisateurs (Équipe SLAM)**
S'assurer qu'aucun déploiement n'est en cours.

**Étape 2 : Redémarrer proprement**
```bash
sudo reboot
```

**Étape 3 : Contrôle de redémarrage**
1. Lancer un `ping 192.168.10.150 -t` depuis un poste client.
2. Attendre que le ping réponde (environ 20 secondes sur SSD).
3. S'assurer que la page web Classcord charge de nouveau correctement sur un navigateur.
