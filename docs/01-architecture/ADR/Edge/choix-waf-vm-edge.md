---
title: Choix du WAF et IPS pour la VM Edge
date: 2026-07-29
status: accepted
author: Louis MEDO
owner: Louis MEDO
tags: [adr, architecture, securite, waf, ips]
---

# ADR : {{ page.meta.title }}

![Bannière LoutikCloud](https://raw.githubusercontent.com/loutik/design-assets/main/loutikcloud/banniere_loutikcloud.png)

!!! info "Informations"
    * **Date de création** : {{ page.meta.date }}
    * **Statut** : {{ page.meta.status }}
    * **Auteur** : {{ page.meta.author }}
    * **Responsable** : {{ page.meta.owner }}

## 1. Contexte
Dans le cadre de la sécurisation de la VM "Edge" exposée sur Internet, il est nécessaire de filtrer les requêtes HTTP/HTTPS malveillantes (rôle du WAF). Cependant, cette VM est également exposée à des attaques réseau globales (bruteforce SSH, scans de ports). L'objectif est de sélectionner une solution unifiée, gratuite et automatisable (IaC) capable de couvrir de manière centralisée les couches réseau (IPS) et applicative (WAF), tout en exploitant des listes de réputation d'IP publiques mises à jour en continu. La solution doit nativement exporter ses métriques vers Prometheus et permettre une gestion fine des faux positifs pour les applications internes générant un fort volume de requêtes.

## 2. Cahiers des charges

| ID | Type | Exigence | Description |
| :--- | :--- | :--- | :--- |
| **[REQ-F01]** | Fonctionnel | Protection unifiée (WAF + IPS) | Capacité à bloquer simultanément les attaques applicatives (injections, XSS) et les attaques réseau (SSH bruteforce, scans). |
| **[REQ-F02]** | Fonctionnel | Threat Intelligence collaborative | Utilisation et mise à jour automatique de listes publiques d'IP malveillantes (Blocklists) mutualisées par la communauté. |
| **[REQ-T01]** | Technique | Gestion des faux positifs (Allowlist) | Configuration simple as-code de listes blanches pour éviter le blocage des IP d'administration et des outils internes légitimes (ex: Grafana). |
| **[REQ-T02]** | Technique | Observabilité Prometheus | Export natif des métriques (alertes déclenchées, décisions de blocage, scénarios) pour le scraping par Prometheus. |

## 3. Les solutions du marché

### 3.1. Présentations des solutions

#### 3.1.1. ModSecurity
* **Présentation générale :** Le WAF open-source historique de référence.
* **Fonctionnement :** Analyse le trafic HTTP entrant via un module serveur web (connecteur dynamique).
* **Profil :** Strictement limité à la couche applicative (L7), vieillissant et sans intégration native des menaces réseau.

#### 3.1.2. Coraza WAF couplé à Fail2Ban
* **Présentation générale :** WAF nouvelle génération en Go (Coraza), associé à l'IPS historique (Fail2Ban) pour le réseau.
* **Fonctionnement :** Coraza filtre le trafic HTTP (OWASP CRS), tandis que Fail2Ban parse les logs système pour bannir les IP sur le pare-feu local (iptables/nftables).
* **Profil :** Couvre l'ensemble du périmètre, mais multiplie les agents de sécurité indépendants sans intelligence collaborative.

#### 3.1.3. CrowdSec
* **Présentation générale :** Moteur IPS moderne, collaboratif, doté d'un composant AppSec (WAF).
* **Fonctionnement :** Analyse les logs de multiples services (SSH, système) et inspecte le trafic HTTP à la volée via un *Bouncer* lié au reverse proxy. Partage et récupère les IP malveillantes via une API centrale.
* **Profil :** Solution de sécurité holistique (réseau et applicatif) orientée automatisation (DevSecOps) et Threat Intelligence.

### 3.2. Comparatifs des solutions

| Exigence | ModSecurity | Coraza + Fail2Ban | CrowdSec |
| :--- | :--- | :--- | :--- |
| **[REQ-F01 (WAF + IPS)]** | Non validé (WAF uniquement) | Validé (Mais nécessite deux stacks distinctes) | Validé (Agent unique avec module AppSec) |
| **[REQ-F02 (Listes IP)]** | Non validé | Non validé (Protection isolée localement) | Validé (Base de réputation mondiale intégrée) |
| **[REQ-T01 (Allowlist)]** | Validé (Exceptions via SecLang) | Validé | Validé (Via fichiers YAML dédiés au whitelisting) |
| **[REQ-T02 (Metrics)]** | Non validé | Validé pour Coraza, complexe pour Fail2Ban | Validé (Exporteur Prometheus natif unifié) |

## 4. Solution proposée

La solution proposée pour l'infrastructure est **CrowdSec**.

Le choix de CrowdSec permet de consolider l'architecture de sécurité de la VM Edge autour d'un agent unique. Il répond au besoin de filtrage applicatif (WAF) via son module AppSec, tout en protégeant les couches système et réseau (IPS), notamment le port SSH. Cette unification réduit considérablement la dette technique et simplifie le déploiement via Ansible. L'atout majeur de l'outil réside dans son moteur collaboratif : la VM Edge bénéficie préventivement de listes publiques d'adresses IP malveillantes constamment mises à jour, bloquant les botnets avant même qu'ils n'atteignent nos services.

Pour maîtriser le risque de faux positifs inhérent aux IPS sur des applications bavardes (comme les appels API fréquents de Grafana), la configuration Ansible déploiera des fichiers d'allowlist stricts, garantissant que les adresses IP d'administration et les réseaux internes ne soient jamais bloqués. Enfin, l'observabilité est totalement intégrée avec un exporteur Prometheus natif centralisant les métriques de sécurité.

**Justification du rejet des solutions alternatives :**

* **Coraza WAF (+ Fail2Ban) :** Bien que Coraza soit un excellent WAF, il ne couvre pas les attaques sur le port SSH. L'ajout d'un second outil comme Fail2Ban pour combler ce manque créerait une architecture fragmentée et complexe à maintenir as-code. De plus, cette combinaison ne bénéficie d'aucune Threat Intelligence communautaire (les blocages ne se font qu'en réaction à des attaques locales).
* **ModSecurity :** Solution purement L7, techniquement vieillissante, sans protection réseau globale ni export Prometheus natif. Son adoption pour une infrastructure moderne constituerait une régression architecturale.