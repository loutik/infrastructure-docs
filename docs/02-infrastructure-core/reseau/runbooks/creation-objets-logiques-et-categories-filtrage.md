---
title: Création des objets logiques et catégories de filtrage
service: Réseau
date: 2026-06-30
author: Louis MEDO
owner: Louis MEDO
tags: [deploiement, reseau, alias, objets-logiques, categories]
---

# {{ page.meta.title }}

![Bannière LoutikCloud](https://raw.githubusercontent.com/loutik/design-assets/main/loutikcloud/banniere_loutikcloud.png)

!!! info "Informations"
    * **Date de création** : {{ page.meta.date }}
    * **Service ciblé** : {{ page.meta.service }}
    * **Auteur** : {{ page.meta.author }}
    * **Responsable** : {{ page.meta.owner }}

## 1. Architecture et contexte

Cette procédure formalise la création des objets logiques (Alias) et des structures d'organisation (Catégories) au sein du moteur de filtrage d'OPNsense. Elle abstrait les variables réseau (IP, CIDR, ports) pour découpler l'infrastructure réseau de la logique des règles. La déclaration stricte de ces objets est indispensable avant d'exploiter l'interface de règles modernes `Rules [new]` et garantit la conformité avec la matrice de flux globale de LoutikCLOUD.

## 2. Prérequis

* La procédure de création des VLANs et interfaces est entièrement finalisée et validée.
* Consultation du référentiel **REF-003 (Convention de nommage réseau et pare-feu)**, Sections 2.2 et 2.3 pour valider les règles de casse et de préfixage.
* Schéma de la cartographie réseau à disposition pour relever les identifiants de VLANs et les adresses de sous-réseaux.

## 3. Création des catégories de filtrage

3.1. **Déclaration des structures d'organisation.** Depuis l'interface Web, naviguez dans `Firewall > Category` et cliquez sur le bouton "+" pour ajouter les catégories qui serviront à classifier les flux métiers.

  ```bash
  Name : Internet-Access
  Auto color : Coché
  ```

  `Name` : Nom de la catégorie respectant les valeurs autorisées (`Internet-Access`, `Inter-VLAN`, `Management`) du **REF-003, Section 2.3**.

  `Auto color` : Attribue dynamiquement une couleur d'identification visuelle pour le repérage dans l'IHM.

## 4. Création des alias réseau (NET_ et HOST_)

4.1.  **Déclaration des alias de sous-réseaux.** Naviguez dans `Firewall > Aliases`, cliquez sur le bouton "+" et déclarez les sous-réseaux d'infrastructure d'après la cartographie LoutikCLOUD.

  ```bash
  Name : NET_DMZI_SERVICES
  Type : Network(s)
  Content : 10.0.12.0/24
  Description : Réseau complet du VLAN 12 dédié aux clusters et services internes
  ```

  `Name` : Nommage en majuscules avec le préfixe strict `NET_` suivi de la description en snake_case (**REF-003, Section 2.2**).

  `Type` : Sélectionnez `Network(s)` pour englober un masque CIDR complet.

  `Content` : Bloc d'adresses IPv4 correspondant (ex: `10.0.12.0/24` pour la DMZ Interne, `10.0.20.0/24` pour le LAN Administration).

4.2. **Déclaration des alias d'hôtes uniques.** Répétez l'opération dans `Firewall > Aliases` pour isoler des machines spécifiques nécessitant des flux unitaires.

  ```bash
  Name : HOST_DEBIAN_CLI_01
  Type : Host(s)
  Content : 10.0.20.254
  Description : Poste client Debian d'administration réseau
  ```

  `Name` : Nommage en majuscules avec le préfixe strict `HOST_` suivi du nom système de la machine (**REF-003, Section 2.2**).

  `Type` : Sélectionnez `Host(s)` pour restreindre l'alias à une adresse IP unique.

## 5. Création des alias de services (PORT_)

5.1. **Déclaration des groupes de ports.** Toujours dans `Firewall > Aliases`, créez les abstractions de ports applicatifs pour rationaliser l'écriture des règles.

  ```bash
  Name : PORT_SSH_ADMIN
  Type : Port(s)
  Content : 22
  Description : Port standard dédié au protocole de communication sécurisé SSH
  ```

  `Name` : Nommage en majuscules avec le préfixe strict `PORT_` suivi du protocole métier associé (**REF-003, Section 2.2**).

  `Type` : Sélectionnez `Port(s)` pour encapsuler des numéros de ports TCP/UDP individuels ou en listes séparées par des virgules.

## Annexe

- [REF-003 - Convention de nommage réseau et pare-feu](../../../01-architecture/referentiels/ref-003-convention-nommage-reseau-pare-feu.md)