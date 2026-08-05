---
title: Fiche recette - Serveur NFS
service: Stockage / NFS
date: 2026-08-04
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
Ce composant agit en tant que backend de stockage persistant pour les applications de l'infrastructure LoutikCLOUD. Il s'intègre en fournissant des espaces isolés par service, où chaque dossier est verrouillé par un couple UID/GID dédié. Les flux réseaux sont restreints aux adresses IP spécifiques des clients déclarés, et la sécurité est renforcée par l'utilisation exclusive du protocole NFSv4.

## 2. Prérequis

* Serveur NFS déployé et configuré via le rôle Ansible `debian-nfs`.
* Une machine cliente disposant du paquet `nfs-common` et dont l'IP est autorisée dans la configuration (ex: 10.0.0.10).
* Une machine tierce (IP non déclarée) pour valider l'isolation réseau.

## 3. Validation du fonctionnement et de la sécurité

3.1. **Vérification des versions actives sur le serveur.** S'assurer que le serveur n'expose que le protocole sécurisé NFSv4 et que les anciennes versions (v2/v3) sont bien inactives.

  ```bash
  rpcinfo -p 127.0.0.1 | grep nfs
  ```

  * `rpcinfo -p` : Affiche les programmes RPC enregistrés sur le serveur. La sortie ne doit mentionner que la version 4.
  * `grep nfs` : Filtre la sortie pour isoler le composant serveur NFS.

  **Retour attendu :**

  ````bash
  100003    4   tcp   2049  nfs
  ````

3.2. **Test de montage depuis un client autorisé.** Valider que le client disposant de l'IP déclarée peut accéder au partage réseau et le monter localement.

  ```bash
  sudo mount -t nfs4 10.0.0.5:/srv/nfs/web /mnt/test_nfs
  ```

  * `-t nfs4` : Force le client à utiliser le protocole NFS version 4 pour initier la connexion.
  * `10.0.0.5:/srv/nfs/web` : IP du serveur NFS suivie du chemin absolu exporté.

  **Retour attendu :**

  ````text
  Aucun message (retour sans erreur)
  ````

3.3. **Validation du mappage des droits (UID/GID).** Confirmer que l'option `all_squash` attribue correctement l'identité du compte de service (ex: 2001) aux nouveaux fichiers créés.

  ```bash
  touch /mnt/test_nfs/test.txt && ls -ln /mnt/test_nfs/test.txt
  ```

  * `touch` : Tente de créer un fichier vide sur le montage NFS avec l'utilisateur du client.
  * `ls -ln` : Affiche les permissions avec les identifiants numériques (UID et GID doivent correspondre à 2001, indépendamment de l'utilisateur ayant exécuté le `touch`).

  **Retour attendu :**

  ````bash
  -rw-rw-r-- 1 2001 2001 0 Aug  4 15:53 /mnt/test_nfs/test.txt
  ````

3.4. **Test de rejet d'une adresse IP non autorisée.** Tenter d'accéder au partage réseau depuis une machine dont l'adresse IP ne figure pas dans le fichier `/etc/exports`.

  ```bash
  sudo mount -t nfs4 10.0.0.5:/srv/nfs/web /mnt/test_nfs
  ```

  * `mount` : La commande d'exécution. Elle doit se solder par une erreur bloquante : `access denied by server while mounting`.
  * `10.0.0.5:/srv/nfs/web` : La cible du montage que le pare-feu logiciel NFS refusera, confirmant la sécurité périmétrique.