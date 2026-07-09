---
id: 004
title: Choix de la solution DNS pour l'infrastructure on-premise
date: 2026-07-09
status: accepted
author: Louis MEDO
owner: Louis MEDO
tags: [adr, architecture, dns]
---

# ADR-{{ page.meta.id }} : {{ page.meta.title }}

![Bannière LoutikCloud](https://raw.githubusercontent.com/loutik/design-assets/main/loutikcloud/banniere_loutikcloud.png)

!!! info "Informations"
    * **Date de création** : {{ page.meta.date }}
    * **Statut** : {{ page.meta.status }}
    * **Auteur** : {{ page.meta.author }}
    * **Responsable** : {{ page.meta.owner }}

## 1. Contexte
L'infrastructure on-premise sous-jacente au domaine `infra.loutik.fr` nécessite une gestion centralisée, dynamique et hautement disponible de ses enregistrements DNS. Actuellement, le provisionnement des machines s'oriente vers des processus automatisés (pipelines CI/CD, playbooks Ansible). L'absence d'un mécanisme d'enregistrement automatisé des noms pleinement qualifiés (FQDN) via API constitue un point de blocage pour l'automatisation globale de l'infrastructure. De plus, la tolérance aux pannes du service DNS (failover) doit être assurée par le déploiement d'une architecture redondante sur deux serveurs distincts.

## 2. Cahiers des charges

| ID | Type | Exigence | Description |
| :--- | :--- | :--- | :--- |
| **REQ-F01** | Fonctionnel | API Programmable | Capacité à créer, modifier et supprimer des enregistrements FQDN via des requêtes programmatiques. |
| **REQ-F02** | Fonctionnel | Haute Disponibilité / Failover | Support d'une architecture à deux nœuds pour assurer la continuité de service en cas de panne d'un serveur. |
| **REQ-T01** | Technique | Intégration Ansible / IaC | Disponibilité de modules ou de providers officiels/communautaires pour s'intégrer dans une philosophie Infrastructure as Code. |
| **REQ-T02** | Technique | Stockage persistant / Réplication | Flexibilité du backend de stockage permettant une synchronisation ou réplication native et robuste des zones entre les nœuds. |
| **REQ-T03** | Technique | Empreinte ressources | Consommation mémoire et CPU optimisée pour une intégration fluide en environnement de services virtualisés (on-premise). |
| **REQ-T04** | Technique | Compatibilité système (Debian) | Support officiel, installation et maintenance simplifiées via les gestionnaires de paquets (APT) sur un système d'exploitation GNU/Linux Debian. |

## 3. Les solutions du marché

### 3.1. Présentations des solutions

#### 3.1.1. PowerDNS (Authoritative Server)

* **Présentation générale :** Serveur DNS autoritaire open-source moderne conçu pour les infrastructures à forte charge et les environnements automatisés.
* **Fonctionnement :** Contrairement aux serveurs traditionnels basés sur des fichiers de zone plats, PowerDNS s'appuie sur des backends relationnels (PostgreSQL, MySQL, SQLite) pour stocker les données DNS et expose une API REST native.
* **Profil :** Solution orientée ingénierie DevOps et production d'entreprise, hautement compatible avec les outils d'automatisation modernes.

#### 3.1.2. BIND9

* **Présentation générale :** Le serveur DNS de référence historique sur Internet, reconnu pour sa stabilité absolue et son respect strict des RFC.
* **Fonctionnement :** Fonctionne de manière native avec des fichiers de configuration texte. Les mises à jour dynamiques s'effectuent via le protocole standard RFC 2136 à l'aide de l'utilitaire `nsupdate` et de clés TSIG.
* **Profil :** Solution traditionnelle, robuste et académique, privilégiant la standardisation historique aux interfaces HTTP modernes.

#### 3.1.3. CoreDNS

* **Présentation générale :** Serveur DNS rapide et modulaire écrit en Go, devenu le standard de facto pour la résolution de noms dans l'écosystème Cloud-Native (Kubernetes).
* **Fonctionnement :** S'appuie sur une chaîne de plugins configurables via un "Corefile". Il s'interface dynamiquement avec des backends de type clé-valeur distribués comme `etcd` pour la synchronisation.
* **Profil :** Solution hautement spécialisée pour les infrastructures conteneurisées et les architectures micro-services.

#### 3.1.4. Technitium DNS

* **Présentation générale :** Serveur DNS faisant office de serveur autoritaire et de résolveur, développé en C#/.NET, intégrant une interface graphique moderne.
* **Fonctionnement :** Gère les zones via des bases locales ou des fichiers, et rend l'intégralité de ses fonctionnalités accessibles via une API HTTP REST.
* **Profil :** Solution accessible et polyvalente, adaptée pour les environnements de laboratoires et les infrastructures de PME.

### 3.2. Comparatifs des solutions

| Exigence | PowerDNS | BIND9 | CoreDNS | Technitium DNS |
| :--- | :--- | :--- | :--- | :--- |
| **REQ-F01 (API REST Native)** | Validé (REST exhaustive) | Non validé (Nécessite RFC 2136) | Non validé (Via plugins/etcd) | Validé (HTTP REST) |
| **REQ-F02 (Failover / HA)** | Validé (Réplication DB ou Master/Slave) | Validé (Transfert AXFR/IXFR natif) | Validé (Via cluster etcd externe) | Validé (Transfert de zone standard) |
| **REQ-T01 (Intégration IaC)** | Excellent (Modules Ansible / Terraform) | Moyen (Modules Ansible spécifiques) | Moyen (Configuration via GitOps) | Faible (API peu intégrée dans l'écosystème Ansible) |
| **REQ-T02 (Backend de stockage)** | Flexible (PostgreSQL, MySQL) | Rigide (Fichiers plats textes) | Spécialisé (etcd, mémoire) | Propriétaire / Fichiers |
| **REQ-T03 (Légèreté / On-prem)** | Excellent | Excellent | Très bon | Moyen (Dépendance runtime .NET) |
| **REQ-T04 (Compatibilité Debian)** | Excellent (Dépôts APT officiels) | Excellent (Paquet standard) | Bon (Binaire unique) | Moyen (Dépendance runtime) |

## 4. Solution proposée

La solution proposée pour l'infrastructure LoutikCloud est **PowerDNS Authoritative Server**.

L'intégration d'une API REST native au sein de PowerDNS élimine la complexité liée à la gestion des fichiers plats et permet un provisionnement en temps réel des enregistrements FQDN depuis l'infrastructure de déploiement Ansible. La redondance du service et le mécanisme de bascule (failover) seront assurés par le déploiement de deux instances PowerDNS. Ces instances seront adossées à un cluster de base de données PostgreSQL hautement disponible situé dans le VLAN 17 (ZDS), offrant ainsi une synchronisation des enregistrements DNS instantanée et sans perte de données, tout en maintenant les fichiers de configuration système sous le contrôle de la philosophie GitOps.

**Justification du rejet des solutions alternatives :**

* **BIND9 :** Bien que parfaitement stable et nativement optimisé pour Debian, BIND9 est écarté en raison de l'absence d'une API REST moderne. Le recours obligatoire au protocole RFC 2136 (`nsupdate`) ajoute une complexité non négligeable pour l'intégration continue et la gestion idempotente via Ansible.
* **CoreDNS :** Malgré d'excellentes performances, CoreDNS est rejeté car son architecture est conçue pour des environnements Kubernetes. L'implémentation de la haute disponibilité hors cluster nécessiterait le déploiement et la maintenance d'une base distribuée `etcd` additionnelle, créant une sur-ingénierie injustifiée pour une infrastructure on-premise basée sur des machines virtuelles classiques.
* **Technitium DNS :** Écarté pour des raisons techniques liées au socle d'exploitation. Son développement en C# impose l'installation du runtime .NET sur les serveurs Debian, ce qui alourdit l'empreinte système. De plus, son écosystème communautaire (modules Ansible et providers Terraform) est nettement moins mature que celui de PowerDNS.