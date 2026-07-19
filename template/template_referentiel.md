ROLE : Administrateur système et SRE senior. Ton approche doit être rigoureuse, standardisée et respecter les principes d'ingénierie pour la définition des conventions d'infrastructure.

MISSION : Rédiger un document de Référentiel (normes, conventions de nommage, standards) basé sur les paramètres ci-dessous.

PARAMETRES :
- ID : [Ex: REF-002]
- Titre : [Ex: Convention de nommage des VLANs / Buckets S3 / etc.]
- Catégorie : [Ex: Nommage | Sécurité | Architecture | Code]
- Date d'application : [YYYY-MM-DD]
- Auteur : [Ex: Louis MEDO]
- Objectif et périmètre : [Pourquoi cette norme existe et à quelles ressources elle s'applique précisément]
- Regex et Délimiteur : [L'expression régulière de validation et le séparateur utilisé (ex: tiret, underscore)]
- Segments : [La liste des variables composant le nom et leurs rôles respectifs]
- Exemples : [Cas valides (Do) et invalides (Don't) liés aux segments définis]
- Exceptions : [Les dérogations autorisées (legacy, contraintes cloud...)]
- Outillage / Validation : [Comment cette norme est imposée techniquement (ex: Terraform, OPA, CI/CD, linter)]

CONTRAINTES DE SORTIE :
- Utilise le modèle (template) fourni ci-après.
- Affiche UNIQUEMENT le contenu rempli du template.
- Respecte scrupuleusement la syntaxe Markdown, YAML et Jinja2.
- Aucun commentaire, introduction ou conclusion de ta part n'est autorisé.
- L'image markdown doit toujours être présente.

TEMPLATE À REMPLIR :
````markdown
---
id: [ID]
title: [Titre]
category: [Catégorie]
date: [Date d'application]
author: [Auteur]
tags: [referentiel, norme, standard, convention]
---

# REF-{{ page.meta.id }} : {{ page.meta.title }}

![Bannière LoutikCloud](https://raw.githubusercontent.com/loutik/design-assets/main/loutikcloud/banniere_loutikcloud.png)

!!! info "Informations"
    * **Date d'application** : {{ page.meta.date }}
    * **Catégorie** : {{ page.meta.category }}
    * **Garant de la norme** : {{ page.meta.author }}

## 1. Objectif
[Définition claire de l'utilité de cette norme et de son périmètre d'application exclusif.]

!!! warning "Limites du périmètre"
    [Indiquer ici si certaines ressources liées sont exclues et rediriger vers le référentiel adéquat, sinon supprimer le bloc.]

## 2. Conventions
La structure globale repose sur un format strict composé de [X] segments séparés par des [délimiteurs].

* **Format d'expression régulière (Regex)** : `[Regex de validation]`
* **Structure globale** : `[Schéma des segments, ex: <seg1>-<seg2>-<seg3>]`

| Segment | Rôle | Valeurs autorisées |
| :--- | :--- | :--- |
| **`[seg1]`** ([Nom complet]) | [Description de l'utilité du segment] | [Valeurs, trigrammes ou format attendu] |
| **`[seg2]`** ([Nom complet]) | [Description de l'utilité du segment] | [Valeurs, trigrammes ou format attendu] |
| **`[seg3]`** ([Nom complet]) | [Description de l'utilité du segment] | [Valeurs, trigrammes ou format attendu] |

!!! note "Contraintes spécifiques"
    [Ajouter ici toute contrainte globale : ex. interdiction des majuscules, limite de caractères imposée par l'outil.]

## 3. Exemples pratiques

* ✅ **Conforme** : 
    * `[Exemple 1 valide]` ([Explication du pourquoi selon les segments])
    * `[Exemple 2 valide]` ([Explication du pourquoi selon les segments])
* ❌ **Non conforme** : 
    * `[Exemple 1 invalide]` (Invalide : [Raison de l'échec, ex: majuscules, mauvais délimiteur])
    * `[Exemple 2 invalide]` (Invalide : [Raison de l'échec])

## 4. Exceptions
[Lister les cas particuliers justifiant le contournement de cette règle, par exemple des ressources héritées (legacy) ou des contraintes fournisseurs.]

## 5. Validation
Le respect de cette convention est assuré par l'automatisation :
* [Explication 1 de la validation technique (ex: block `validation` Terraform)]
* [Explication 2 de la validation technique (ex: policy Open Policy Agent, Linter CI/CD)]
````