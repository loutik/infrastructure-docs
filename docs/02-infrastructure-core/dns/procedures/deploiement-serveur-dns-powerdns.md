---
title: Déploiement du serveur DNS autoritaire - PowerDNS
service: PowerDNS
date: 2026-07-11
author: Louis MEDO
owner: Louis MEDO
---

# {{ page.meta.title }}

![Bannière LoutikCloud](https://raw.githubusercontent.com/loutik/design-assets/main/loutikcloud/banniere_loutikcloud.png)

!!! info "Informations"
    * **Date de création** : {{ page.meta.date }}
    * **Service ciblé** : {{ page.meta.service }}
    * **Auteur** : {{ page.meta.author }}
    * **Responsable** : {{ page.meta.owner }}

## 1. Contexte

Ce composant assure la résolution DNS interne et autoritaire pour l'environnement LoutikCLOUD. Intégré dans l'orchestrateur WandOps, PowerDNS s'exécute selon une architecture *stateless* dans la DMZ Infrastructure (VLAN 13). Il dépend fonctionnellement d'un backend PostgreSQL isolé en Zone de Diffusion Restreinte (ZDR, VLAN 17) pour le stockage persistant des zones et enregistrements. La gestion des zones s'effectue exclusivement via des appels API REST pour garantir l'idempotence et le respect des standards SRE.

## 2. Prérequis

* **Ressources d'infrastructure :** Machine virtuelle allouée sur le cluster Proxmox (ex: `mlt1-dns-vm-prd-01`).
* **Réseau :** Connectivité autorisée vers le dépôt distant `repo.powerdns.com` (via proxy) et vers le backend PostgreSQL (`10.0.17.1:5432`).
* **Base de données :** Schéma SQL PowerDNS préalablement injecté dans la base `db_powerdns`.
* **Gestion des accès :** Clé API REST et identifiants de base de données documentés et intégrés au Vault.

## 3. Procédure de déploiement

3.1. **Désactivation du résolveur local.** Libération du port d'écoute standard pour prévenir tout conflit matériel et configuration d'un DNS de secours.

  ```bash
  sudo systemctl stop systemd-resolved
  sudo systemctl disable systemd-resolved
  echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf
  ```

  `systemd-resolved` : Service natif de l'OS qui écoute par défaut sur le port 53.

  `nameserver 8.8.8.8` : Instruction statique garantissant la résolution de noms pendant la phase de provisionnement.

3.2. **Configuration des sources APT.** Importation de la clé GPG officielle et déclaration du dépôt PowerDNS avec un épinglage strict.

  ```bash
  sudo install -d /etc/apt/keyrings
  curl https://repo.powerdns.com/FD380FBB-pub.asc | sudo tee /etc/apt/keyrings/auth-51-pub.asc
  echo "deb [signed-by=/etc/apt/keyrings/auth-51-pub.asc] http://repo.powerdns.com/debian trixie-auth-51 main" | sudo tee /etc/apt/sources.list.d/pdns.list
  sudo tee /etc/apt/preferences.d/auth-51 << EOF
  Package: pdns-*
  Pin: origin repo.powerdns.com
  Pin-Priority: 600
  EOF
  ```

  `signed-by` : Directive de sécurité forçant la vérification cryptographique des paquets via le trousseau isolé.

  `Pin-Priority: 600` : Règle forçant le gestionnaire de paquets à privilégier la branche amont officielle de PowerDNS plutôt que celle des dépôts Debian.

3.3. **Installation des binaires.** Mise à jour de l'index des paquets et déploiement du service principal et de son connecteur de base de données.

  ```bash
  sudo apt-get update && sudo apt-get install pdns-server pdns-backend-pgsql
  ```

  `pdns-server` : Le daemon principal assurant le traitement des requêtes UDP/TCP.

  `pdns-backend-pgsql` : Le module d'extension permettant la communication native avec PostgreSQL.

3.4. **Configuration du plan de contrôle et de données.** Édition du fichier principal pour lier la base de données et exposer l'interface d'administration.

  **Edition du fichier de configuration :**
  ```bash
  sudo vim /etc/powerdns/pdns.conf
  ```

  **Exemple de configuration pour PowerDNS :**
  ````
  # ---
  # Type        : Template de configuration du service PowerDNS
  # Auteur      : Louis MEDO - louis.medo@loutik.fr
  # ---
  # Fichier généré dynamiquement par Ansible (Rôle : pdns-install)
  # ⚠️ Toute modification manuelle sera écrasée. ⚠️
  # ---

  # ==============================
  # Privileges
  # ==============================

  setuid=pdns
  setgid=pdns
  version-string=anonymous

  # ==============================
  # DNS (Port 53)
  # ==============================

  # --- Network ---
  local-address=10.0.13.1
  local-port=53

  # --- Configuration ---
  security-poll-suffix=

  # ==============================
  # DATABASE
  # ==============================

  # --- PostgreSQL ---
  launch=gpgsql
  gpgsql-host=10.0.17.1
  gpgsql-port=5432
  gpgsql-dbname=db-name
  gpgsql-user=db-user
  gpgsql-password=changer-db-password
  gpgsql-dnssec=yes

  # ==============================
  # API
  # ==============================

  # --- Configuration ---
  api=yes
  api-key=changer-api-key
  webserver=yes

  # --- Network ---
  webserver-address=10.0.13.1
  webserver-port=8081
  webserver-allow-from=127.0.0.1,::1,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16
  ````

  `launch=gpgsql` : Paramètre définissant le moteur de stockage persistant ciblant l'instance `10.0.17.1`.

  `api-key=changeme` : Jeton d'authentification statique utilisé par OpsBricks pour piloter les zones.

3.5. **Initialisation du service.** Application de la configuration asynchrone et démarrage du daemon DNS.

  ```bash
  sudo systemctl restart pdns
  ```

  `restart` : Commande de contrôle systemd pour purger l'état précédent et recharger la configuration depuis le backend SQL.

## 4. Vérification

4.1. **Test de connectivité API.** Interrogation du point de terminaison REST pour confirmer l'authentification et l'état opérationnel du plan de contrôle.

  ```bash
  curl -X GET -H 'X-API-Key: changeme' http://<ip-serveur-dns>:8081/api/v1/servers/localhost | jq .
  ```

  `X-API-Key` : En-tête HTTP obligatoire pour authentifier la requête auprès du webserver de PowerDNS.

  `/api/v1/servers/localhost` : Route de l'API REST retournant l'état global et les métadonnées de l'instance.

4.2. **Test de résolution DNS.** Interrogation du daemon sur le port 53 pour valider la capacité de réponse depuis le plan de données.

  ```bash
  dig @<ip-serveur-dns> test.loutik.fr A +short
  ```

  `@127.0.0.1` : Argument contraignant l'outil de diagnostic à cibler directement l'instance PowerDNS locale, contournant tout cache externe.

  `+short` : Option de formatage affichant uniquement l'adresse IP résolue, idéale pour les validations algorithmiques.