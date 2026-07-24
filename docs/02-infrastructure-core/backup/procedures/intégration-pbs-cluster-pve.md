---
title: Intégration de Proxmox Backup Server au cluster Proxmox
service: Backup
date: 2026-07-24
author: Louis MEDO
owner: Louis MEDO
---

# {{ page.meta.title }}

![Bannière LoutikCloud](https://raw.githubusercontent.com/loutik/design-assets/main/loutikcloud/banniere_loutikcloud.png)

!!! info "Informations"
    * **Date de création** : {{ page.meta.date }}
    * **Service ciblé** : {{ page.meta.service }}
    * **Auteur** : {{ page.meta.author }}
    * **Responsable** : {{ page.meta.owner }}

## 1. Contexte
Déploiement et liaison du service Proxmox Backup Server (v4) au cluster Proxmox VE (PVE). PBS centralise les sauvegardes avec déduplication. Le cluster PVE s'authentifie auprès de PBS via un jeton d'API (API Token) et vérifie l'identité du serveur via une empreinte de certificat TLS (Fingerprint) pour sécuriser les flux réseau sur le port TCP 8007.

## 2. Prérequis

* Serveur PBS v4 installé et joignable sur le port TCP 8007.
* Datastore préalablement créé sur le PBS (ex: `backups-loutikcloud`).
* Identifiants d'administration PBS temporaires ou accès root en SSH.
* Résolution DNS configurée entre les nœuds PVE et le serveur PBS.

## 3. Configuration et liaison du stockage

3.1. **Génération des accès sur PBS.** Création de l'utilisateur, du jeton d'API et récupération de l'empreinte TLS depuis le shell du PBS.

  ```bash
  proxmox-backup-manager user create svc-pve-backup@pbs
  proxmox-backup-manager user generate-token svc-pve-backup@pbs pve-token
  proxmox-backup-manager cert info | grep Fingerprint
  ```

  * `proxmox-backup-manager` : Utilitaire en ligne de commande de PBS pour gérer la configuration locale.
  * `user create` : Commande permettant de créer un nouvel utilisateur dans le domaine local (realm `pbs`).
  * `user generate-token` : Génère un jeton d'accès sans mot de passe pour automatiser et sécuriser la connexion du cluster PVE.
  * `cert info | grep Fingerprint` : Affiche les détails du certificat SSL/TLS du serveur PBS et filtre la sortie pour isoler l'empreinte cryptographique (SHA-256), requise pour valider l'authenticité du serveur lors de la connexion.

3.2. **Attribution des permissions (ACL)**. Assignation des droits nécessaires à l'utilisateur de l'API pour interagir avec le datastore.

  ```bash
  # Pour l'utilisateur
  proxmox-backup-manager acl update /datastore/backups-loutikcloud DatastorePowerUser --auth-id svc-pve-backup@pbs

  # Pour le jeton API
  proxmox-backup-manager acl update /datastore/backups-loutikcloud DatastorePowerUser --auth-id 'svc-pve-backup@pbs!pve-token' 
  ```

  * `proxmox-backup-manager` : L'outil en ligne de commande de Proxmox Backup Server.
  * `acl update` : Met à jour la liste de contrôle d'accès.
  * `/datastore/backups-loutikcloud` : Le chemin ciblant ton espace de sauvegarde.
  * `DatastorePowerUser` : Rôle PBS natif qui inclut les permissions de base (lecture/écriture).
  * `--auth-id svc-pve-backup@pbs` : On cible cette fois-ci l'utilisateur lui-même (et non plus le token avec le `!`), ce qui permettra à ses jetons d'hériter automatiquement de ses droits.

3.3. **Ajout du datastore sur le cluster Proxmox.** Déclaration du serveur PBS en tant que stockage de sauvegarde depuis le shell d'un nœud PVE.

  ```bash
  pvesm add pbs pbs-storage --server mlt1-pbs-vm-prd-01.infra.loutik.fr --datastore backups-loutikcloud --username 'svc-pve-backup@pbs!pve-token' --password <SECRET_TOKEN> --fingerprint <FINGERPRINT_SHA256>
  ```

  * `pvesm add pbs` : Commande (Proxmox VE Storage Manager) pour ajouter un nouveau support de stockage de type "Proxmox Backup Server".
  * `pbs-storage` : L'identifiant (ID) qui sera affiché dans l'interface web Proxmox pour désigner ce stockage.
  * `--server` : Adresse IP ou nom de domaine complet (FQDN) du serveur PBS distant.
  * `--datastore` : Le nom exact de la banque de données créée sur le PBS qui recevra les sauvegardes.
  * `--username` : L'identifiant de connexion, utilisant la syntaxe spécifique `utilisateur@realm!nom_du_token`.
  * `--password` : La valeur secrète du jeton d'API (affichée uniquement lors de sa création à l'étape 3.1).
  * `--fingerprint` : L'empreinte du certificat TLS récupérée précédemment, garantissant qu'il n'y a pas d'interception (attaque Man-in-the-Middle) de la connexion.