---
title: Préparation du stockage externalisé Nextcloud pour les sauvegardes Proxmox
service: Nextcloud / WebDAV
date: 2026-07-25
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
Mise en place d'un espace de stockage à froid (cold storage) externalisé sur l'instance Nextcloud ([drive.loutik.fr](https://drive.loutik.fr)). Cet espace accueillera les archives complètes (`.vma.zst`) générées par le cluster Proxmox VE via un flux WebDAV. L'architecture repose sur le principe de moindre privilège : les données sont hébergées et maîtrisées par le compte administrateur (`tiring`), qui délègue un accès strict et limité à un répertoire spécifique pour un compte de service automatisé (`svc-proxmox-backup`). 

## 2. Prérequis

* Accès administrateur au compte `tiring` sur l'instance Nextcloud cible.
* Espace de stockage suffisant (quota) alloué sur l'hébergement mutualisé pour accueillir les archives.
* URL racine WebDAV de l'instance Nextcloud (généralement `https://[domaine]/remote.php/webdav/`).

## 3. Configuration des comptes et de l'espace de stockage

3.1. **Création du compte de service.** Depuis l'interface d'administration Nextcloud (connecté avec un compte d'administration global), instancier l'utilisateur dédié aux opérations automatisées.

  ```text
  Utilisateurs > Nouvel utilisateur
  Identifiant : svc-proxmox-backup
  Mot de passe : [Générer un mot de passe fort aléatoire]
  ```

  * `Identifiant` : Nom d'utilisateur standardisé désignant un compte de service (Service Account) exclusif à Proxmox.
  * `Mot de passe` : Clé de connexion initiale (ne sera pas utilisée dans les scripts de sauvegarde finaux).

3.2. **Création de l'arborescence cible.** Depuis l'interface "Fichiers" du compte propriétaire `tiring`, structurer l'espace d'accueil des sauvegardes.

  ```text
  Créer dossier : infrastructure-backup
  Entrer dans infrastructure-backup > Créer sous-dossier : proxmox-vm
  ```

  * `infrastructure-backup` : Dossier racine de niveau 1 servant à isoler logiquement toutes les données liées à l'infrastructure des fichiers personnels du compte `tiring`.
  * `proxmox-vm` : Dossier cible final de niveau 2 qui sera exposé au client de sauvegarde.

3.3. **Partage et attribution des permissions.** Toujours depuis le compte `tiring`, déléguer l'accès du dossier cible au compte de service.

  ```text
  Partager le dossier 'proxmox-vm'
  Utilisateur cible : svc-proxmox-backup
  Droits : Cochez "Autoriser la modification" (Création, Modification, Suppression)
  ```

  * `Partage` : Mécanisme permettant à `tiring` de rester propriétaire exclusif des données tout en autorisant le dépôt par un tiers.
  * `Autoriser la modification` : Droit obligatoire pour permettre à Proxmox d'écrire de nouvelles archives et d'exécuter sa tâche de rétention (suppression des anciens fichiers).

3.4. **Génération du mot de passe d'application.** Se connecter à l'interface Nextcloud avec le compte de service `svc-proxmox-backup` pour générer un jeton d'accès API/WebDAV.

  ```text
  Paramètres personnels > Sécurité > Appareils et sessions
  Nom de l'application : proxmox-rclone-webdav
  Cliquer sur "Créer un mot de passe d'application"
  ```

  * `Nom de l'application` : Identifiant textuel pour identifier rapidement la provenance des connexions dans les journaux d'audit.
  * `Mot de passe d'application` : Chaîne de caractères unique générée par le système. Elle contourne la double authentification (2FA) et doit être copiée immédiatement pour être intégrée comme secret dans le client rclone sur le nœud Proxmox.

---

## Annexe

- [Documentation officielle Nextcloud - Clients WebDAV](https://docs.nextcloud.com/server/latest/user_manual/en/files/access_webdav.html)