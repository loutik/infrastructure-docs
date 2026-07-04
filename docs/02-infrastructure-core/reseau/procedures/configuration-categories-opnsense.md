---
title: Configuration des catégories - OPNsense
service: Réseau
date: 2026-07-04
author: Louis MEDO
owner: Louis MEDO
tags: [deploiement, opnsense]
---

# {{ page.meta.title }}

!!! info "Informations"
    * **Date de création** : {{ page.meta.date }}
    * **Service ciblé** : {{ page.meta.service }}
    * **Auteur** : {{ page.meta.author }}
    * **Responsable** : {{ page.meta.owner }}

## 1. Contexte

Pour structurer la politique de filtrage et maintenir la scannabilité de l'IHM d'OPNsense, l'attribution de catégories de pare-feu est obligatoire sur l'infrastructure LoutikCLOUD. Plutôt que de multiplier les règles en cascade, les catégories permettent de regrouper visuellement les flux par intention technique ou zone architecturale, de faciliter le filtrage dans la vue en direct et d'automatiser les rapports de conformité. Ce document liste de manière exhaustive les catégories validées pour l'infrastructure et détaille la procédure de provisionnement manuel.

## 2. Référentiel des catégories

Les catégories pour l'ensemble des règles de flux du pare-feu :

| Catégorie | Description technique associée | Code Couleur |
| --- | --- | --- |
| **Management** | Flux d'administration du routeur (HTTPS, SSH) et accès aux hyperviseurs. | `#FF8080` |
| **Internet** | Flux de sortie WAN directe ou via exclusion locale (RFC 1918). | `#80FF80` |
| **Proxy** | Flux acheminés vers ou depuis le cluster de proxies applicatifs (Squid). | `#80FFFF` |
| **Services** | Flux vers les services partagés internes (DNS, NTP, serveurs web internes). | `#FFFF80` |
| **Database** | Flux stricts de requêtes vers la zone hermétique des bases de données (ZDS). | `#FF80FF` |
| **VPN** | Flux d'interconnexion chiffrés pour le tunnel (Tailscale). | `#9999FF` |
| **System** | Flux techniques d'infrastructure (Supervision, Sauvegardes, Déploiements PXE). | `#CCCCCC` |
| **InterVLAN** | Flux d'échange transversaux autorisés de zone à zone (ex: Utilisateurs vers DMZ). | `#FFCC99` |

## 3. Procédure de création des catégories sur OPNsense

3.1. **Navigation vers le module.** Connectez-vous à la WebUI d'OPNsense depuis le VLAN ADMOOB. Dans le menu latéral gauche, accédez à l'emplacement suivant :

  ```bash
  Firewall > Category
  ```

  `Category` : Ce sous-menu regroupe l'ensemble des étiquettes applicables aux règles de pare-feu pour le tri et le reporting.

3.2. **Provisionnement d'une catégorie.** Cliquez sur le bouton **`+`** (Ajouter) en haut à droite de la grille pour ouvrir la fenêtre de configuration.

  ```bash
  Name : [Nom_de_la_catégorie]
  Auto : Coché
  Color : [Code_Couleur_Hexadécimal]
  ```

  `Name` : Le nom doit obligatoirement correspondre à un mot unique issu du référentiel ci-dessus (respecter la casse).

  `Auto` : L'activation de cette option permet à OPNsense d'inclure automatiquement la catégorie dans les filtres dynamiques de la Live View du pare-feu.

  `Color` : Attribuez la couleur spécifiée dans le tableau pour assurer l'homogénéité visuelle lors de la lecture des règles de l'interface.

## Annexe

* [Référentiel des normes de nommage réseau LoutikCLOUD](/01-architecture/referentiels/ref-003-convention-nommage-reseau-pare-feu.md)