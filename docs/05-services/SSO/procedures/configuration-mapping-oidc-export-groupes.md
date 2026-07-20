---
title: Configuration du mapping OIDC pour l'export des groupes
service: Authentik
date: 2026-07-20
author: Louis MEDO
owner: Louis MEDO
---

# {{ page.meta.title }}

![Bannière LoutikCloud](https://raw.githubusercontent.com/loutik/design-assets/main/loutikcloud/banniere_loutikcloud.png)

!!! info "Informations"

    * **Date de mise à jour** : {{ page.meta.date }}
    * **Service ciblé** : {{ page.meta.service }}
    * **Auteur** : {{ page.meta.author }}
    * **Responsable** : {{ page.meta.owner }}

## 1. Contexte

Cette procédure détaille la procédure opérationnelle permettant de configurer un mappage de portée (Scope Mapping) personnalisé et de l'associer à un fournisseur OIDC (Provider). L'objectif est de s'assurer que les groupes auxquels appartient un utilisateur dans Authentik soient correctement injectés et transmis dynamiquement au service cible (ex: Proxmox) via le jeton d'authentification.

## 2. Prérequis

* Accès administrateur (Superuser) actif sur la console d'administration Web Authentik.
* Fournisseur OIDC (Provider) de l'application cible préalablement créé.

## 3. Procédure d'exécution

1. **Création de la Customization (Scope Mapping).** Naviguer dans `Customization` > `Property Mappings` > `Create` > `Scope Mapping`. Nommer le mappage `authentik default OAuth Mapping: OpenID 'groups'` et définir le nom de la portée à `groups`. Insérer le code suivant dans le champ d'expression.

  ```python
  return {
      "groups": [group.name for group in request.user.ak_groups.all()]
  }
  ```

  - **`request.user.ak_groups.all()`** : Interroge la base de données interne d'Authentik pour récupérer la liste de tous les objets groupes associés à l'utilisateur lors de sa demande de jeton.

2. **Configuration du Fournisseur OIDC (Provider).** Naviguer dans `Applications` > `Providers` et éditer le Provider OIDC cible. Dérouler la section **Advanced protocol settings** pour ajouter la portée au jeton.

  ```text
  Sélectionner : authentik default OAuth Mapping: OpenID 'groups'
  ```

  - **`Scopes sélectionnés`** : Autorise le fournisseur à injecter de manière dynamique le claim JSON `"groups"` dans le token OIDC transmis à l'application.

## 4. Validation

1. Naviguer dans **Customization > Property Mappings** et sélectionner le mappage `authentik default OAuth Mapping: OpenID 'groups'`.
2. Cliquer sur le bouton **Test**, sélectionner un utilisateur membre d'un groupe et vérifier que la clé `"groups"` est bien présente et renseignée dans le résultat JSON.
3. Effectuer une connexion de test sur l'application cible et vérifier dans ses paramètres internes (ou logs) que les groupes ont bien été reçus via le jeton OIDC.