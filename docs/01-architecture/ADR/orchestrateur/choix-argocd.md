---
title: Choix de ArgoCD
date: 2026-08-18
status: accepted
author: Louis MEDO
owner: Louis MEDO
tags: [adr, architecture, gitops, kubernetes]
---

# ADR : {{ page.meta.title }}

![Bannière LoutikCloud](https://raw.githubusercontent.com/loutik/design-assets/main/loutikcloud/banniere_loutikcloud.png)

!!! info "Informations"
    * **Date de création** : {{ page.meta.date }}
    * **Statut** : {{ page.meta.status }}
    * **Auteur** : {{ page.meta.author }}
    * **Responsable** : {{ page.meta.owner }}

## 1. Contexte
L'infrastructure s'appuie sur un cluster Kubernetes hautement disponible, composé de trois nœuds "control plane" et de deux nœuds "worker". Pour rationaliser les opérations, il est nécessaire d'adopter une approche GitOps permettant d'automatiser le cycle de vie des applications. Le besoin métier est de s'assurer que les manifests Kubernetes stockés dans un dépôt Git soient la seule source de vérité et soient automatiquement synchronisés sur le cluster.

Au-delà de l'automatisation technique, il est impératif de disposer d'une solution simple à maintenir, offrant un tableau de bord visuel pour identifier rapidement l'état des ressources déployées. L'outil retenu doit également être un standard de l'industrie afin de servir de plateforme de formation professionnelle, tout en s'intégrant dans la stack d'observabilité de l'infrastructure (exposition de métriques pour Prometheus ou Zabbix).

## 2. Cahiers des charges

| ID | Type | Exigence | Description |
| :--- | :--- | :--- | :--- |
| **[REQ-F01]** | Fonctionnel | Automatisation GitOps | L'outil doit appliquer automatiquement les manifests Kubernetes stockés dans un dépôt Git vers le cluster. |
| **[REQ-F02]** | Fonctionnel | Interface utilisateur (Dashboard) | L'outil doit fournir une interface web native et intuitive pour visualiser l'état de synchronisation et de santé des ressources. |
| **[REQ-F03]** | Fonctionnel | Standard entreprise | La solution doit être largement adoptée en entreprise pour garantir une montée en compétence valorisable. |
| **[REQ-T01]** | Technique | Simplicité de maintenance | L'architecture de la solution doit être simple à déployer, à configurer et à maintenir au quotidien. |
| **[REQ-T02]** | Technique | Observabilité | La solution doit exposer des métriques de performance et d'état récupérables par Prometheus ou Zabbix. |

## 3. Les solutions du marché

### 3.1. Présentations des solutions

#### 3.1.1. ArgoCD
* **Présentation générale :** Outil open-source de livraison continue (CD) déclaratif et orienté GitOps spécifiquement conçu pour Kubernetes.
* **Fonctionnement :** Utilise un contrôleur Kubernetes qui surveille en permanence les applications en cours d'exécution et compare l'état réel avec l'état désiré spécifié dans le dépôt Git (pull-based).
* **Profil :** Standard de l'industrie, ciblant les équipes d'ingénierie DevOps et SRE. Réputé pour son interface web très aboutie et sa simplicité d'intégration.

#### 3.1.2. FluxCD
* **Présentation générale :** Solution open-source GitOps pour Kubernetes, développée par Weaveworks et incubée par la CNCF.
* **Fonctionnement :** Architecture modulaire basée sur des "GitOps Toolkit components" (source-controller, kustomize-controller, etc.) qui réconcilient l'état du cluster (pull-based).
* **Profil :** Très orienté automatisation CLI et infrastructure as code pure. Conçu pour des intégrations techniques poussées en entreprise.

#### 3.1.3. GitLab Agent pour Kubernetes
* **Présentation générale :** Composant intégré à l'écosystème GitLab permettant de connecter un cluster Kubernetes aux pipelines CI/CD de GitLab.
* **Fonctionnement :** Un agent tourne dans le cluster et tire (pull) les configurations directement depuis les projets GitLab associés.
* **Profil :** Orienté développeurs et équipes utilisant exclusivement GitLab comme forge logicielle et registre central.

### 3.2. Comparatifs des solutions

| Exigence | ArgoCD | FluxCD | GitLab Agent |
| :--- | :--- | :--- | :--- |
| **[REQ-F01 (GitOps)]** | Validé | Validé | Validé |
| **[REQ-F02 (Dashboard)]** | Validé (UI native et complète) | Non validé (UI nécessitant des addons externes) | Partiel (Intégré à l'UI GitLab, moins spécifique au cluster) |
| **[REQ-F03 (Standard Entreprise)]** | Validé (Très forte demande) | Validé | Évaluation (Dépend de la stack de l'entreprise) |
| **[REQ-T01 (Simplicité)]** | Validé | Évaluation (Courbe d'apprentissage liée à l'architecture modulaire) | Évaluation (Couplage fort avec GitLab) |
| **[REQ-T02 (Observabilité)]** | Validé (Métriques Prometheus natives) | Validé (Métriques Prometheus natives) | Évaluation |

## 4. Solution proposée

La solution proposée pour l'infrastructure est **ArgoCD**.

L'intégration d'ArgoCD au sein du cluster Kubernetes (3 control planes, 2 workers) répond parfaitement aux exigences de simplicité, d'automatisation et de visibilité. ArgoCD sera déployé en tant que contrôleur dans un namespace dédié. Il se connectera de manière sécurisée aux dépôts Git contenant les manifests (YAML, Kustomize ou Helm) et assurera la boucle de réconciliation automatique. 

L'interface web native d'ArgoCD offre un atout décisif pour visualiser en temps réel l'arbre des ressources déployées, facilitant grandement le débogage et la formation aux concepts Kubernetes. De plus, ArgoCD expose nativement un endpoint `/metrics` au format Prometheus, ce qui permet de configurer des alertes (ex: désynchronisation d'une application, état dégradé) et des tableaux de bord directement via la stack de supervision de l'infrastructure.

**Justification du rejet des solutions alternatives :**

* **FluxCD :** Bien que FluxCD soit un excellent moteur GitOps (déjà expérimenté dans d'autres contextes d'automatisation), il est écarté pour ce besoin spécifique en raison de l'absence d'un tableau de bord natif et intuitif. La nécessité de déployer des composants supplémentaires (comme Weave GitOps) pour obtenir une interface graphique complexifie la maintenance par rapport à l'approche tout-en-un d'ArgoCD.
* **GitLab Agent pour Kubernetes :** Cette solution a été rejetée car elle crée un couplage fort avec l'écosystème GitLab. L'objectif est de conserver une indépendance de la forge logicielle pour les déploiements (capacité à utiliser n'importe quel dépôt Git) et de bénéficier d'une interface centrée exclusivement sur l'état du cluster Kubernetes, ce qu'ArgoCD fait de manière beaucoup plus spécialisée.