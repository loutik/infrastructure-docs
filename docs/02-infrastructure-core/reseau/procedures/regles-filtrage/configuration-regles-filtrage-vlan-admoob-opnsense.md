---
title: Règles de filtrage - Accès Administration (VLAN ADMOOB 20)
service: Réseau
date: 2026-07-03
author: Louis MEDO
owner: Louis MEDO
tags: [deploiement, iac, mise-en-production, securite, firewall]
---

# {{ page.meta.title }}

![Bannière LoutikCLOUD](../../../../assets/banniere_loutikcloud.png)

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

## 3. Configuration de l'accès à l'intergace d'administration OPNsense

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

## 4. Configuration de l'accès Internet via Alias Inverse (RFC 1918)

!!! warning "Accès internet "
    Cette règle est **temporaire**, elle nous permet de pouvoir installer et configurer nos hyperviseur sans avoir besoin du proxy qui n'est pas encore configuré.

4.1. **Création de l'Alias des réseaux privés (Prérequis).** Avant de créer la règle, naviguez dans `Firewall > Aliases`, créez un alias nommé `RFC1918` de type `Network(s)` contenant les sous-réseaux `10.0.0.0/8`, `172.16.0.0/12` et `192.168.0.0/16`.

4.2. **Création de la règle d'autorisation Internet.** Depuis l'interface d'administration, naviguez dans `Firewall > Rules > [Nom de l'interface]` et ajoutez une nouvelle règle en dessous de vos règles d'accès locaux.

  ```bash
  Categories : Internet
  Description : [LAN_ADMOOB_20 -> WAN] Accès Internet
  Interface : LAN_ADMOOB_20
  Action : Pass
  Direction : In
  Version : IPv4
  Protocol : any
  Source : LAN_ADMOOB_20 net
  Source Port : any
  Invert Destination : coché
  Destination : RFC1918
  Destination Port : any
  ```

  `Action` : Définie sur `Pass` pour autoriser le trafic sortant vers les destinations externes.

  `Interface` : Applique le filtrage sur les paquets entrants initiés par la zone réseau spécifiée.

  `Source` : Restreint l'origine du flux au sous-réseau complet (`net`) de l'interface courante.

  `Destination` : Exploite l'alias `RFC1918` combiné à la directive d'inversion (`!`). Le pare-feu bloque ainsi toute tentative de routage vers un autre VLAN interne et n'autorise le flux que vers le WAN.

  `Destination Port` : Positionné sur `any` (ou restreint à `PORT_WEB` / 80, 443) selon le niveau de filtrage applicatif désiré pour la zone.

  `Description` : Identifie le flux selon la nomenclature LoutikCLOUD en spécifiant l'usage de la règle d'inversion pour le WAN.

## Annexe

- [Référentiel des normes de nommage réseau LoutikCLOUD](../../../../01-architecture/referentiels/ref-003-convention-nommage-reseau-pare-feu.md)