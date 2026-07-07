---
id: 004
title: Convention de nommage PostgreSQL
category: Nommage
date: 2026-07-06
author: Louis MEDO
tags: [referentiel, norme, standard, convention]
---

# REF-{{ page.meta.id }} : {{ page.meta.title }}

![Bannière LoutikCLOUD](../../assets/banniere_loutikcloud.png)

!!! info "Informations"
    * **Date d'application** : {{ page.meta.date }}
    * **Catégorie** : {{ page.meta.category }}
    * **Garant de la norme** : {{ page.meta.author }}

## 1. Objectif et périmètre
Cette norme définit les conventions de nommage applicables aux ressources PostgreSQL (bases de données, comptes de service et groupes d'administration) hébergées sur l'infrastructure LoutikCLOUD. Son objectif est de garantir la lisibilité des métriques et des journaux d'audit (logs), d'éviter les ambiguïtés et les collisions de noms, et d'identifier immédiatement le rôle et le niveau de privilège (DDL vs DML) d'une entité.

!!! warning "Limites du périmètre"
    Cette convention ne s'applique pas aux comptes administrateurs humains (DBA nominatifs), qui doivent utiliser l'identifiant standard de l'utilisateur (ex: `louismedo`).

## 2. Conventions
La structure globale repose sur un format strict composé de 2 segments séparés par des underscores (`_`).

* **Format d'expression régulière (Regex)** : `^(db|usr|role_adm)_[a-z0-9]+$`
* **Structure globale** : `[type]_[service]`

| Segment | Rôle | Valeurs autorisées |
| :--- | :--- | :--- |
| **`[type]`** (Typologie) | Identifie la nature de la ressource PostgreSQL et son niveau de privilège. | `db` (Base de données), `usr` (Utilisateur applicatif - DML), `role_adm` (Groupe d'administration - DDL) |
| **`[service]`** (Application) | Définit le nom du service ou de l'application consommant la ressource. | Nom de l'application en minuscules, sans espaces (ex: `authentik`, `netbox`, `opsbricks`) |

!!! note "Contraintes spécifiques"
    L'utilisation de majuscules, d'espaces et de tirets (`-`) est strictement interdite pour assurer la compatibilité avec le moteur PostgreSQL et les outils d'observabilité.

## 3. Exemples pratiques

* ✅ **Conforme** : 
    * `db_authentik` (Base de données allouée au service Authentik)
    * `usr_netbox` (Compte de service restreint DML pour l'application Netbox)
    * `role_adm_opsbricks` (Groupe propriétaire DDL pour le projet OpsBricks)
* ❌ **Non conforme** : 
    * `authentik` (Invalide : Absence du segment de typologie `db_`, `usr_` ou `role_adm_`)
    * `db-netbox` (Invalide : Utilisation d'un tiret au lieu de l'underscore)
    * `Usr_Authentik` (Invalide : Présence de majuscules et non-respect de la casse)

## 4. Exceptions
La base de données système `postgres` et les rôles natifs générés par le moteur lors de l'installation (ex: `postgres`, `pg_monitor`) sont exemptés de cette convention et ne doivent pas être altérés ou supprimés.

## 5. Validation

Le respect de cette convention est assuré par l'automatisation :

* Les revues de code (Pull Requests) sur le dépôt `infrastructure-ansible` vérifient la syntaxe des variables déclarées dans le fichier `group_vars/all.yml`.
* Le linter Ansible (ansible-lint) intégré dans la pipeline d'intégration continue (CI) rejette tout déploiement contenant des noms de ressources mal formatés ou ne respectant pas l'expression régulière définie.