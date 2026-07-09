---
title: Jonction d'un nœud au cluster
service: Proxmox
date: 2026-05-31
author: Louis MEDO
owner: Louis MEDO
tags: [proxmox]
---

# {{ page.meta.title }}

![Bannière LoutikCloud](https://raw.githubusercontent.com/loutik/design-assets/main/loutikcloud/banniere_loutikcloud.png)

!!! info "Informations"
    * **Date de création** : {{ page.meta.date }}
    * **Service ciblé** : {{ page.meta.service }}
    * **Auteur** : {{ page.meta.author }}
    * **Responsable** : {{ page.meta.owner }}

## 1. Architecture et contexte
L'ajout d'un nouveau nœud de calcul au cluster Proxmox VE existant de l'infrastructure LoutikCLOUD nécessite de sécuriser les échanges initiaux. La politique de sécurité imposant la désactivation de l'authentification SSH par mot de passe pour le compte `root`, la jonction s'effectue obligatoirement via l'utilisation d'une clé SSH privée asymétrique stockée de manière sécurisée et approuvée par le cluster maître.

## 2. Prérequis

* Un nœud Proxmox VE vierge installé et configuré sur le réseau de management (IP statique documentée dans Netbox).
* Connectivité réseau validée vers le réseau Corosync du cluster existant.
* Clé SSH publique du nouveau nœud déjà présente dans le fichier `~/.ssh/authorized_keys` du nœud maître du cluster.
* Accès au gestionnaire de mots de passe pour récupérer la clé SSH privée d'administration.

## 3. Déploiement et jonction au cluster

1.  **Importation de la clé SSH privée.** Récupération de la clé depuis le gestionnaire de mots de passe et restriction stricte de ses droits d'accès pour respecter les standards de sécurité SSH.

    ```bash
    nano ~/.ssh/id_ed25519_cluster
    chmod 600 ~/.ssh/id_ed25519_cluster
    ```

    `nano` : Éditeur de texte utilisé pour coller le contenu de la clé privée récupérée dans le gestionnaire de mots de passe.

    `chmod 600` : Commande modifiant les permissions du fichier pour qu'il soit lisible et inscriptible uniquement par le propriétaire (`root`), condition requise par le client SSH.

2.  **Chargement de la clé dans l'agent SSH.** Démarrage du processus d'agent en arrière-plan et ajout de la clé pour permettre à Proxmox d'établir la connexion sans demander de mot de passe lors de la jonction.

    ```bash
    eval "$(ssh-agent -s)"
    ssh-add ~/.ssh/id_ed25519_cluster
    ```

    `eval "$(ssh-agent -s)"` : Démarre l'agent SSH et exporte les variables d'environnement nécessaires dans la session courante.

    `ssh-add` : Charge la clé privée spécifiée en mémoire dans l'agent SSH, qui gérera automatiquement l'authentification pour les commandes sous-jacentes.

3.  **Exécution de la commande de jonction.** Lancement du processus d'intégration du nœud au cluster existant via la liaison réseau isolée.

    ```bash
    pvecm add 10.10.10.11 --link0 10.10.10.12
    ```

    `pvecm add` : Commande principale du gestionnaire de cluster Proxmox (Proxmox VE Cluster Manager) initiant l'ajout d'un nœud.

    `10.10.10.11` : Adresse IP du nœud maître du cluster auquel le serveur actuel doit se rattacher.

    `--link0 10.10.10.12` : Spécifie l'adresse IP de l'interface réseau locale dédiée au trafic Corosync pour ce nouveau nœud.

4.  **Validation de la topologie du cluster.** Vérification post-déploiement pour confirmer que le nouveau nœud a correctement rejoint le quorum et que la synchronisation est active.

    ```bash
    pvecm nodes
    ```

    `nodes` : Argument de `pvecm` qui liste tous les nœuds reconnus par le cluster, leurs identifiants (Node ID) et leur état de connexion actuel.

## Annexe

- [Documentation officielle Proxmox VE - Cluster Manager (pvecm)](https://pve.proxmox.com/pve-docs/chapter-pvecm.html)