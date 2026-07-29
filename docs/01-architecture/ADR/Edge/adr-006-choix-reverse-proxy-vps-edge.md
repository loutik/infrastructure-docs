---
id: 0005
title: Choix du Reverse Proxy pour la VM Edge
date: 2026-07-29
status: accepted
author: Louis MEDO
owner: Louis MEDO
tags: [adr, architecture, reseau, securite]
---

# ADR-{{ page.meta.id }} : {{ page.meta.title }}

![Bannière LoutikCloud](https://raw.githubusercontent.com/loutik/design-assets/main/loutikcloud/banniere_loutikcloud.png)

!!! info "Informations"
    * **Date de création** : {{ page.meta.date }}
    * **Statut** : {{ page.meta.status }}
    * **Auteur** : {{ page.meta.author }}
    * **Responsable** : {{ page.meta.owner }}

## 1. Contexte

La VM "Edge", directement exposée sur Internet, nécessite un composant de reverse proxy robuste agissant comme point d'entrée unique de l'infrastructure. Afin de garantir la sécurité, la maintenabilité et l'observabilité, nous devons sélectionner une solution standardisée. Ce composant doit pouvoir être déployé et configuré de manière totalement automatisée (IaC), supporter l'injection dynamique de contexte dans les pages d'erreur (ex: affichage du domaine en échec), s'interfacer avec un pare-feu applicatif (WAF) et s'intégrer nativement à notre stack de monitoring Prometheus, le tout sur un système de base Debian.

## 2. Cahiers des charges

| ID | Type | Exigence | Description |
| :--- | :--- | :--- | :--- |
| **[REQ-T01]** | Technique | Compatibilité Debian native | La solution doit être installable et maintenable via les dépôts officiels sur Debian (apt), sans nécessiter de compilation manuelle complexe. |
| **[REQ-T02]** | Technique | Capacités d'automatisation | La configuration doit pouvoir être gérée as-code via des rôles Ansible, avec des rechargements à chaud sans interruption de service. |
| **[REQ-F01]** | Fonctionnel | Pages d'erreur dynamiques | Capacité à personnaliser les pages d'erreur (ex: 404, 502) en injectant dynamiquement des variables (comme le nom de domaine requis). |
| **[REQ-F02]** | Fonctionnel | Interopérabilité WAF | Possibilité d'intégrer un Web Application Firewall (ex: ModSecurity, Coraza) pour filtrer les requêtes malveillantes. |
| **[REQ-T03]** | Technique | Observabilité Prometheus | Mise à disposition d'exporteurs de métriques natifs ou officiellement supportés pour le scraping par Prometheus. |

## 3. Les solutions du marché

### 3.1. Présentations des solutions

#### 3.1.1. Nginx
* **Présentation générale :** Serveur web open-source performant, massivement adopté comme reverse proxy et load balancer.
* **Fonctionnement :** Architecture asynchrone orientée événements. Configuration via des fichiers déclaratifs structurés en blocs.
* **Profil :** Standard de l'industrie, très adapté aux déploiements bare-metal/VM avec une gestion fine par Ansible et une communauté massive.

#### 3.1.2. Traefik
* **Présentation générale :** Reverse proxy open-source moderne et cloud-native, écrit en Go.
* **Fonctionnement :** Découverte dynamique des services, configuration automatique basée sur les labels (Docker, K8s) ou des providers de fichiers.
* **Profil :** Orienté écosystème conteneurisé et orchestrateurs, idéal pour les environnements fortement dynamiques.

#### 3.1.3. Caddy
* **Présentation générale :** Serveur web moderne écrit en Go, axé sur la simplicité et le "HTTPS-by-default".
* **Fonctionnement :** Gestion automatique des certificats TLS, configuration via un `Caddyfile` très concis. Extensible via des modules.
* **Profil :** Orienté développeurs, homelabs et infrastructures cherchant une mise en place extrêmement rapide.

### 3.2. Comparatifs des solutions

| Exigence | Nginx | Traefik | Caddy |
| :--- | :--- | :--- | :--- |
| **[REQ-T01 (Debian)]** | Validé | Évaluation (binaire statique / apt) | Non validé (Nécessite `xcaddy` pour ajouter des modules WAF) |
| **[REQ-T02 (Automation)]**| Validé (Très mature via Ansible) | Validé (Fichier dynamique ou API) | Validé (API native ou Caddyfile) |
| **[REQ-F01 (Erreurs)]** | Validé (Via SSI `<!--# echo var="host" -->` ou Lua) | Non validé (Gestion basique des erreurs, injection complexe) | Validé (Via directives de `templates`) |
| **[REQ-F02 (WAF)]** | Validé (Modules ModSecurity ou Coraza supportés) | Évaluation (Via plugins externes) | Évaluation (Via module Coraza nécessitant un build sur mesure) |
| **[REQ-T03 (Metrics)]** | Validé (Via `nginx-prometheus-exporter`) | Validé (Natif) | Validé (Natif) |

## 4. Solution proposée

La solution proposée pour l'infrastructure est **Nginx**.

Nginx répond à l'intégralité de nos exigences avec un niveau de maturité et de stabilité maximal pour une VM Edge fonctionnant sous Debian. L'intégration au système sera gérée par Ansible (déploiement de la configuration as-code). Les pages d'erreur dynamiques seront implémentées en utilisant les Server Side Includes (SSI) natifs de Nginx, permettant d'injecter la variable `$host` directement dans le code HTML renvoyé au client, simulant ainsi le comportement de solutions comme Cloudflare. La sécurité sera assurée par l'intégration du module WAF Coraza, compilé dynamiquement ou installé via les paquets pré-compilés. Enfin, l'observabilité sera garantie par le déploiement en side-car du binaire `nginx-prometheus-exporter`.

**Justification du rejet des solutions alternatives :**

* **Caddy :** Bien que très séduisant pour sa gestion des certificats et ses templates d'erreurs natifs, l'ajout du WAF Coraza nécessite l'utilisation de l'outil de build `xcaddy`. Cela nous obligerait à gérer des binaires personnalisés, rompant avec l'exigence de maintenabilité native via les gestionnaires de paquets Debian (REQ-T01) et alourdissant le pipeline de mise à jour de sécurité.
* **Traefik :** Solution d'excellence pour Kubernetes ou Docker, mais surdimensionnée et moins adaptée pour une VM bare-metal statique. La gestion des pages d'erreur dynamiques avec injection de contexte réseau est complexe et nécessite souvent le déploiement d'un micro-service tiers de gestion d'erreurs, complexifiant inutilement l'architecture.
* **HAProxy :** Considéré lors de la réflexion initiale, mais rejeté car il s'agit d'un pur load-balancer de niveau 4/7. Sa capacité à servir des pages HTML complexes et dynamiques (avec des variables) sans backend dédié est extrêmement limitée par rapport à un serveur web complet comme Nginx.