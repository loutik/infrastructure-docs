---
title: Choix de l'orchestrateur kubernetes
date: 2026-08-18
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
L'infrastructure sous-jacente repose sur un cluster de trois serveurs physiques Proxmox VE. Le besoin est de déployer une plateforme d'orchestration de conteneurs hautement disponible (HA) pour héberger les services de l'environnement homelab. L'architecture cible doit intégrer trois nœuds Control Plane (distribués à raison d'un par nœud Proxmox) exposés via une adresse IP virtuelle (VIP), ainsi que deux nœuds Workers (répartis sur PVE1 et PVE2). L'environnement étant contraint, la solution requiert une grande simplicité d'exploitation et une consommation de ressources optimisée.

## 2. Cahiers des charges

| ID | Type | Exigence | Description |
| :--- | :--- | :--- | :--- |
| **[REQ-F01]** | Fonctionnel | Haute Disponibilité (HA) | Le cluster doit tolérer la perte d'un serveur physique Proxmox sans interruption du service d'orchestration. |
| **[REQ-T01]** | Technique | Topologie 3 CP + 2 Workers | Capacité à déployer une architecture multi-master avec une VIP pour l'API Server. |
| **[REQ-T02]** | Technique | Faible empreinte ressource | Optimisation de la consommation RAM et CPU pour préserver les ressources de l'homelab. |
| **[REQ-F02]** | Fonctionnel | Simplicité d'exploitation | Installation, configuration et gestion quotidienne aisées via des outils d'automatisation. |

## 3. Les solutions du marché

### 3.1. Présentations des solutions

#### 3.1.1. K3s
* **Présentation générale :** Distribution Kubernetes certifiée, allégée et open-source développée par Rancher.
* **Fonctionnement :** Empaquette les composants Kubernetes dans un binaire unique. Gère la HA avec un backend etcd embarqué.
* **Profil :** Conçu pour l'Edge computing, l'IoT, le CI/CD et les environnements homelab/ressources limitées.

#### 3.1.2. Kubernetes Vanilla (Kubeadm)
* **Présentation générale :** Distribution officielle standard (upstream) du projet Kubernetes.
* **Fonctionnement :** Déploiement modulaire complet nécessitant l'installation et la configuration de chaque composant (etcd, API, scheduler) de manière isolée.
* **Profil :** Datacenters et environnements de production d'entreprise (Enterprise Grade).

#### 3.1.3. MicroK8s
* **Présentation générale :** Distribution Kubernetes minimaliste maintenue par Canonical.
* **Fonctionnement :** S'installe de manière conteneurisée et packagée via le gestionnaire Snap. 
* **Profil :** Stations de travail de développement, tests locaux et environnements Edge sous Ubuntu.

### 3.2. Comparatifs des solutions

| Exigence | K3s | Kubernetes Vanilla | MicroK8s |
| :--- | :--- | :--- | :--- |
| **[REQ-F01 (HA)]** | Validé (etcd embarqué) | Validé (etcd externe) | Validé (dqlite) |
| **[REQ-T01 (Topologie)]** | Validé (kube-vip natif) | Validé (HAProxy/Keepalived) | Évaluation (VIP complexe) |
| **[REQ-T02 (Ressources)]** | Validé (Très léger) | Non validé (Overhead élevé) | Évaluation (Moyen) |
| **[REQ-F02 (Simplicité)]** | Validé (Binaire unique) | Non validé (PKI complexe) | Validé (Commandes Snap) |

## 4. Solution proposée

La solution proposée pour l'infrastructure est **K3s**.

K3s s'intègre parfaitement aux contraintes matérielles de l'homelab grâce à sa très faible empreinte mémoire et CPU. L'architecture déployée consistera en trois machines virtuelles Control Plane avec etcd embarqué (une par hyperviseur Proxmox pour garantir le quorum) couplées au démon `kube-vip` gérant la Virtual IP (VIP) en mode ARP ou BGP pour l'accès à l'API Server. Les deux machines virtuelles Workers seront instanciées sur PVE1 et PVE2 pour exécuter les charges de travail. Cette topologie assure une résilience complète face à la perte d'un nœud physique.

**Conséquences et impacts :**

* **Impacts positifs :** Réduction drastique de la consommation des ressources à vide, déploiement extrêmement rapide et automatisable, tolérance de panne validée.
* **Risques :** L'architecture nécessite une supervision stricte du quorum etcd (la perte simultanée de deux nœuds PVE entraînerait un blocage en écriture du cluster) et une configuration réseau précise pour le basculement transparent de la VIP via `kube-vip`.

**Justification du rejet des solutions alternatives :**

* **Kubernetes Vanilla (Kubeadm) :** Rejeté en raison de son overhead de ressources (consommation à vide trop importante pour un homelab) et de la lourdeur de maintenance opérationnelle (gestion manuelle de la PKI et du cluster etcd externe).
* **MicroK8s :** Rejeté à cause de sa forte dépendance à l'écosystème Canonical et au démon Snap, ce qui restreint le choix de la distribution Linux sous-jacente et ajoute une couche de complexité non désirée pour l'administration système globale.