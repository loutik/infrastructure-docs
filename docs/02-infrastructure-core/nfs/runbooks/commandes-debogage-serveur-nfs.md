---
title: Commandes de débogage - Serveur NFS
service: Stockage / NFS
date: 2026-08-05
author: Louis MEDO
owner: Louis MEDO
---

# {{ page.meta.title }}

![Bannière LoutikCloud](https://raw.githubusercontent.com/loutik/design-assets/main/loutikcloud/banniere_loutikcloud.png)

!!! info "Informations"
    * **Date de mise à jour** : {{ page.meta.date }}
    * **Service ciblé** : {{ page.meta.service }}
    * **Auteur** : {{ page.meta.author }}
    * **Responsable** : {{ page.meta.owner }}

!!! warning "Avertissement"
    La configuration et l'installation de cette infrastructure sont gérées intégralement par Ansible. Les commandes présentées ci-dessous ont un but exclusif de débogage. Aucune création ou modification manuelle ne doit être effectuée dans des conditions de production.

## 1. Contexte

Ce runbook documente les commandes essentielles pour le diagnostic, l'analyse et la maintenance d'un serveur NFS. Il doit être exécuté lors d'une investigation technique (ex: inaccessibilité des partages, problèmes de montage côté client, audit des performances) sans altérer la configuration déployée.

## 2. Prérequis

* Accès SSH au serveur de stockage avec privilèges `sudo`.
* Outils de diagnostic réseau et NFS installés (`nfs-common`, `nfs-kernel-server` ou `rpcbind`).

## 3. Procédure d'exécution

1. **Vérifier les partages actifs** :

    Affiche la liste détaillée des répertoires exportés et les options réelles appliquées en mémoire.

    ```bash
    sudo exportfs -v
    ```

    * **`sudo`** : Exécute la commande avec les privilèges superutilisateur.
    * **`exportfs`** : Gère la table des systèmes de fichiers exportés.
    * **`-v`** : Mode verbeux, détaille de manière exhaustive les options actives (ex: rw, sync, root_squash).

    *Exemple de retour :*

    ```text
    /srv/nfs_share  192.168.1.0/24(rw,wdelay,root_squash,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
    ```

2. **Consulter les statistiques NFS** :

    Récupère les compteurs d'appels RPC pour analyser les performances ou identifier des erreurs de requêtes.

    ```bash
    nfsstat -s
    ```

    * **`nfsstat`** : Affiche les statistiques globales de fonctionnement NFS.
    * **`-s`** : Restreint la sortie aux statistiques spécifiques du serveur (Server).

    *Exemple de retour :*

    ```text
    Server rpc stats:
    calls      badcalls   badclnt    badauth    xdrcall
    142351     0          0          0          0
    ```

3. **Lister les services RPC** :

    Vérifie les ports et les versions des services RPC (dont NFS) actuellement en écoute sur le serveur.

    ```bash
    rpcinfo -p 127.0.0.1 | grep nfs

    ```

    * **`rpcinfo -p`** : Liste tous les programmes RPC enregistrés sur la cible.
    * **`127.0.0.1`** : Cible l'interface locale (localhost).
    * **`| grep nfs`** : Filtre le flux de sortie pour n'isoler que les processus liés à NFS.

    *Exemple de retour :*

    ```text
        100003    3   tcp   2049  nfs
        100003    4   tcp   2049  nfs
        100227    3   tcp   2049  nfs_acl
    ```

4. **Lire la table d'exportation brute** :

    Consulte la configuration exacte des exports telle qu'elle est interprétée par le noyau Linux.

    ```bash
    cat /var/lib/nfs/etab
    ```

    * **`cat`** : Lit et affiche le contenu d'un fichier dans le terminal.
    * **`/var/lib/nfs/etab`** : Fichier système contenant la table compilée et active des exports NFS.

    *Exemple de retour :*

    ```text
    /srv/nfs_share   192.168.1.0/24(rw,sync,wdelay,hide,nocrossmnt,secure,root_squash,no_all_squash,no_subtree_check,secure_locks,acl,no_pnfs,anonuid=65534,anongid=65534,sec=sys,secure,root_squash,no_all_squash)
    ```

5. **Vérifier les points de montage annoncés** :

    Teste quels partages sont publiquement annoncés comme disponibles par le serveur aux clients.

    ```bash
    showmount -e localhost
    ```

    * **`showmount`** : Interroge le démon de montage du serveur NFS.
    * **`-e`** : (exports) Liste les systèmes de fichiers que le serveur est configuré pour exporter.
    * **`localhost`** : Interroge le serveur local.

    *Exemple de retour :*

    ```text
    Export list for localhost:
    /srv/nfs_share 192.168.1.0/24
    ```

6. **Vérifier les performances du partage NFS** :

    Évalue la vitesse d'écriture brute sur le point de montage NFS pour détecter d'éventuelles latences réseau ou disques. (À exécuter depuis un client ou directement sur le stockage si monté localement).

    ```bash
    dd if=/dev/zero of=/mnt/nfs_share/test_perf.img bs=1G count=1 oflag=direct
    ```

    * **`dd`** : Utilitaire de copie de bas niveau pour convertir et formater des fichiers.
    * **`if=/dev/zero`** : (Input File) Source générant un flux continu de zéros (données vides).
    * **`of=/mnt/nfs_share/test_perf.img`** : (Output File) Fichier de destination sur le point de montage NFS.
    * **`bs=1G`** : (Block Size) Définit la taille d'un bloc de données à 1 Gigaoctet.
    * **`count=1`** : Définit le nombre de blocs à copier (ici, 1 bloc de 1 Go).
    * **`oflag=direct`** : Contourne le cache mémoire (RAM) du système d'exploitation pour forcer l'écriture directe et mesurer la performance réelle du réseau/disque.

    *Exemple de retour :*

    ```text
    1+0 records in
    1+0 records out
    1073741824 bytes (1.1 GB, 1.0 GiB) copied, 9.4532 s, 114 MB/s
    ```