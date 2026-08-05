---
title: Installation et configuration manuelle de NFS
service: Stockage / NFS
date: 2026-08-05
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

!!! warning "Avertissement"
    La configuration et l'installation de cette infrastructure sont gérées intégralement par Ansible. Les commandes présentées ci-dessous illustrent le processus manuel équivalent à des fins de compréhension ou de débogage. Aucune création ou modification manuelle ne doit être effectuée dans des conditions de production.

## 1. Contexte

Déploiement du composant serveur NFS fournissant les volumes de stockage partagés. Ce service s'intègre à l'infrastructure LoutikCLOUD en exposant des espaces persistants pour les nœuds Proxmox et les clusters Kubernetes. La configuration isole chaque partage avec des utilisateurs dédiés et restreint le protocole à la version 4.

## 2. Prérequis

* VM Debian/Ubuntu allouée et accessible en SSH avec privilèges `sudo`.
* Espace de stockage (LVM ou partition) monté et disponible.
* Variables de déploiement identifiées (chemins, UIDs, GIDs, IPs autorisées).

## 3. Procédure de déploiement

3.1. **Installation du serveur NFS.** Installation des paquets requis pour le service.

  ```bash
  sudo apt-get update && sudo apt-get install -y nfs-kernel-server
  ```

  * `-y` : Valide automatiquement les demandes de confirmation d'installation.
  * `nfs-kernel-server` : Paquet fournissant le démon serveur NFS natif du noyau Linux.

3.2. **Création du répertoire de surcharge.** Préparation de l'arborescence pour la configuration modulaire.

  ```bash
  sudo mkdir -p /etc/nfs.conf.d && sudo chmod 0755 /etc/nfs.conf.d
  ```

  * `mkdir -p` : Crée le répertoire cible ainsi que les répertoires parents manquants.
  * `chmod 0755` : Attribue les droits de lecture/exécution pour tous, et d'écriture uniquement au propriétaire.

3.3. **Restriction du protocole à NFSv4.** Désactivation des anciennes versions.

  ```bash
  echo -e "[nfsd]\nvers2=n\nvers3=n\nvers4=y" | sudo tee /etc/nfs.conf.d/10-nfsv4-only.conf
  ```

  * `echo -e` : Interprète les caractères d'échappement (comme `\n` pour les sauts de ligne) pour générer le bloc de configuration.
  * `tee` : Lit l'entrée standard et l'écrit dans un fichier avec élévation de privilèges.

3.4. **Création des groupes d'isolation système.** Mise en place d'un groupe dédié par partage.

  ```bash
  sudo groupadd --system --gid 2000 sharegroup
  ```

  * `--system` : Crée un groupe système (GID généralement < 1000, n'apparaissant pas dans les interfaces de connexion).
  * `--gid 2000` : Assigne manuellement un identifiant de groupe statique pour garantir la cohérence inter-serveurs.

3.5. **Création des utilisateurs d'isolation.** Génération d'un compte de service restreint associé au groupe.

  ```bash
  sudo useradd --system --uid 2000 --gid sharegroup --no-create-home --shell /usr/sbin/nologin shareuser
  ```

  * `--no-create-home` : Empêche la création d'un répertoire personnel standard dans `/home`.
  * `--shell /usr/sbin/nologin` : Bloque toute tentative de connexion interactive pour des raisons de sécurité.

3.6. **Création et sécurisation de l'espace de stockage.** Initialisation du répertoire de partage avec les bons droits.

  ```bash
  sudo mkdir -p /srv/nfs/share && sudo chown 2000:2000 /srv/nfs/share && sudo chmod 0750 /srv/nfs/share
  ```

  * `chown 2000:2000` : Attribue la propriété du dossier à l'UID et au GID de service créés précédemment.
  * `chmod 0750` : Autorise la lecture/écriture/exécution pour le propriétaire, lecture/exécution pour le groupe, aucun droit pour les autres.

3.7. **Génération et application des exports.** Déclaration du partage et redémarrage du service.

  ```bash
  echo "/srv/nfs/share 192.168.1.0/24(rw,sync,root_squash,all_squash,anonuid=2000,anongid=2000)" | sudo tee -a /etc/exports
  sudo exportfs -ra
  sudo systemctl restart nfs-kernel-server
  ```

  * `tee -a` : Ajoute (append) la nouvelle ligne à la fin du fichier `/etc/exports` existant.
  * `exportfs -ra` : Recharge entièrement la table des exports NFS en mémoire à partir du fichier de configuration.
  * `systemctl restart` : Redémarre complètement le démon système pour appliquer les surcharges NFSv4.

## 4. Annexe

* [Stéphane Robert - Utiliser NFS sur Linux](https://blog.stephane-robert.info/docs/services/stockage/nfs/)