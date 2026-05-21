ROLE : Administrateur système et SRE senior. Ton approche doit être rigoureuse, standardisée et respecter les principes d'ingénierie pour la définition des conventions d'infrastructure.

MISSION : Rédiger un document de Référentiel (normes, conventions de nommage, standards) basé sur les paramètres ci-dessous.

PARAMETRES :
- ID : [Ex: REF-001]
- Titre : [Ex: Plan de nommage des machines virtuelles]
- Catégorie : [Ex: Nommage | Sécurité | Architecture | Code]
- Date d'application : [YYYY-MM-DD]
- Auteur (Garant) : [Ex: Louis MEDO]
- Objectif et périmètre : [Pourquoi cette norme existe et à quelles ressources elle s'applique]
- Règles : [La définition exacte de la convention (ex: syntaxe, regex, format)]
- Exemples : [Cas valides (Do) et invalides (Don't)]
- Exceptions : [Les dérogations autorisées]
- Outillage / Validation : [Comment cette norme est imposée techniquement (ex: linter, CI/CD)]

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
category: [Catégorie]
date: [Date d'application]
author: [Auteur]
tags: [referentiel, norme, standard, convention]
---

# {{ page.meta.title }}

![Bannière LoutikCLOUD](/assets/banniere_loutikcloud.png)

!!! info "Informations du Référentiel"
    * **Date d'application** : {{ page.meta.date }}
    * **Catégorie** : {{ page.meta.category }}
    * **Garant de la norme** : {{ page.meta.author }}

## 1. Objectif et périmètre
[Définition claire de l'utilité de cette norme (ex: harmoniser et faciliter l'identification des ressources) et de son périmètre d'application précis (ex: s'applique uniquement aux nœuds Proxmox, pas aux conteneurs Docker).]

## 2. Conventions
[Exposé détaillé et sans ambiguïté de la standardisation. Utiliser des listes, des tableaux de correspondances ou des expressions régulières (Regex) si nécessaire.]

* **Règle fondamentale** : [Ex: Le nommage doit suivre le format `[env]-[role]-[index]`.]
* **Composant 1 (`env`)** : [Ex: `prd` (Production), `stg` (Staging), `dev` (Développement).]
* **Composant 2 (`role`)** : [Ex: `web` (Serveur HTTP), `db` (Base de données), `k8s` (Nœud Kubernetes).]
* **Composant 3 (`index`)** : [Ex: Numérotation séquentielle sur deux chiffres commençant à `01`.]

## 3. Exemples pratiques
[Illustration par des cas concrets pour faciliter la compréhension immédiate.]

* ✅ **Conforme** : 
    * `prd-k8s-01` (Nœud Kubernetes en production)
    * `dev-db-02` (Deuxième serveur de base de données en développement)
* ❌ **Non conforme** : 
    * `PRD_K8S_1` (Majuscules, underscores au lieu de tirets, manque le zéro initial)
    * `serveur-web-test` (Ne respecte pas le format des variables)

## 4. Exceptions
[Cas particuliers reconnus où cette norme peut être légitimement contournée (ex: contraintes d'un logiciel tiers imposant un nom spécifique, ressources héritées (legacy)).]

## 5. Validation
[Moyen technique mis en œuvre pour garantir le respect de cette norme au quotidien.]
* [Ex: Validation automatisée par un linter (ex: `ansible-lint`, `yamllint`) dans le pipeline CI/CD de GitHub Actions.]
* [Ex: Contrôle bloquant via une politique OPA (Open Policy Agent) sur le cluster Kubernetes.]
``````