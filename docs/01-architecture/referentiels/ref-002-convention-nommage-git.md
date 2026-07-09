---
id: 002
title: Convention de nommage Git
category: Code
date: 2026-05-17
author: Louis MEDO
tags: [convention, git, ci-cd]
---

# REF-{{ page.meta.id }} : {{ page.meta.title }}

![Bannière LoutikCloud](https://raw.githubusercontent.com/loutik/design-assets/main/loutikcloud/banniere_loutikcloud.png)

!!! info "Informations du Référentiel"
    * **Date d'application** : {{ page.meta.date }}
    * **Catégorie** : {{ page.meta.category }}
    * **Garant de la norme** : {{ page.meta.author }}

## 1. Objectif et périmètre
Ce référentiel a pour objectif de standardiser le cycle de vie du code source (nommage des dépôts, arborescence des branches, messages de commits et Pull Requests). Cette rigueur est indispensable pour générer des changelogs automatiques, faciliter le débogage et déclencher des pipelines CI/CD fiables.

Le périmètre s'applique à l'intégralité des dépôts de l'écosystème LoutikCLOUD, incluant l'Infrastructure as Code (Ansible, Terraform), la documentation (MkDocs), et les projets de développement logiciel.

## 2. Conventions

### 2.1. Nommage des Dépôts (Repositories)

* **Format strict** : `kebab-case` (minuscules, mots séparés par des tirets).
* **Structure** : `[domaine]-[projet]` ou `[projet]-[composant]`.

### 2.2. Stratégie de Branches (Branching)

* **Branche principale** : `main` (environnement de production / source de vérité).
* **Format des branches de travail** : `[type]/[issue-id]-[courte-description]`.
* **Types autorisés** : `feat` (nouvelle fonctionnalité), `fix` (correction de bug), `docs` (documentation), `chore` (tâche de maintenance).

### 2.3. Messages de Commit (Conventional Commits)

Le projet adopte la norme industrielle *Conventional Commits*.

* **Format** : `<type>(<scope>): <description courte>`
* **`type`** : `feat` (ajout/modification de code), `fix` (patch), `docs` (documentation), `style` (formatage, CSS), `refactor` (restructuration sans changement de comportement), `ci` (pipelines), `test` (tests).
* **`scope`** (optionnel) : Le module impacté, toujours en minuscules.
* **`description`** : Impératif, présent, sans majuscule au début, ni point final.

## 3. Exemples pratiques

### Nommage des Dépôts

* ✅ **Conforme** : 
    * `infrastructure-ansible` (Dépôt contenant les rôles et playbooks de l'infrastructure)
    * `bot-mathik` (Code source applicatif)
    * `infrastructure-docs` (Site de documentation MkDocs)

* ❌ **Non conforme** : 
    * `Bot_Mathik` (Utilisation de majuscules et d'underscores)
    * `Ansible Portfolio` (Espace dans le nom du dépôt)

### Nommage des Branches

* ✅ **Conforme** : 
    * `feat/12-add-sso-authentik`
    * `fix/45-typo-readme-k3s`

* ❌ **Non conforme** : 
    * `louis-dev-test` (Absence de type et description vague)
    * `patch-1` (Nommage généré par défaut ne décrivant pas l'action)

### Messages de Commit

* ✅ **Conforme** : 

    * `feat(ansible): ajout du rôle de déploiement apache`
    * `fix(network): correction de la route par défaut vers la gateway`
    * `docs: mise à jour du schéma d'architecture`

* ❌ **Non conforme** : 

    * `Mise à jour du script` (Absence de type de commit)
    * `fix: résolution du bug` (Description inutilement générique)
    * `feat(WAF): Ajout de Crowdsec.` (Scope en majuscule et point final)

## 4. Exceptions

* **Dépôts forkés** : Les projets *upstream* forkés (clonés depuis une source externe pour contribution) conservent leur nom d'origine.
* **Branches automatisées** : Les branches générées automatiquement par des outils de maintenance des dépendances (ex: Dependabot, Renovate) dérogent à la règle du format (ex: `dependabot/npm_and_yarn/mkdocs-material-9.5`).

## 5. Validation

* **Pre-commit Hooks** : Utilisation de l'outil `commitlint` en local pour empêcher la création d'un commit ne respectant pas le formalisme *Conventional Commits*.
* **CI/CD** : Un workflow GitHub Actions / Jenkins rejette systématiquement les Pull Requests dont le titre ou les commits ne sont pas conformes, bloquant ainsi la fusion vers la branche `main`.