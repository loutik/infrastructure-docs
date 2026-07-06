---
title: Provisionnement d'une nouvelle base de données applicative
service: Base de données / PostgreSQL
date: 2026-07-06
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

L'objectif de cette procédure est d'automatiser et de standardiser la création d'une nouvelle base de données isolée au sein du cluster de production LoutikCLOUD. Ce runbook doit être exécuté lors du déploiement d'un nouveau service nécessitant de la persistance (ex: Authentik, Netbox) afin de lui fournir une base de données avec séparation stricte des privilèges (DDL vs DML) respectant l'architecture Zero Trust.

## 2. Prérequis

* Accès au dépôt Git `infrastructure-ansible` à jour (branche `main`).
* Binaire `ansible-playbook` fonctionnel avec la collection `community.postgresql` installée. 
* Clé SSH du compte de service `svc-ansible` chargée dans l'agent SSH local de l'opérateur. (⚠️ Ceci est temporaire le temps que le bastion soit mis en place.)
* Connaissance du mot de passe maître pour le déchiffrement du coffre Ansible (`secrets/vault.yml`).

## 3. Procédure d'exécution

1. **Déclaration de la nomenclature (Architecture RBAC).** Éditer le fichier d'inventaire global en clair pour ajouter le nouveau service dans la liste `pg_apps` en respectant la convention de nommage (`db_[service]`, `role_adm_[service]`, `usr_[service]`). Assigner ensuite le groupe d'administration à un utilisateur humain (DBA) dans la liste `pg_users_roles`.

  ```bash
  nano inventories/production/group_vars/all.yml
  ```

  **`nano`** : Éditeur de texte en ligne de commande pour modifier l'inventaire des variables d'environnement.

2. **Stockage sécurisé des identifiants applicatifs.** Ouvrir le coffre-fort chiffré pour y ajouter le mot de passe du compte de service (DML) qui vient d'être déclaré dans l'étape précédente.

  ```bash
  ansible-vault edit secrets/vault.yml
  ```

  **`ansible-vault edit`** : Commande Ansible permettant d'éditer un fichier chiffré à la volée de manière sécurisée.

3. **Déploiement de la configuration via IaC.** Exécuter le playbook de configuration PostgreSQL pour appliquer les modifications déclarées. Ce playbook est idempotent et ne modifiera que les ressources nouvellement ajoutées.

  ```bash
  ansible-playbook -i inventories/production/hosts.yml playbooks/postgresql-config.yml --ask-vault-pass
  ```

  **`-i inventories/production/hosts.yml`** : Cible explicitement le périmètre de production pour l'exécution du playbook.

  **`--ask-vault-pass`** : Demande interactivement le mot de passe maître pour déchiffrer les identifiants en mémoire.

## 4. Validation

* Le retour de la commande `ansible-playbook` n'affiche aucune erreur (`failed=0`) et affiche le statut `changed` pour les nouvelles ressources.
* **Validation de la donnée (DML)** : L'application tierce (via K3s) ou une connexion manuelle avec le compte `usr_[service]` réussit, mais toute tentative de modification de structure (ex: `DROP TABLE`) renvoie une erreur de permission.
* **Validation de la structure (DDL)** : La connexion avec le compte d'administration nominatif (ex: `louismedo`) permet de s'attribuer le rôle de propriétaire (`SET ROLE role_adm_[service];`) et de créer une table de test.

## 5. Rollback (Retour arrière)

Afin d'éviter tout dommage accidentel sur l'infrastructure de production (Blast Radius), le code Ansible ne gère pas la destruction automatisée des bases de données. Le retrait s'effectue manuellement.

1. Supprimer l'application et les affectations de privilèges dans les fichiers `group_vars/all.yml` et `secrets/vault.yml`, puis valider le commit.

2. Se connecter en CLI sur le serveur de base de données en tant qu'administrateur système pour procéder au nettoyage manuel.

```bash
sudo -u postgres psql
```

```sql
DROP DATABASE db_[service];
DROP ROLE usr_[service];
DROP ROLE role_adm_[service];
```