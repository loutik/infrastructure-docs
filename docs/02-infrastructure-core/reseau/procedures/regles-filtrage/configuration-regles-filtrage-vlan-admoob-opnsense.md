---
title: Règles de filtrage - Accès Administration (VLAN ADMOOB 20)
service: Réseau
date: 2026-07-03
author: Louis MEDO
owner: Louis MEDO
tags: [deploiement, iac, mise-en-production, securite, firewall]
---

# {{ page.meta.title }}

![Bannière LoutikCLOUD](/assets/banniere_loutikcloud.png)

!!! info "Informations"
    * **Date de création** : {{ page.meta.date }}
    * **Service ciblé** : {{ page.meta.service }}
    * **Auteur** : {{ page.meta.author }}
    * **Responsable** : {{ page.meta.owner }}

## 1. Architecture et contexte
Cette procédure documente la mise en place de la règle de filtrage autorisant l'accès au plan de contrôle du routeur OPNsense (WebGUI et API) depuis le réseau d'administration dédié (VLAN ADMOOB 20). Cette configuration répond au principe de moindre privilège et sécurise l'infrastructure en isolant l'administration du reste des flux utilisateurs ou services. 

## 2. Prérequis

* L'interface réseau logique correspondant au VLAN ADMOOB 20 (ex: `LAN_ADMOOB_20`) est assignée et active.
* Le plan d'adressage IP du VLAN ADMOOB 20 est fonctionnel et la passerelle OPNsense est accessible (niveau 2/3).
* Les alias de ports pour l'administration (ex: `PORT_WEB_ADMIN`, `PORT_SSH_ADMIN`) sont préalablement déclarés dans le pare-feu.

## 3. Configuration de la règle d'accès

3.1. **Création de la règle d'autorisation.** Depuis l'interface d'administration, naviguez dans `Firewall > Rules > [Nom de l'interface ADMOOB 20]` (ou `Rules [new]`) et ajoutez une nouvelle règle en tête de liste pour garantir l'accès au pare-feu.

  ```bash
  Action : Pass
  Interface : LAN_ADMOOB_20
  Direction : In
  Protocol : TCP
  Source : LAN_ADMOOB_20 net
  Destination : This Firewall
  Destination Port : PORT_WEB_ADMIN,
  Category : Management
  Description : [ADMOOB -> This Firewall] Accès à l'administration du routeur
  ```

  `Action` : Définie sur `Pass` pour autoriser explicitement le trafic correspondant.

  `Interface` : Limite l'écoute de cette règle au trafic entrant par l'interface physique ou logique du VLAN ADMOOB 20.

  `Source` : Utilise le sous-réseau complet du VLAN ADMOOB 20 (`net`) ou un alias regroupant les machines d'administration (ex: les postes bastion).

  `Destination` : Cible l'objet système dynamique `This Firewall` (ou `Self`), qui englobe toutes les adresses IP du routeur.

  `Destination Port` : Restreint l'accès aux seuls ports nécessaires à l'administration (HTTPS pour l'IHM/API, TCP/22 pour SSH).

  `Description` : Contextualise la règle selon la nomenclature LoutikCLOUD en précisant la source, la destination et le but du flux.

3.2. **Application de la politique de sécurité.** Enregistrez la règle puis rechargez le moteur de filtrage pour que les modifications soient appliquées en mémoire.

  ```bash
  Apply Changes : Clic sur le bouton d'application
  ```

  `Apply Changes` : Valide la nouvelle matrice de flux et recharge `pf` sans interrompre les connexions actives établies.

## Annexe

- [Référentiel des normes de nommage réseau LoutikCLOUD](/01-architecture/referentiels/ref-003-convention-nommage-reseau-pare-feu.md)