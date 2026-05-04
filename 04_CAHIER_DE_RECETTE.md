# Cahier de Recette et Tests - Serveur Classcord

> **Auteur :** Edib Saoud
> **Date :** 06/2025
> **Version :** 1.0
> **Statut :** Validé

---

## 1. Objectifs de la Recette

L'objectif de ce document est de valider que l'infrastructure serveur mise en place (`srv-classcord`) pour l'équipe SLAM respecte les exigences de disponibilité, de sécurité (filtrage réseau UFW) et de performance fixées dans le cahier des charges.

## 2. Tableau de Recette Infrastructure (SISR)

| N° | Catégorie | Test | Procédure | Résultat Attendu | Statut |
|:---|:---|:---|:---|:---|:---:|
| **T1** | **Réseau** | Connectivité locale | Depuis un poste LAN : `ping 192.168.10.150` | Le serveur répond aux requêtes ICMP de manière stable (< 2ms). | ✅ Validé |
| **T2** | **Sécurité** | Accès SSH autorisé | Depuis le poste admin : `ssh sysadmin@192.168.10.150` | Connexion réussie, demande de mot de passe affichée. | ✅ Validé |
| **T3** | **Sécurité** | Interdiction Root SSH | Tentative de connexion via : `ssh root@192.168.10.150` | Accès refusé (`Permission denied`), blocage confirmé par `sshd_config`. | ✅ Validé |
| **T4** | **Sécurité** | Filtrage UFW (Pare-feu) | Scan nmap depuis un client : `nmap 192.168.10.150` | Seuls les ports 22, 80 et 443 apparaissent ouverts (open). Le port BDD 5432 est invisible (filtered/closed). | ✅ Validé |
| **T5** | **Applicatif** | Accessibilité Web | Ouvrir un navigateur client : `http://192.168.10.150` | La page d'accueil de Classcord s'affiche (ou page Nginx de test). | ✅ Validé |
| **T6** | **Supervision**| Stabilité de charge | Lancer `htop` pendant que 5 clients se connectent à l'application. | Le CPU reste inférieur à 50% de charge et la RAM consommée ne dépasse pas 2 Go. Aucune anomalie. | ✅ Validé |

---

## 3. Synthèse de la Recette

- L'infrastructure est conforme aux attentes en termes de sécurité : le serveur n'expose au réseau de l'école que les flux HTTP/HTTPS utiles à l'application et le flux d'administration SSH.
- Le dimensionnement matériel (2 vCPU / 4 Go RAM) est parfaitement adapté pour une charge de développement et de tests intensifs.
- L'équipe SLAM a confirmé l'accès à son environnement de base de données sans latence.

**Décision finale : L'infrastructure est validée pour servir de base opérationnelle au développement du projet Classcord.**
