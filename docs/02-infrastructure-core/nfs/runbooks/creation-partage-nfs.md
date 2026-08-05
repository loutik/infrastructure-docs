---
title: Création d'un partage NFS
service: Stockage / NFS
date: 2026-08-05
author: Louis MEDO
owner: Louis MEDO
---

# {{ page.meta.title }}

![Bannière LoutikCloud](https://raw.githubusercontent.com/loutik/design-assets/main/loutikcloud/banniere_loutikcloud.png)

!!! info "Informations"

    * **Date de mise à jour** : {{ page.meta.date }}
    * **Service ciblé** : {{ page.meta.service }}
    * **Auteur** : {{ page.meta.author }}
    * **Responsable** : {{ page.meta.owner }}

!!! warning "Avertissement"
    La configuration et l'installation de cette infrastructure sont gérées intégralement par Ansible. Les commandes présentées ci-dessous illustrent le processus manuel équivalent à des fins de compréhension ou de débogage. Aucune création ou modification manuelle ne doit être effectuée dans des conditions de production.

## 1. Contexte

L'objectif de ce runbook est de documenter la création manuelle d'un nouveau partage NFS, en reproduisant l'architecture d'isolation (création d'un UID/GID et groupe système dédiés) définie dans le rôle Ansible `debian-nfs` de l'infrastructure LoutikCLOUD. Cette procédure doit être exécutée pour provisionner un espace de stockage en urgence ou pour qualifier une configuration avant son automatisation dans le code d'infrastructure (IaC).

## 2. Prérequis

* Accès SSH au serveur de stockage avec privilèges `sudo`.
* Définition des paramètres cibles (exemple utilisé ci-dessous : Nom `app-data`, UID/GID `2050`, Chemin `/srv/nfs/app-data`, Réseau autorisé `192.168.10.0/24`).

## 3. Procédure d'exécution

1. **Création du groupe d'isolation système** :

    Création d'un groupe dédié pour garantir que l'accès au répertoire physique soit cloisonné.

    ```bash
    sudo groupadd --system --gid 2050 app-data
    ```

    - **`groupadd`** : Commande d'ajout de groupe local.
    - **`--system`** : Indique au système d'exploitation qu'il s'agit d'un groupe de service.
    - **`--gid 2050`** : Définit statiquement l'identifiant du groupe pour maintenir la cohérence de l'infrastructure.

2. **Création de l'utilisateur d'isolation système** :

    Génération d'un compte de service sans accès interactif, rattaché au groupe d'isolation.

    ```bash
    sudo useradd --system --uid 2050 --gid app-data --no-create-home --shell /usr/sbin/nologin app-data
    ```

    - **`useradd`** : Commande d'ajout d'un utilisateur local.
    - **`--no-create-home`** : Empêche la génération d'un répertoire personnel par défaut.
    - **`--shell /usr/sbin/nologin`** : Désactive l'ouverture de session pour ce compte de service.

3. **Création de l'espace de stockage et assignation des permissions** :

    Création du répertoire d'exportation sur le disque et application stricte des droits au compte de service.

    ```bash
    sudo mkdir -p /srv/nfs/app-data && sudo chown 2050:2050 /srv/nfs/app-data && sudo chmod 0750 /srv/nfs/app-data
    ```

    - **`mkdir -p`** : Crée le répertoire cible ainsi que l'arborescence parente si nécessaire.
    - **`chown`** : Assigne la propriété du dossier au nouvel utilisateur et groupe d'isolation.
    - **`chmod 0750`** : Restreint l'accès (Lecture/Écriture/Exécution pour le propriétaire, Lecture/Exécution pour le groupe, aucun droit pour les autres).

4. **Déclaration du partage dans la configuration NFS** :

    Ajout de la règle d'exportation, forçant les clients réseau à hériter des droits du compte de service.

    ```bash
    echo "/srv/nfs/app-data 192.168.10.0/24(rw,sync,root_squash,all_squash,anonuid=2050,anongid=2050)" | sudo tee -a /etc/exports
    ```

    - **`tee -a`** : Ajoute la configuration de manière sécurisée à la fin du fichier `/etc/exports`.
    - **`all_squash,anonuid=2050,anongid=2050`** : Force toutes les connexions clientes, y compris root, à être traitées sous l'UID/GID 2050 défini pour ce partage.

5. **Rechargement de la table des exports** :

    Application de la nouvelle configuration NFS sans interruption globale du service.

    ```bash
    sudo exportfs -ra
    ```

    - **`exportfs`** : Outil de gestion de la table d'exportation.
    - **`-ra`** : Relit le fichier `/etc/exports` (`-r`) et applique toutes les configurations (`-a`).

## 4. Validation

* Exécuter la commande `sudo exportfs -v` sur le serveur : vérifier que le répertoire `/srv/nfs/app-data` est présent avec les options `anonuid=2050` et `anongid=2050`.
* Depuis une machine cliente dans le sous-réseau autorisé, exécuter `showmount -e <IP_SERVEUR>` pour vérifier l'annonce du partage.
* Monter le partage sur le client (`sudo mount -t nfs4 <IP_SERVEUR>:/srv/nfs/app-data /mnt`), créer un fichier test (`touch /mnt/test.txt`), et vérifier côté serveur que le fichier appartient bien à l'utilisateur `app-data` (UID 2050).