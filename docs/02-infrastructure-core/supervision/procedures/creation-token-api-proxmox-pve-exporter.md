---
title: Création du Token API Proxmox pour PVE Exporter
service: Proxmox VE / Prometheus
date: 2026-08-23
author: Louis MEDO
owner: Louis MEDO
---

# {{ page.meta.title }}

![Bannière LoutikCloud](https://raw.githubusercontent.com/loutik/design-assets/main/loutikcloud/banniere_loutikcloud.png)

!!! info "Informations"
    * **Date de création** : {{ page.meta.date }}
    * **Service ciblé** : {{ page.meta.service }}
    * **Auteur** : {{ page.meta.author }}
    * **Responsable** : {{ page.meta.owner }}

## 1. Contexte
Déploiement d'un jeton d'authentification API (Token API) sur le cluster Proxmox VE. Ce jeton est destiné au composant `prometheus-pve-exporter` de la stack de supervision (NOC). Il permet à l'exportateur de s'authentifier auprès de l'API Proxmox pour récupérer l'état de l'hyperviseur, du cluster, et la consommation de ressources des machines virtuelles, tout en respectant le principe de moindre privilège via l'attribution exclusive du rôle natif de lecture seule (`PVEAuditor`).

## 2. Prérequis

* Accès administrateur (`root@pam`) à l'interface d'administration web d'un nœud du cluster Proxmox VE.
* Stack Prometheus opérationnelle.
* L'outil d'automatisation Ansible configuré avec un accès au Vault pour la gestion du secret du jeton.

## 3. Configuration via l'interface web Proxmox VE

3.1.  **Création de l'utilisateur.** Dans le menu de gauche, naviguer vers `Datacenter` > `Permissions` > `Users`. Cliquer sur `Add`.

* `User name` : `svc_prometheus`
* `Realm` : `Proxmox VE authentication server (pve)`
* Laisser le champ `Password` vide (l'authentification se fera par token).

3.2.  **Création du Jeton API (API Token).** Dans le menu de gauche, naviguer vers `Datacenter` > `Permissions` > `API Tokens`. Cliquer sur `Add`.

* `User` : Sélectionner l'utilisateur `svc_prometheus@pve` créé précédemment.
* `Token ID` : `prometheus`
* Décocher la case `Privilege Separation` (le jeton héritera strictement des droits attribués ci-dessous).
* Copier immédiatement la valeur du `Secret` affichée à l'écran. Cette valeur ne sera plus jamais visible. Stocker cette valeur de manière chiffrée dans `ansible-vault` sous la variable `vault_prometheus_pve_exporter_token_value`.

3.3.  **Attribution des permissions.** Dans le menu de gauche, naviguer vers `Datacenter` > `Permissions`. Cliquer sur `Add` > `API Token Permission`.

* `Path` : `/` (Permet la lecture sur l'ensemble du cluster).
* `API Token` : Sélectionner le jeton `monitoring@pve!prometheus`.
* `Role` : Sélectionner `PVEAuditor`.