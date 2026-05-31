---
title: Création d'un cluster
service: Proxmox
date: 2026-05-31
author: Louis MEDO
owner: Louis MEDO
tags: [proxmox]
---

# {{ page.meta.title }}

![Bannière LoutikCLOUD](/assets/banniere_loutikcloud.png)

!!! info "Informations"
    * **Date de création** : {{ page.meta.date }}
    * **Service ciblé** : {{ page.meta.service }}
    * **Auteur** : {{ page.meta.author }}
    * **Responsable** : {{ page.meta.owner }}

## 1. Architecture et contexte

Le cluster Proxmox VE constitue la couche de virtualisation bare-metal de base de l'infrastructure LoutikCLOUD. Sa création permet l'activation de la haute disponibilité (HA), la migration à chaud des machines virtuelles et conteneurs (LXC), ainsi que la gestion centralisée du quorum Corosync. Ce composant est critique pour l'orchestration résiliente des ressources en amont des déploiements K3s.

## 2. Prérequis

* Au minimum 3 nœuds physiques Proxmox VE pré-installés avec la même version de l'hyperviseur et synchronisés via NTP.
* Un réseau de management/Corosync (VLAN dédié) configuré et isolé, avec une latence inférieure à 2ms et des adresses IP statiques documentées dans Netbox.
* Accès SSH en `root` configuré par clés asymétriques sur l'ensemble des nœuds à associer.

## 3. Initialisation et déploiement du cluster

1.  **Création du cluster Corosync.** Initialisation de l'entité cluster sur le premier nœud physique (nœud maître initial) pour activer la communication inter-nœuds.

    ```bash
    pvecm create loutik-cluster --link0 10.10.10.11
    ```

    `loutik-cluster` : Nom unique attribué au cluster au sein de l'infrastructure (ne peut plus être modifié par la suite).

    `--link0 10.10.10.11` : Définition de l'adresse IP de l'interface réseau stricte et isolée dédiée au trafic Corosync du nœud actuel.

2.  **Jonction des nœuds secondaires.** Ajout des autres serveurs physiques au cluster nouvellement créé, en exécutant cette commande depuis le shell de chaque nouveau nœud.

    ```bash
    pvecm add 10.10.10.11 --link0 10.10.10.12
    ```

    `10.10.10.11` : Adresse IP du nœud initial du cluster auquel le serveur doit se connecter.

    `--link0 10.10.10.12` : Adresse IP de l'interface réseau Corosync spécifique au nœud rejoignant le cluster.

3.  **Validation de l'état du cluster.** Vérification du quorum, de l'état des liaisons Corosync et de la synchronisation des nœuds post-déploiement.

    ```bash
    pvecm status
    ```

    `status` : Argument permettant d'afficher l'état du quorum, le total des votes, la liste des nœuds actifs et la santé globale de la grappe.
    
## Annexe

- [Documentation officielle Proxmox VE - Cluster Manager (pvecm)](https://pve.proxmox.com/pve-docs/chapter-pvecm.html)