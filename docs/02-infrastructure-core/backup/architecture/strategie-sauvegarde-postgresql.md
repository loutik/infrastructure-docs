---
title: Stratégie de sauvegarde PostgreSQL
service: PostgreSQL
date: 2026-07-26
author: Louis MEDO
owner: Louis MEDO
---

# {{ page.meta.title }}

!!! info "Informations"
    * **Date de création** : {{ page.meta.date }}
    * **Service ciblé** : {{ page.meta.service }}
    * **Auteur** : {{ page.meta.author }}
    * **Responsable** : {{ page.meta.owner }}

## 1. Objectifs de l'implémentation

L'architecture de sauvegarde de la base de données PostgreSQL au sein de LoutikCLOUD repose sur les objectifs :

* **Découplage Stateful / Stateless** : Isolation de la donnée (le dump SQL) du système d'exploitation de la machine virtuelle. En cas de compromission, l'OS est détruit et recréé, seule la donnée est restaurée.
* **Principe KISS (Keep It Simple, Stupid)** : Exécution locale des tâches pour limiter le nombre de dépendances, de points de défaillance réseau et de serveurs intermédiaires (absence de relais NFS).
* **Résilience via la règle 3-2-1** : Double expédition de la sauvegarde depuis la source vers un stockage local haute performance (PBS) et un stockage externalisé à froid (Nextcloud).

## 2. Architecture

L'architecture repose sur un pipeline séquentiel déclenché directement au sein de la machine virtuelle hébergeant l'instance PostgreSQL, piloté par un orchestrateur externe (Ansible).

### 2.1. Topologie Logique

```text
                          [ Orchestrateur Ansible ]
                                     | (Déclenchement du Playbook)
                                     v
+-------------------------------------------------------------------------+
| VM POSTGRESQL (Stateless)                                               |
|                                                                         |
|  1. Extraction Logique                                                  |
|     (pg_dump)                                                           |
|         |                                                               |
|         v                                                               |
|  [ /tmp/backup.sql ]  -- 2. proxmox-backup-client --+                   |
|  (Espace Volatile)    --    (Déduplication pxar)    |                   |
|         |                                           |                   |
|         +--------------- 3. rclone copy ------------|---+               |
|                             (Fichier Monolithique)  |   |               |
|                                                     |   |               |
|  4. Nettoyage                                       |   |               |
|     (rm /tmp/...)                                   |   |               |
+-----------------------------------------------------+   |               |
                                                      |   |               |
                     +--------------------------------+   +-----------+   |
                     |                                                |   |
                     v                                                v   v
    +---------------------------------+             +-----------------------------------+
    | PROXMOX BACKUP SERVER (Chaud)   |             | NEXTCLOUD WEBDAV (Froid)          |
    |---------------------------------|             |-----------------------------------|
    | - Haute performance             |             | - Hébergement mutualisé (250 Go)  |
    | - Transfert Delta (Blocs)       |             | - Isolations par fichiers         |
    | - Rétention Courte              |             | - Rétention Longue                |
    +---------------------------------+             +-----------------------------------+
```

### 2.2. Mécanismes de fonctionnement

Le processus d'encapsulation et de transport de la donnée se déroule en quatre phases strictes :

1. **Extraction (PostgreSQL)** : Exécution de la commande native `pg_dump`. La base de données est exportée sous forme de requêtes SQL brutes dans le répertoire volatile `/tmp/`.
2. **Sauvegarde chaude (PBS)** : Le client `proxmox-backup-client` découpe (chunking), hache et compare le fichier `.sql` local avec le serveur PBS. Seuls les blocs de données modifiés sont transférés via le port 8007 pour une déduplication optimale.
3. **Sauvegarde froide (WebDAV)** : L'utilitaire `rclone` utilise la sous-commande `copy` pour téléverser le fichier monolithique généré de `/tmp/` vers le dossier cible sur Nextcloud, évitant la saturation de la base de données de l'hébergeur.
4. **Nettoyage** : Suppression immédiate de l'archive dans `/tmp/` pour préserver l'espace de stockage de la VM.

## 3. Mécanismes de rétention

La purge des anciennes sauvegardes est gérée directement par le client lors du déroulement du pipeline Ansible, garantissant le respect des quotas de stockage.

* **Datastore PBS** : Exécution de la sous-commande `proxmox-backup-client prune` ciblant le namespace de la base de données, avec le drapeau `--keep-last 7`.
* **Dossier Nextcloud** : Exécution de la commande `rclone delete` avec le filtre `--min-age 7d`. Ce mécanisme déclaratif ordonne à l'API WebDAV de purger exclusivement les fichiers dépassant l'ancienneté autorisée.

## 4. Gestion des secrets et Sécurité

Le principe du moindre privilège est appliqué pour garantir que la compromission de la VM de base de données ne permette pas l'altération des sauvegardes ou du système central.

**Configuration Rclone (Nextcloud)** :

* Utilisation exclusive d'un **Mot de passe d'application** spécifique à rclone, généré via l'utilisateur de service `svc-postgresql-backup` sur Nextcloud.
* Ce mot de passe est rendu illisible dans le fichier `rclone.conf` de la VM via la fonction `rclone obscure`.

**Configuration client (PBS)** :

* Authentification auprès du serveur PBS via un jeton API (API Token) dédié à l'utilisateur `svc-postgresql-backup`, restreint en écriture et en ajout sur le Datastore de sauvegarde.
* Le mot de passe de la variable d'environnement `PBS_PASSWORD` est provisionné de manière éphémère par Ansible lors de l'exécution, sans jamais être stocké en clair sur le disque de la machine virtuelle.