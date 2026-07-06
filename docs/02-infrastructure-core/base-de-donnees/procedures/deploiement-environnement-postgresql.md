---
title: Déploiement de l'environnement PostgreSQL
service: Base de données / PostgreSQL
date: 2026-07-06
author: Louis MEDO
owner: Louis MEDO
tags: [deploiement, iac, mise-en-production]
---

# {{ page.meta.title }}

![Bannière LoutikCLOUD](/assets/banniere_loutikcloud.png)

!!! info "Informations"
    * **Date de création** : {{ page.meta.date }}
    * **Service ciblé** : {{ page.meta.service }}
    * **Auteur** : {{ page.meta.author }}
    * **Responsable** : {{ page.meta.owner }}

## 1. Architecture et contexte
Le service de base de données PostgreSQL est le socle de données de l'infrastructure LoutikCLOUD. Son provisionnement est entièrement géré en Infrastructure as Code (IaC) via Ansible. Le serveur cible de production est `mlt1-db-vm-prd-01`, déployé sur l'adresse IP `10.0.17.1`. L'architecture respecte les principes Zero Trust avec une séparation des privilèges : les propriétaires de base gèrent la structure (DDL) tandis que les comptes applicatifs limités manipulent la donnée (DML). Les configurations systèmes, incluant le passage par le proxy de la DMZ (`debian-proxy`), sont orchestrées en amont du provisionnement du moteur.

## 2. Prérequis

* Serveur virtuel cible `mlt1-db-vm-prd-01` instancié et accessible sur le réseau de production.
* Utilisateur système `svc-ansible` configuré sur le serveur cible avec élévation de privilèges.
* Clé SSH privée `~/.ssh/id_ed25519_svc-ansible` disponible sur le poste de travail de l'administrateur. (⚠️ Ceci est temporaire le temps que le bastion soit mis en place.)
* Connaissance du mot de passe maître permettant de déchiffrer le fichier `secrets/vault.yml`.

## 3. Préparation de l'environnement de travail local

3.1. **Installation d'Ansible**. Installation du moteur d'automatisation sur la machine locale du collaborateur en fonction de son système d'exploitation.

  * **Linux** : 
  ```bash
  sudo apt update && sudo apt install ansible
  ```

  * **MacOS** : 
  ```bash
  brew install ansible
  ```

  * **Windows** : Ansible n'étant pas nativement supporté, il est impératif d'installer WSL2 (Windows Subsystem for Linux), d'y déployer une distribution (ex: Ubuntu), puis d'exécuter la commande d'installation Linux ci-dessus.

3.2. **Clonage du dépôt d'infrastructure**. Récupération du référentiel Git contenant l'arborescence des inventaires, rôles et playbooks.

  ```bash
  git clone https://github.com/loutik/infrastructure-ansible.git
  cd infrastructure-ansible
  ```

3.3. **Installation des dépendances**. Ajout de la collection communautaire PostgreSQL, requise pour permettre à Ansible d'interagir avec le moteur de base de données.

  ```bash
  ansible-galaxy collection install community.postgresql
  ```

## 4. Configuration des variables et gestion des secrets

4.1. **Déclaration des bases et du modèle RBAC**. Les nouvelles bases de données (ex: `db_authentik`, `db_netbox`), les groupes d'administration associés (`role_adm_authentik`), et l'affectation des utilisateurs humains (ex: `louismedo`) s'effectuent dans le fichier de variables d'environnement en clair.

  ```bash
  nano inventories/production/group_vars/all.yml
  ```

4.2. **Mise à jour des mots de passe dans le Vault**. Les secrets de production des comptes applicatifs et nominatifs doivent être injectés dans le coffre-fort chiffré Ansible Vault.

  ```bash
  ansible-vault edit secrets/vault.yml
  ```

## 5. Déploiement et exécution des Playbooks

La logique d'Ansible repose sur des "Playbooks" (les recettes globales) qui exécutent des "Rôles" (les listes de tâches spécifiques). 

5.1. **Initialisation complète du serveur**. Ce playbook global exécute successivement l'application du proxy système (`debian-proxy`), les paramètres de base OS (`debian-common`), l'installation du moteur PostgreSQL (`postgresql-install`), et la configuration RBAC des utilisateurs (`postgresql-config`). À utiliser **uniquement** lors de la création d'une nouvelle machine.

  ```bash
  ansible-playbook -i inventories/production/hosts.yml playbooks/postgresql-bootstrap.yml --ask-vault-pass
  ```

  `-i` : Cible le fichier d'inventaire définissant les serveurs de production.

  `--ask-vault-pass` : Invite l'administrateur à saisir le mot de passe pour déchiffrer les secrets.

5.2. **Mise à jour de la configuration de base de données**. Ce playbook restreint permet d'appliquer les modifications déclaratives (ajout/suppression de bases ou d'utilisateurs) en exécutant uniquement le rôle `postgresql-config`. À exécuter lors des opérations de maintien en condition opérationnelle (MCO).

  ```bash
  ansible-playbook -i inventories/production/hosts.yml playbooks/postgresql-config.yml --ask-vault-pass
  ```

## 6. Validation post-déploiement

6.1. **Validation de l'isolation Zero Trust**. Connexion au moteur de base de données via un client lourd (pgAdmin) ou en CLI (`psql`) pour confirmer la révocation des privilèges par défaut (`PUBLIC`).

6.2. **Validation RBAC**. Vérification de la création des comptes administrateurs avec le flag `NOLOGIN`, des bases affectées à ces propriétaires exclusifs, et des comptes de services disposant du privilège de connexion `CONNECT`.