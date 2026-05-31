---
title: Supprimer un noeud du cluster
service: Proxmox
date: 2026-05-31
author: Louis MEDO
owner: Louis MEDO
tags: [runbook, exploitation, mco]
---

# {{ page.meta.title }}

![Bannière LoutikCLOUD](/assets/banniere_loutikcloud.png)

!!! info "Informations"

    * **Date de mise à jour** : {{ page.meta.date }}
    * **Service ciblé** : {{ page.meta.service }}
    * **Auteur** : {{ page.meta.author }}
    * **Responsable** : {{ page.meta.owner }}

## 1. Contexte et Objectif

Décommissionnement propre et définitif d'un nœud physique du cluster Proxmox VE au sein de l'infrastructure LoutikCLOUD. Cette procédure doit être exécutée lors d'un retrait de matériel, d'un remplacement de serveur défectueux ou d'une réduction de capacité, afin de maintenir l'intégrité du quorum Corosync sans perturber les services en production.

## 2. Prérequis

* Accès SSH en `root` (via clé asymétrique) à un nœud **actif** du cluster (différent de celui à supprimer).
* L'ensemble des machines virtuelles (VM) et conteneurs (LXC) du nœud cible doivent avoir été migrés vers d'autres nœuds ou détruits.
* Vérification que le cluster dispose de suffisamment de nœuds restants pour maintenir le quorum (idéalement un nombre impair > 2).

## 3. Procédure d'exécution

1. **Extinction du nœud à décommissionner** :

    Il est strictement impératif d'éteindre le nœud avant sa suppression. Le laisser allumé provoquerait un conflit Corosync et une corruption potentielle du cluster. À exécuter sur le nœud à retirer.

    ```bash
    poweroff
    ```

    - **`poweroff`** : Commande système ACPI provoquant l'arrêt propre du système d'exploitation et la mise hors tension de la machine physique.


2. **Vérification de la topologie** :

    À exécuter depuis un nœud restant et actif du cluster pour récupérer le nom exact du nœud hors ligne.

    ```bash
    pvecm nodes
    ```

    - **`pvecm nodes`** : Argument de l'outil Proxmox VE Cluster Manager affichant la liste des membres du cluster, leur ID et leur statut actuel.


3. **Suppression définitive du nœud** :

    À exécuter depuis le même nœud actif. Remplacer `nom-du-noeud` par la valeur trouvée à l'étape précédente.

    ```bash
    pvecm delnode nom-du-noeud
    ```

    - **`pvecm delnode`** : Commande qui supprime les informations du nœud spécifié dans la configuration partagée de Corosync (`corosync.conf`), isolant ainsi définitivement l'ancien serveur de la grappe.
    - **`nom-du-noeud`** : Le nom d'hôte exact du serveur Proxmox à expulser.

## 4. Validation

Vérifier la bonne santé du cluster depuis un nœud actif :

1. Exécuter `pvecm status`. Le champ `Expected votes` doit avoir diminué de 1 et le statut de `Quorum` doit indiquer `Yes`.
2. Exécuter `pvecm nodes`. Le nœud supprimé ne doit plus apparaître dans la liste.
3. Vérifier l'interface web (GUI) de Proxmox : le nœud supprimé ne doit plus être visible dans l'arborescence du datacenter (un rafraîchissement de la page ou un redémarrage du service `pveproxy` peut être nécessaire : `systemctl restart pveproxy`).

## 5. Rollback

**Attention : Un nœud supprimé d'un cluster Proxmox VE ne peut pas être réintégré tel quel.** Si le nœud a été supprimé par erreur, le retour arrière nécessite la réinstallation complète de l'hyperviseur sur la machine cible :

1. Formater et réinstaller Proxmox VE de zéro sur le serveur physique.
2. Reprendre la procédure standard "Création d'un cluster" (ou "Jonction d'un nœud") pour ajouter ce "nouveau" nœud vierge au cluster existant.