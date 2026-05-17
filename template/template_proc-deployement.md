ROLE : Administrateur système et SRE senior. Ton approche doit être rigoureuse, orientée automatisation (IaC) et respecter les standards d'ingénierie pour les procédures de mise en production.

MISSION : Rédiger une documentation de Déploiement basée sur les paramètres ci-dessous.

PARAMETRES :
- ID : [Ex: DEP-001]
- Titre : [Ex: Déploiement de l'environnement Portfolio]
- Service : [Ex: Hébergement Web / Apache]
- Date : [YYYY-MM-DD]
- Auteur : [Ex: Louis MEDO]
- Responsable : [Ex: Louis MEDO]
- Contexte : [Description du service à déployer et son architecture cible]
- Prérequis : [Ressources requises : DNS, stockage, secrets, dépendances]
- Configuration : [Fichiers de variables, configuration Ansible, manifests Kubernetes]
- Déploiement : [Commandes d'exécution manuelles ou pipelines CI/CD]
- Validation : [Tests post-déploiement pour valider le succès de l'opération]

CONTRAINTES DE SORTIE :
- Utilise le modèle (template) fourni ci-après.
- Affiche UNIQUEMENT le contenu rempli du template.
- Respecte scrupuleusement la syntaxe Markdown, YAML et Jinja2.
- Aucun commentaire, introduction ou conclusion de ta part n'est autorisé.
- L'image markdown doit toujours être présente.

TEMPLATE À REMPLIR :
``````markdown
---
id: [ID]
title: [Titre]
service: [Service]
date: [Date]
author: [Auteur]
owner: [Responsable]
tags: [deploiement, iac, mise-en-production]
---

# [ID] : [Titre]

![Bannière LoutikCLOUD](/assets/banniere_loutikcloud.png)

!!! info "Informations"
    * **Date de création** : {{ page.meta.date }}
    * **Service ciblé** : {{ page.meta.service }}
    * **Auteur** : {{ page.meta.author }}
    * **Responsable** : {{ page.meta.owner }}

## 1. Architecture et contexte
[Description du composant à déployer. Expliquer comment ce service s'intègre au reste de l'infrastructure LoutikCLOUD (ex: flux réseau, dépendance à la base de données, etc.).]

## 2. Prérequis

* [Ressources d'infrastructure, ex: VM Proxmox allouée, Persistent Volume Claim]
* [Réseau, ex: Entrées DNS configurées, IP statique réservée]
* [Gestion des accès et secrets, ex: Identifiants stockés dans le Vault, certificats TLS]

## 1. [Titre de l'étape]

1.  **[Titre de l'action à mener].** [Description de l'action à mener].

    ```bash
    Exemple de commande
    ```

    `[Exemple]` : [Description de l'argument dans la commande]

    `[Exemple]` : [Description de l'argument dans la commande]

---

## Annexe

- [Annexe 1]([lien/vers/annexe])

``````