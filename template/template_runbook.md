ROLE : Administrateur système et SRE senior. Ton approche doit être rigoureuse, précise et respecter les standards d'ingénierie pour les procédures d'exploitation.

MISSION : Rédiger un Runbook (procédure opérationnelle) basé sur les paramètres ci-dessous.

PARAMETRES :
- ID : [Ex: RB-001]
- Titre : [Ex: Mise à jour du cluster K3s]
- Service : [Ex: Orchestrateur / K3s]
- Date : [YYYY-MM-DD]
- Auteur : [Ex: Louis MEDO]
- Responsable : [Ex: Louis MEDO]
- Contexte : [Objectif de la procédure et conditions de déclenchement]
- Prérequis : [Droits, accès, outils nécessaires]
- Étapes : [Actions séquentielles avec commandes]
- Validation : [Critères de succès]
- Rollback : [Procédure de retour arrière en cas d'échec]

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
tags: [runbook, exploitation, mco]
---

# [ID] : [Titre]

![Bannière LoutikCLOUD](/assets/banniere_loutikcloud.png)

!!! info "Informations"

    * **Date de mise à jour** : {{ page.meta.date }}
    * **Service ciblé** : {{ page.meta.service }}
    * **Auteur** : {{ page.meta.author }}
    * **Responsable** : {{ page.meta.owner }}

## 1. Contexte et Objectif

[Description précise de l'objectif de ce runbook. Préciser dans quel cas de figure cette procédure doit être exécutée (ex: maintenance planifiée, réponse à une alerte spécifique).]

## 2. Prérequis

* [Accès ou rôle nécessaire, ex: Accès SSH avec privilèges sudo, permissions cluster-admin]
* [Outils et dépendances requis, ex: binaire `kubectl` configuré, `ansible-playbook`]

## 3. Procédure d'exécution
[Étapes séquentielles, claires et sans ambiguïté. Les commandes doivent être prêtes à être copiées-collées.]

1. **[Nom de l'étape 1]** :

    [Explication de l'action]

    ```bash
    [Commande exacte à exécuter]
    ```

    - **`[commande]`** : [Description]


2. **[Nom de l'étape 2]** :

    [Explication de l'action]

    ```bash
    [Commande exacte à exécuter]
    ```

## 4. Validation (Critères de succès)

[Instructions pour vérifier que la procédure a fonctionné et que le service est dans l'état attendu (ex: requêtes HTTP de test, vérification de l'état des pods, lecture des logs).]

## 5. Rollback (Retour arrière)

[Procédure exacte et commandes pour annuler les changements et restaurer l'état initial en cas d'échec critique lors de l'exécution.]

``````