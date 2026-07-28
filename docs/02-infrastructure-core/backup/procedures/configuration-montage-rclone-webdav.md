---
title: Configuration du montage Rclone WebDAV
service: Proxmox VE / Rclone
date: 2026-07-25
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
Déploiement de l'utilitaire `rclone` sur l'ensemble des nœuds du cluster Proxmox VE de LoutikCLOUD. L'objectif est de configurer un point de montage local virtuel (FUSE) ciblant le stockage WebDAV Nextcloud. Cette procédure est rédigée comme un *blueprint* déclaratif (Infrastructure as Code) afin de faciliter sa future traduction en *playbook* Ansible. Le déploiement sur chaque nœud garantit la haute disponibilité du Plan de Reprise d'Activité (PRA) et permet l'exportation des sauvegardes froides (archives `.vma.zst`) depuis n'importe quel hyperviseur.

## 2. Prérequis

* Privilèges `root` sur l'ensemble des nœuds Proxmox VE.
* Identifiants du compte de service (ex: `svc-proxmox-backup`) et mot de passe d'application généré.
* URL racine WebDAV de l'instance Nextcloud.
* Outil d'obscurcissement Rclone disponible localement pour chiffrer le mot de passe dans le fichier de configuration statique.

## 3. Déploiement

3.1. **Installation des paquets requis.** Déploiement des dépendances système nécessaires pour rclone et la gestion du système de fichiers en espace utilisateur.

  ```bash
  apt update && apt install rclone fuse3 -y
  ```

  * `rclone` : Utilitaire en ligne de commande pour synchroniser des fichiers et monter des stockages cloud.
  * `fuse3` : Module noyau et utilitaire permettant à rclone de créer un système de fichiers virtuel accessible par Proxmox.

3.2. **Obscurcissement du secret.** Génération du mot de passe chiffré pour le fichier de configuration (à exécuter sur un poste de contrôle ou directement sur le nœud pour récupérer la valeur à intégrer dans Ansible).

  ```bash
  rclone obscure "LE_MOT_DE_PASSE_APPLICATION_NEXTCLOUD"
  ```

  * `obscure` : Fonction de rclone qui chiffre le mot de passe en clair pour le rendre illisible dans le fichier de configuration `.conf`. La chaîne générée (ex: `pass = Oky...`) sera utilisée à l'étape suivante.

3.3. **Création du fichier de configuration déclaratif.** Création du fichier statique de configuration Rclone (approche IaC, remplaçant la commande interactive `rclone config`).

  ```bash
  mkdir -p /root/.config/rclone/
  sudoedit /root/.config/rclone/rclone.conf
  ```
  
  **rclone.conf :**

  ```ini
  [nextcloud-webdav]
  type = webdav
  url = https://drive.loutik.fr/remote.php/webdav/
  vendor = nextcloud
  user = svc-proxmox-backup
  pass = [MOT_DE_PASSE_OBSCURCI]
  ```

  * `[nextcloud-webdav]` : Nom du profil distant (remote) qui sera appelé par le service systemd.
  * `type`, `url`, `vendor` : Paramètres stricts définissant le protocole et le type d'implémentation serveur pour adapter les requêtes API.

3.4. **Préparation du point de montage local.** Création du répertoire cible sur le système de fichiers du nœud Proxmox.

  ```bash
  mkdir -p /mnt/nextcloud-backups
  ```

  * `/mnt/nextcloud-backups` : Chemin absolu où l'arborescence distante sera virtuellement exposée au processus de sauvegarde VZDump.

3.5. **Création du service Systemd.** Création du fichier d'unité pour automatiser et surveiller le montage du volume au démarrage du nœud.

  ```bash
  sudoedit /etc/systemd/system/rclone-nextcloud.service
  ```
  
  **rclone-nextcloud.service :**
  
  ```ini
  [Unit]
  Description=Montage Rclone WebDAV Nextcloud
  After=network-online.target

  [Service]
  Type=notify
  ExecStart=/usr/bin/rclone mount nextcloud-webdav:/proxmox-vm /mnt/nextcloud-backups \
    --vfs-cache-mode writes \
    --dir-cache-time 1m \
    --allow-other \
    --config /root/.config/rclone/rclone.conf
  ExecStop=/bin/fusermount3 -u /mnt/nextcloud-backups
  Restart=always
  RestartSec=10

  [Install]
  WantedBy=default.target
  ```

  * `Type=notify` : Indique à Systemd d'attendre que rclone signale que le montage est prêt avant de considérer le service comme démarré.
  * `--vfs-cache-mode writes` : Permet la mise en cache des flux d'écriture pour éviter les erreurs I/O avec VZDump, expédiant les blocs vers le cloud à la volée.
  * `--allow-other` : Autorise les autres utilisateurs (comme l'utilisateur de sauvegarde de Proxmox) à accéder au point de montage FUSE créé par root.

3.6. **Activation et démarrage du service.** Prise en compte du nouveau fichier d'unité et lancement du montage.

  ```bash
  systemctl daemon-reload
  systemctl enable --now rclone-nextcloud.service
  ```

  * `daemon-reload` : Recharge la configuration du gestionnaire de services système.
  * `enable --now` : Active le service au démarrage (enable) et le lance immédiatement (now).

3.7. **Déclaration du stockage dans le cluster Proxmox.** Ajout du répertoire monté comme cible de sauvegarde de type "Directory" via la CLI Proxmox (à exécuter sur un seul nœud, la configuration étant répliquée sur le cluster).

  ```bash
  pvesm add dir nc-storage --path /mnt/nextcloud-backups --content backup --is_mountpoint 1 --shared 1
  ```
  
  * `nc-storage` : Nom du datatstore pour le stockage des vm à froid. (nc = NextCloud)
  * `pvesm add dir` : Commande d'ajout d'un stockage de type répertoire.
  * `--content backup` : Restreint l'usage de ce stockage aux seules archives de sauvegarde (archives `.vma`, fichiers `.log`).
  * `--is_mountpoint 1` : Paramètre de sécurité fondamental (Pre-flight check). Si le montage `rclone` échoue (réseau coupé), Proxmox refusera d'écrire dans le dossier vide, évitant ainsi la saturation du disque système `rootfs`.
  * `--shared 1` : Indique au cluster que ce stockage est partagé (puisqu'il pointera vers le même *backend* Nextcloud depuis chaque nœud).

---

## Annexe

- [Documentation Officielle Rclone WebDAV](https://rclone.org/webdav/)
- [Documentation Proxmox VE - Directory Storage](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#storage_directory)