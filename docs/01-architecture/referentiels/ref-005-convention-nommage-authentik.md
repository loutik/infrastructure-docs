---
id: 005
title: Convention de nommage des objets Authentik
category: Nommage
date: 2026-07-19
author: Louis MEDO
tags: [referentiel, norme, standard, convention, authentik, sso]
---

# REF-{{ page.meta.id }} : {{ page.meta.title }}

![Bannière LoutikCloud](https://raw.githubusercontent.com/loutik/design-assets/main/loutikcloud/banniere_loutikcloud.png)

!!! info "Informations"
    * **Date d'application** : {{ page.meta.date }}
    * **Catégorie** : {{ page.meta.category }}
    * **Garant de la norme** : {{ page.meta.author }}

## 1. Objectif
Standardiser le nommage des groupes, applications et fournisseurs (providers) dans Authentik afin de faciliter l'attribution et l'automatisation des droits via OIDC sur l'infrastructure (ex: Proxmox, NetBox).

## 2. Conventions
La structure globale repose sur un format strict composé de 5 segments séparés par des tirets (`-`).

* **Format d'expression régulière (Regex)** : `^[a-z0-9]+(-[a-z0-9]+){3}$`
* **Structure globale** : `<domaine>-<service>-<environnement>-<role>`

| Segment | Rôle | Valeurs autorisées |
| :--- | :--- | :--- |
| **`[domaine]`** (Domaine technique) | Identifie la catégorie d'infrastructure. | `inf` (Infra), `app` (Applicatif), `net` (Réseau) |
| **`[service]`** (Service cible) | Identifie l'application ou le service lié. | `pve` (Proxmox), `ntb` (NetBox), `glpi` |
| **`[environnement]`** (Environnement) | Définit le contexte de déploiement. | `prd` (Production), `stg` (Staging), `lab` (Lab) |
| **`[role]`** (Rôle d'accès) | Définit le niveau de permission accordé. | `adm` (Admin), `usr` (User), `aud` (Audit) |

!!! note "Contraintes spécifiques"
    L'utilisation de lettres minuscules est strictement obligatoire. Aucun espace ni caractère spécial (hors tiret) n'est toléré pour garantir la compatibilité avec les claims OIDC.

## 3. Exemples pratiques

* ✅ **Conforme** : 
    * `inf-pve-prd-adm` (Groupe d'administration infrastructure Proxmox en production géré par LoutikSSO)
    * `app-glpi-lab-usr` (Groupe d'utilisateurs standards GLPI en lab géré par LoutikSSO)
* ❌ **Non conforme** : 
    * `INF_PVE_PRD_ADM` (Invalide : majuscules, mauvais délimiteur `_`, manque le segment IDP)
    * `inf-proxmox-prd-admin` (Invalide : non-respect des trigrammes de service et de rôle)

## 4. Exceptions
Les groupes intégrés par défaut par Authentik (ex: `authentik Admins`) ne sont pas soumis à cette règle afin de ne pas altérer le fonctionnement interne du système.