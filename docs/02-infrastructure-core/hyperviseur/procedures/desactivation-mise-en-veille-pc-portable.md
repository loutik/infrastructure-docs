---
title: Désactivation de la mise en veille (Serveur Portable)
service: Proxmox VE
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

## 1. Architecture et contexte
Cette procédure permet de maintenir un nœud hyperviseur Proxmox hébergé sur un ordinateur portable actif lors de la fermeture physique de l'écran.

## 2. Prérequis

* Hôte Proxmox VE installé sur un châssis de type ordinateur portable.
* Accès SSH ou console physique avec des privilèges `root` sur l'hyperviseur.

## 3. Configuration des événements matériels

3.1. **Modification du comportement du capot.** Altération des règles du gestionnaire de connexion pour ignorer le déclencheur de fermeture de l'écran, tant sur batterie que sur secteur.

  ```bash
  sed -i 's/#HandleLidSwitch=suspend/HandleLidSwitch=ignore/g' /etc/systemd/logind.conf
  sed -i 's/#HandleLidSwitchExternalPower=suspend/HandleLidSwitchExternalPower=ignore/g' /etc/systemd/logind.conf
  ```

  `sed -i` : Utilitaire d'édition de flux (Stream Editor). L'argument `-i` (in-place) applique les modifications directement dans le fichier cible sans créer de fichier temporaire manuel.

  `HandleLidSwitch=ignore` : Directive de configuration remplaçant l'action par défaut (`suspend`). Elle instruit le démon d'ignorer purement et simplement l'événement physique de fermeture du capot.

3.2. **Application des nouveaux paramètres.** Redémarrage du service de gestion des sessions pour recharger la configuration à chaud.

  ```bash
  systemctl restart systemd-logind.service
  ```

  `systemctl restart` : Commande d'exécution déclenchant l'arrêt puis le démarrage immédiat du processus ciblé.

  `systemd-logind.service` : Démon système gérant le cycle de vie des sessions et la réponse aux boutons/switchs matériels. Son redémarrage applique les nouvelles directives sans perturber le fonctionnement des machines virtuelles en cours d'exécution.