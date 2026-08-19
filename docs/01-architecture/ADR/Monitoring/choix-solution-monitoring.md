---
title: Choix de la solution de supervision
date: 2026-08-19
status: accepted
author: Louis MEDO
owner: Louis MEDO
tags: [adr, architecture]
---

# ADR : {{ page.meta.title }}

![Bannière LoutikCloud](https://raw.githubusercontent.com/loutik/design-assets/main/loutikcloud/banniere_loutikcloud.png)

!!! info "Informations"
    * **Date de création** : {{ page.meta.date }}
    * **Statut** : {{ page.meta.status }}
    * **Auteur** : {{ page.meta.author }}
    * **Responsable** : {{ page.meta.owner }}

## 1. Contexte
L'infrastructure LoutikCLOUD repose sur un environnement hétérogène incluant un pare-feu OPNsense, des machines virtuelles Debian et un cluster Kubernetes. Actuellement, la visibilité sur l'état des services est fragmentée. Il est nécessaire d'implémenter une solution de supervision centralisée pour monitorer les métriques clés (taux d'erreurs du proxy NGINX, requêtes DNS, état des pods Kubernetes). La solution doit être légère, gratuite, facilement maintenable, s'intégrer avec Authentik via OIDC, et permettre le partage public de tableaux de bord sans authentification.

## 2. Cahiers des charges

| ID | Type | Exigence | Description |
| :--- | :--- | :--- | :--- |
| **[REQ-F01]** | Fonctionnel | Centralisation | Agrégation des métriques d'OPNsense, Debian et K8s sur une interface unique. |
| **[REQ-F02]** | Fonctionnel | Tableaux de bord publics | Possibilité d'exposer des tableaux de bord en lecture seule sans connexion préalable. |
| **[REQ-T01]** | Technique | Empreinte réduite | La solution doit être légère, peu consommatrice en ressources et facile à maintenir. |
| **[REQ-T02]** | Technique | Compatibilité OIDC | Intégration native avec le fournisseur d'identité Authentik. |
| **[REQ-T03]** | Technique | Gratuité | Solution 100% gratuite. |

## 3. Les solutions du marché

### 3.1. Présentations des solutions

#### 3.1.1. Stack Prometheus & Grafana
* **Présentation générale :** Suite open-source standard du marché cloud-native. Prometheus collecte les métriques (mode pull) et Grafana assure la visualisation.
* **Fonctionnement :** Des "exporters" (node-exporter, kube-state-metrics) exposent les métriques, scrapées par Prometheus. Grafana interroge Prometheus en tant que source de données.
* **Profil :** Idéal pour les environnements dynamiques (Kubernetes) et l'automatisation déclarative.

#### 3.1.2. Zabbix
* **Présentation générale :** Outil de supervision monolithique open-source d'entreprise.
* **Fonctionnement :** Utilise des agents installés sur les cibles et stocke la configuration/métriques dans une base de données relationnelle (PostgreSQL/MySQL).
* **Profil :** Adapté aux infrastructures traditionnelles et à la gestion centralisée des alertes, moins agile pour les conteneurs.

### 3.2. Comparatifs des solutions

| Exigence | Stack Prometheus & Grafana | Zabbix |
| :--- | :--- | :--- |
| **[REQ-F01] Centralisation** | Validé | Validé |
| **[REQ-F02] TdB Publics** | Validé (Fonction *Anonymous Access* native) | Limité (Nécessite la création complexe d'un compte invité restrictif) |
| **[REQ-T01] Empreinte réduite** | Validé (Architecture découplée et légère) | Non validé (Lourd, base de données relationnelle exigée) |
| **[REQ-T02] OIDC** | Validé (Support natif dans Grafana) | Validé |
| **[REQ-T03] Gratuité** | Validé | Validé |

## 4. Solution proposée

La solution proposée pour l'infrastructure est la **Stack Prometheus & Grafana**.

Cette stack s'intègre nativement à l'écosystème Kubernetes de LoutikCLOUD, garantissant une maintenance minimale en cohérence avec les pratiques d'Infrastructure as Code. Grafana sera configuré avec son intégration OIDC pointant vers Authentik pour l'administration sécurisée. Prometheus interrogera les exportateurs déployés sur les instances Debian, le cluster K8s et le pare-feu OPNsense pour offrir un point d'observation unifié des métriques (taux d'erreur NGINX, requêtes DNS, etc.).

**Justification du rejet des solutions alternatives :**

* **Zabbix :** Écarté en raison de sa consommation de ressources plus élevée et de la nécessité de maintenir une infrastructure de base de données relationnelle lourde, ce qui viole l'exigence de légèreté.
* **Solutions SaaS (Datadog, New Relic) :** Écartées car elles ne respectent pas l'exigence stricte de gratuité et impliquent une dépendance forte envers un service cloud externe.