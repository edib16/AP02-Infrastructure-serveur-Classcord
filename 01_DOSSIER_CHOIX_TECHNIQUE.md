# Dossier de Choix Technique - Serveur Classcord

> **Auteur :** Edib Saoud
> **Date :** 06/2025
> **Version :** 1.0
> **Statut :** Validé

---

## 1. Contexte et Objectif

### Situation
Dans le cadre d'un projet commun SISR/SLAM (BTS SIO), la mise en place de la messagerie interne **Classcord** a nécessité la construction d'une infrastructure dédiée. Le réseau pédagogique d'IRIS Mediaschool impose des règles strictes concernant l'isolation des serveurs et l'ouverture des flux.

### Problématique
Comment fournir un serveur fiable, sécurisé et maintenable permettant aux développeurs (SLAM) de livrer l'application en continu, tout en respectant les contraintes réseau de l'école ?

### Besoin Exprimé
- Hébergement d'un backend Node.js et d'une base de données PostgreSQL.
- Accessibilité de l'application depuis n'importe quel poste du LAN.
- Isolation réseau (fermer tous les ports non-essentiels).
- Haute stabilité pour supporter de multiples clients simultanés en phase de test.

---

## 2. Choix d'Architecture et Analyse

### 2.1 Système d'Exploitation (OS)

| **Solution** | **Avantages** | **Inconvénients** | **Choix** |
|:---|:---|:---|:---:|
| **Ubuntu Server 22.04 LTS** | Documentation riche, PPA récents | Un peu lourd en consommation RAM | ❌ |
| **Debian 12 (Bookworm)** | Stabilité extrême, légèreté, sécurisé par défaut | Paquets parfois moins récents | ✅ |
| **Windows Server 2022** | Interface graphique, intégration AD | Coût licence, forte consommation de ressources | ❌ |

**Décision : Debian 12** a été retenu pour son excellente fiabilité en milieu serveur et sa légèreté, garantissant une utilisation optimale des ressources allouées pour le backend et la base de données.

### 2.2 Sécurité Réseau (Pare-feu)

Le filtrage local sur le serveur est critique pour éviter une exposition indésirable sur le réseau de l'école.
**Choix : UFW (Uncomplicated Firewall)** a été privilégié face à `iptables` brut pour sa facilité de configuration et de relecture des règles. 

Seuls les flux suivants sont autorisés :
- `TCP 22` : Administration SSH depuis le VLAN IT.
- `TCP 80 / 443` : Accès applicatif (HTTP/HTTPS) pour l'application Web.
*(La base de données PostgreSQL sur le port 5432 n'est écoutée qu'en local `127.0.0.1` pour plus de sécurité).*

---

## 3. Analyse des Risques

| Risque Identifié | Probabilité | Impact | Mesure d'Atténuation (Traitement) |
|:---|:---:|:---:|:---|
| **Indisponibilité du serveur** (Crash OS ou Matériel) | Faible | Élevé | Utilisation de Debian (stable), procédures de redémarrage documentées, supervision en ligne de commande. |
| **Ouverture excessive des ports** (Intrusion LAN) | Moyen | Élevé | Application stricte du principe du moindre privilège via UFW. Audits réguliers des ports ouverts (`ss -tuln`). |
| **Surcharge des ressources (DDoS accidentel / fuite mémoire)** | Moyen | Moyen | Supervision de la charge via `htop`, allocation de 4 GB de RAM. |
| **Conflit d'intégration Infra/Applicatif** | Fort | Moyen | Points d'étape réguliers (Agile) avec l'équipe SLAM. Test réseau et applicatif conjoints avant la recette finale. |

---

## 4. Spécifications Techniques Retenues

### Configuration Matérielle (VM)
- **Rôle :** Serveur d'hébergement Backend/BBD
- **Hostname :** `srv-classcord`
- **vCPU :** 2 cœurs
- **RAM :** 4 GB
- **Stockage :** 40 GB SSD
- **Réseau :** Connexion au LAN, configuration IP statique.

### Adressage IP
- **IP Serveur :** `192.168.10.150`
- **Masque :** `255.255.255.0` (/24)
- **Passerelle :** `192.168.10.254`
- **DNS :** `8.8.8.8` / `8.8.4.4`

**Décision Finale** : Architecture validée par le référent pédagogique et l'équipe SLAM. Passage en phase d'implémentation.
