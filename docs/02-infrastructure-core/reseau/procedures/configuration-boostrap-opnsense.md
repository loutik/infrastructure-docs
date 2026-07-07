---
title: Configuration bootstrap - OPNsense
service: Réseau
date: 2026-06-30
author: Louis MEDO
owner: Louis MEDO
tags: [deploiement, bootstrap, iac, securite]
---

# {{ page.meta.title }}

![Bannière LoutikCLOUD](../../../assets/banniere_loutikcloud.png)

!!! info "Informations"
    * **Date de création** : {{ page.meta.date }}
    * **Service ciblé** : {{ page.meta.service }}
    * **Auteur** : {{ page.meta.author }}
    * **Responsable** : {{ page.meta.owner }}

## 1. Architecture et contexte
Cette documentation détaille la phase de "Bootstrap" post-installation du routeur OPNsense LoutikCLOUD. Ce socle de configuration manuel est strictement nécessaire pour préparer l'environnement à l'orchestration GitOps. Il normalise l'identité du routeur, sécurise les accès d'administration avec des standards robustes, aligne la résolution DNS et prépare le service SSH. Une fois cette procédure appliquée, le pare-feu sera prêt à recevoir de manière fiable ses futures configurations IaC (Ansible) pour les réseaux, le filtrage, le SSO ou les tunnels Tailscale.

## 2. Prérequis

* L'installation de base d'OPNsense est finalisée (cf. procédure d'installation).
* L'interface d'administration Web est accessible via l'IP LAN (`10.0.20.1`).
* Un accès au gestionnaire de mots de passe (Bitwarden) pour générer un secret de 32 caractères.

## 3. Configuration de l'identité et du temps

3.1. **Configuration du FQDN et des serveurs DNS.** Depuis `System > Settings > General`, configurez l'identité réseau du routeur et les serveurs de résolution externes.

!!! warning "NORMES"
    Le hostname est défini par le référentiel **Convention de nommage infrastructure** que vous pouvez consulter ici : [Convention de nommage infrastructure](../../../01-architecture/referentiels/ref-001-convention-nommage-infrastructure.md).

  ```bash
  Hostname : mlt1-opn-fw-prd-01
  Domain : infra.loutik.fr
  DNS Servers : 9.9.9.9, 1.1.1.1, 8.8.8.8
  DNS search domain: infra.loutik.fr, custo.loutik.fr
  ```

  `Hostname` : Nom de la machine défini selon la convention de nommage.

  `Domain` : Domaine principal de l'infrastructure locale.

  `DNS Servers` : Serveurs de résolution récursifs externes (Quad9, Cloudflare, Google).

3.2. **Configuration du fuseau horaire et NTP.** Toujours dans `System > Settings > General`, alignez l'horloge du routeur, élément critique pour la validation des certificats TLS et les logs.

  ```bash
  Time zone : Europe/Paris
  ```

  `Time zone` : Définit le fuseau horaire local.

## 4. Sécurisation et accès (Préparation Ansible)

4.1. **Renforcement du compte Root.** Depuis `System > Access > Users`, modifiez l'utilisateur `root` pour appliquer la norme de sécurité du mot de passe.

  ```bash
  Password : [Générer une chaîne aléatoire de 32 caractères]
  ```

  `Password` : Mot de passe extrêmement robuste, à stocker immédiatement dans le coffre-fort de mots de passe de l'infrastructure.

## 5. Configuration DNS

5.1. **Désactivation de Dnsmasq.** Depuis `Services > Dnsmasq DNS > Settings`, désactivez ce service pour éviter les conflits de ports avec Unbound.

  ```bash
  Enable Dnsmasq : Décoché
  ```

  `Enable Dnsmasq` : Libère le port 53 pour le résolveur principal de l'infrastructure.

5.2. **Configuration d'Unbound DNS en mode relais.** Depuis `Services > Unbound DNS > General`, activez Unbound pour traiter les requêtes LAN et les transférer vers les DNS configurés précédemment.

  ```bash
  Enable Unbound : Coché
  ```

  `Enable Unbound` : Active le résolveur DNS principal sur le routeur.

## 6. Accès internet

6.1. **Désactivation du blocage RFC1918 sur le WAN.** Depuis `Interfaces > [WAN]`, assurez-vous que le pare-feu n'entrave pas le flux si le routeur est placé derrière un modem fournisseur d'accès internet (FAI).

  ```bash
  Block private networks : Décoché
  Block bogon networks : Décoché
  ```

  `Block private networks` : Autorise l'adresse IP WAN (ex: `172.16.1.253`) à communiquer avec la passerelle du réseau local du FAI.

## Annexe

- [Référentiel des normes de nommage réseau LoutikCLOUD](../../../01-architecture/referentiels/ref-003-convention-nommage-reseau-pare-feu.md)