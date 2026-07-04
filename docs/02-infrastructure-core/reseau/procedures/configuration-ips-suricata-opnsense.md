---
title: Configuration du système de prévention d'intrusions (IPS - Suricata)
service: Sécurité
date: 2026-07-03
author: Louis MEDO
owner: Louis MEDO
tags: [deploiement, iac, mise-en-production, securite, ips, suricata]
---

# {{ page.meta.title }}

![Bannière LoutikCLOUD](/assets/banniere_loutikcloud.png)

!!! info "Informations"
    * **Date de création** : {{ page.meta.date }}
    * **Service ciblé** : {{ page.meta.service }}
    * **Auteur** : {{ page.meta.author }}
    * **Responsable** : {{ page.meta.owner }}

## 1. Architecture et contexte
Cette procédure documente l'activation et la configuration du moteur Suricata en mode IPS (Intrusion Prevention System) sur le routeur OPNsense de l'infrastructure LoutikCLOUD. En plus de l'interception et du blocage actif des flux réseaux malveillants identifiés par les signatures via le mode `netmap`, cette procédure intègre une stratégie de "Whitelist" garantissant que le réseau d'administration (ADMOOB) ne soit jamais isolé accidentellement. La gestion des blocages est rationalisée par un ciblage strict des catégories de menaces critiques et une politique de destruction d'états (Kill States) immédiate.

## 2. Prérequis

* Pare-feu OPNsense déployé avec routage et accès Internet fonctionnels.
* Ressources CPU et RAM suffisantes allouées pour supporter l'analyse de paquets en temps réel (Pattern Matching).
* Identification stricte des interfaces réseaux physiques internes portant les VLANs.

## 3. Sécurisation de l'accès (Whitelist d'administration)

3.1. **Création d'une règle personnalisée (User defined).** Avant toute activation du blocage, naviguez dans `Services > Intrusion Detection > User defined` et ajoutez une règle d'autorisation absolue pour la plage réseau d'administration (VLAN ADMOOB 20).

  ```bash
  Enabled : Coché
  Source IP : 10.0.20.0/24
  Destination IP : 10.0.20.254
  Action : pass
  Description : Anti lockout de l'interface web OPNsense.
  ```

  `Action` : La directive `pass` ordonne au moteur de ne pas bloquer les paquets correspondant à cette règle, contournant ainsi toute fausse détection sur les administrateurs.

## 4. Configuration de l'IPS

4.1. **Activation du service et réglage du mode d'écoute.** Naviguez dans `Services > Intrusion Detection > Administration` et configurez les paramètres globaux.

  ```bash
  Enabled : Coché
  Capture mode : Divert (IPS)
  Pattern matcher : Hyperscan
  ```

  `IPS mode` : Bascule le moteur en mode blocage en exploitant le framework `netmap`.

  `Promiscuous mode` : Indispensable dans une architecture VLAN pour capturer le trafic encapsulé sur l'interface mère.

## 5. Gestion des sources de règles

5.1. **Sélection rigoureuse des catégories de menaces.** Dans l'onglet `Download`, activez exclusivement les listes critiques détaillées ci-dessous et désactivez toutes les autres pour éviter la surcharge et les faux positifs.

  ```bash
  abuse.ch/Feodo Tracker
  abuse.ch/ThreatFox
  abuse.ch/URLhaus
  ET open/botcc
  ET open/emerging-exploit
  ET open/emerging-malware
  ET open/emerging-mobile_malware
  ET open/emerging-web_client
  ```

  `abuse.ch/Feodo Tracker` : Bloque spécifiquement les serveurs de Command & Control (C2) associés aux botnets (ex: Emotet, TrickBot).

  `abuse.ch/ThreatFox` : Fournit des indicateurs de compromission (IOC) pour identifier et bloquer les serveurs C2 de malwares, chevaux de troie et backdoors.

  `abuse.ch/URLhaus` : Bloque les connexions sortantes vers les serveurs hébergeant des charges utiles malveillantes.

  `ET open/botcc` : Identifie et bloque les adresses IP reconnues comme agissant comme centres de commande de botnets (C2).

  `ET open/emerging-exploit` : Protège contre l'exploitation de vulnérabilités connues sur des systèmes et applications réseau.

  `ET open/emerging-malware` : Identifie et bloque l'activité réseau générée par des malwares, incluant les communications des backdoors et l'exfiltration de données.

  `ET open/emerging-mobile_malware` : Cible spécifiquement les menaces et logiciels malveillants visant les environnements et terminaux mobiles.

  `ET open/emerging-web_client` : Bloque les attaques ciblant les navigateurs web internes (kits d'exploits, redirections hostiles, scripts malveillants).

  *Activez les listes et cliquez sur **Download & Update Rules**.*

## 6. Stratégie de blocage (Policies)

6.1. **Application du blocage global et destruction d'états.** Dans l'onglet `Policies`, créez une politique ciblant les catégories activées précédemment pour les bloquer fermement.

  ```bash
  Enabled : Coché
  Rulesets : [Sélectionner les listes activées à l'étape 5]
  Action : Nothing selected
  New action : Drop
  Description : IPS_Block_Critical
  ```

  `New action` : Remplace l'action d'alerte par la destruction systématique des paquets suspects (`Drop`).

  *Cliquez sur **Save** puis sur **Apply** pour recharger le moteur.*

## 7. FAQ

1. **Récupération interface web.** En cas d'erreur de configuration entraînant un blocage complet (lockout) ou une fausse détection critique, connectez-vous au routeur via SSH depuis une machine du VLAN ADMOOB et exécutez la commande suivante pour désactiver temporairement l'IPS :

  ```bash
  service suricata stop
  ```

---

## Annexe

* [Référentiel des normes de nommage réseau LoutikCLOUD](/01-architecture/referentiels/ref-003-convention-nommage-reseau-pare-feu.md)