---
title: Configuration des alias - OPNsense
service: Réseau
date: 2026-07-04
author: Louis MEDO
owner: Louis MEDO
tags: [deploiement, opnsense, alias]
---

# {{ page.meta.title }}

![Bannière LoutikCloud](https://raw.githubusercontent.com/loutik/design-assets/main/loutikcloud/banniere_loutikcloud.png)

!!! info "Informations"
    * **Date de création** : {{ page.meta.date }}
    * **Service ciblé** : {{ page.meta.service }}
    * **Auteur** : {{ page.meta.author }}
    * **Responsable** : {{ page.meta.owner }}

## 1. Contexte

Pour éviter la redondance d'adresses IP ou de numéros de ports codés en dur au sein des règles de filtrage, l'infrastructure LoutikCLOUD s'appuie sur une gestion par Alias sur le routeur OPNsense. Les alias agissent comme des variables ou des objets nommés, simplifiant la maintenance opérationnelle : la modification d'un élément au sein d'un alias répercute instantanément le changement sur toutes les règles de filtrage associées sans devoir les éditer une par une.

## 2. Alias du routeur

### 2.1. Alias de ports

| Nom de l'Alias | Contenu (Ports / Protocoles) | Description |
| -------------- | ---------------------------- | ----------- |
| **PORT_WEB_ADMIN** | `80`, `443` | Ports d'accès HTTP et HTTPS pour l'IHM et l'API d'OPNsense. |
| **PORT_PROXY** | `3128` (TCP) | Port de communication standard du cluster de proxy Squid. |

### 2.2. Alias de réseaux

| Nom de l'Alias | Contenu | Description |
| --- | --- | --- |
| **RFC1918** | `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16` | Ensemble complet des espaces d'adressage privés locaux (utilisé pour l'inversion Internet). |

## 3. Procédure de création des alias sur OPNsense

3.1. **Création d'un alias de ports.** Connectez-vous à la WebUI d'OPNsense depuis le VLAN ADMOOB. Naviguez dans `Firewall > Aliases` et cliquez sur le bouton `+` (Ajouter).

  ```bash
  Enabled : Coché
  Name : TYPE_DESCRIPTION (ex. PORT_WEB_ADMIN) 
  Type : Port
  Content : 80, 443
  Description : Interface web de gestion du routeur
  ```

  `Name` : Renseignez le nom exact en lettres majuscules selon la nomenclature défini dans les référentiels.

  `Type` : Sélectionné sur `Port` pour indiquer au moteur que l'objet contient des ports TCP/UDP.

  `Content` : Saisissez les valeurs numériques des ports (une entrée par ligne).

## Annexe

* [Référentiel des normes de nommage réseau LoutikCLOUD](../../../01-architecture/referentiels/ref-003-convention-nommage-reseau-pare-feu.md)