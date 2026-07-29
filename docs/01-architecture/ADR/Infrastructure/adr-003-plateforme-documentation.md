---
id: 003
title: Choix de la plateforme pour la documentation
date: 2026-05-17
status: accepted
author: Louis MEDO
owner: Louis MEDO
tags: [adr, architecture]
---

# ADR-{{ page.meta.id }} : {{ page.meta.title }}

![Bannière LoutikCloud](https://raw.githubusercontent.com/loutik/design-assets/main/loutikcloud/banniere_loutikcloud.png)

!!! info "Informations"
    * **Date de création** : {{ page.meta.date }}
    * **Statut** : {{ page.meta.status }}
    * **Auteur** : {{ page.meta.author }}
    * **Responsable** : {{ page.meta.owner }}

## 1. Contexte
Notre plateforme de documentation actuelle, basée sur Docusaurus, présente des frictions opérationnelles. La maintenance quotidienne est contraignante et impose aux équipes de cloner un dépôt volumineux. Par ailleurs, la complexité de l'arborescence des fichiers ralentit l'intégration et la contribution des collaborateurs débutants.

## 2. Décision
Nous avons pris la décision de migrer vers **MkDocs**. Cette solution a été retenue pour sa légèreté, sa sobriété et son faible coût de fonctionnement par rapport à Docusaurus. MkDocs propose une structure de fichiers nativement plus simple, éliminant la surcharge technique liée aux écosystèmes plus complexes.

## 3. Conséquences
Analyse comparative par rapport à nos besoins opérationnels :

| Besoin / Critère | Docusaurus | MkDocs |
| :--- | :--- | :--- |
| **Poids du dépôt** | Lourd | Léger |
| **Gestion de l'arborescence** | Complexe pour les débutants | Simple et intuitive |
| **Maintenance quotidienne** | Contraignante | Allégée |
| **Coût d'exploitation** | Élevé | Moindre coût |

* **Impacts positifs** :
    * Les équipes n'ont plus à télécharger un lourd dépôt pour contribuer.
    * L'arborescence simplifiée rend la courbe d'apprentissage beaucoup plus rapide pour les débutants.
    * Alignement facilité pour structurer notre contenu selon le framework documentaire Diátaxis.
* **Impacts négatifs (dette technique / risques)** :
    * Effort initial nécessaire pour la migration du contenu existant et l'adaptation du pipeline de déploiement continu.
    * Perte des composants interactifs (React) natifs à Docusaurus.