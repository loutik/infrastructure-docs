---
title: Choix de la solution de sauvegarde
date: 22/07/2026
status: accepted
author: Louis MEDO
owner: Louis MEDO
tags: [adr, architecture]
---

# ADR : {{ page.meta.title }}

![Bannière LoutikCloud](https://raw.githubusercontent.com/loutik/design-assets/main/loutikcloud/banniere_loutikcloud.png)

!!! info "Informations"
    * **Date de création** : {{ page.meta.date }}
    * **Statut** : {{ page.meta.status }}
    * **Auteur** : {{ page.meta.author }}
    * **Responsable** : {{ page.meta.owner }}

## 1. Contexte

L'infrastructure actuelle, hébergée sur un cluster de trois nœuds Proxmox VE (dont un nœud de 1To dédié aux sauvegardes et deux nœuds de 500Go), nécessite une stratégie de sauvegarde centralisée, homogène et respectant le principe KISS. L'écosystème comprend des machines virtuelles complètes, des bases de données transactionnelles (PostgreSQL, MongoDB), un serveur de fichiers NFS, ainsi qu'un pare-feu physique OPNsense en bordure de réseau.

Le besoin métier exige d'automatiser les cycles de vie des sauvegardes avec une rétention stricte de 7 jours, de garantir la cohérence des données lors des exports de bases de données, et de permettre une externalisation vers un stockage Cloud tout en respectant une contrainte budgétaire stricte (inférieure à 2€/mois) ou en exploitant les quotas étudiants disponibles (WebDAV 200Go). L'objectif est d'éviter la multiplication des outils de sauvegarde tout en couvrant des périmètres hétérogènes (hyperviseur, fichiers, configurations réseau).

## 2. Cahiers des charges

| ID | Type | Exigence | Description |
| :--- | :--- | :--- | :--- |
| **REQ-F01** | Fonctionnel | Sauvegarde unifiée | La solution doit pouvoir traiter à la fois des images disques de VM (à chaud) et des arborescences de fichiers de manière centralisée. |
| **REQ-F02** | Fonctionnel | Cycle de vie automatisé | Capacité à planifier les sauvegardes (ex: 00h00) et à appliquer automatiquement une politique de rétention (suppression après 7 jours). |
| **REQ-T01** | Technique | Optimisation du stockage | Nécessité d'employer la déduplication et des méthodes incrémentales pour limiter l'empreinte sur le nœud de 1To. |
| **REQ-T02** | Technique | Cohérence des bases de données | L'outil doit s'intégrer post-export logique (dump) pour ne pas corrompre les données en cours d'écriture (PostgreSQL/MongoDB). |
| **REQ-T03** | Technique | Extensibilité et Cloud | Capacité à exporter les archives de manière sécurisée (chiffrement) vers une cible distante (S3 ou WebDAV). |

## 3. Les solutions du marché

### 3.1. Présentations des solutions

#### 3.1.1. Proxmox Backup Server (PBS)
* **Présentation générale :** Solution open-source native développée par l'éditeur de Proxmox VE, conçue spécifiquement pour la sauvegarde d'infrastructures d'entreprise.
* **Fonctionnement :** Utilise les *QEMU dirty bitmaps* pour des sauvegardes incrémentales de VM ultra-rapides, et propose un agent CLI (`proxmox-backup-client`) pour créer des archives de fichiers dédupliquées (`.pxar`).
* **Profil :** Infrastructures virtualisées sous Proxmox cherchant une intégration sans friction (agentless pour les VM) et une déduplication agressive.

#### 3.1.2. Restic
* **Présentation générale :** Outil en ligne de commande (CLI) open-source, rapide, spécialisé dans la sauvegarde de fichiers de manière sécurisée.
* **Fonctionnement :** Réalise des sauvegardes incrémentales au niveau fichier, avec déduplication et chiffrement robustes par défaut, vers de multiples backends (Local, S3, WebDAV, SFTP).
* **Profil :** Administrateurs système et ingénieurs DevOps cherchant un outil léger, portable et scriptable pour sécuriser des données applicatives et des fichiers.

#### 3.1.3. Kopia
* **Présentation générale :** Solution de sauvegarde moderne open-source combinant une interface graphique (GUI) et une interface en ligne de commande (CLI).
* **Fonctionnement :** S'appuie sur la création de snapshots de répertoires avec déduplication, compression et chiffrement de bout en bout avant l'envoi vers un dépôt distant.
* **Profil :** Utilisateurs nécessitant la flexibilité d'une interface visuelle tout en sauvegardant des fichiers vers diverses solutions de stockage cloud.

### 3.2. Comparatifs des solutions

| Exigence | Proxmox Backup Server (PBS) | Restic | Kopia |
| :--- | :--- | :--- | :--- |
| **REQ-F01 (Unifiée)** | Validé (VMs natives + CLI Fichiers) | Non validé (Fichiers uniquement, pas de VM natives) | Non validé (Fichiers uniquement) |
| **REQ-F02 (Cycle de vie)** | Validé (Gestion native via Datastore) | Validé (Via cron et flags CLI `--keep-last`) | Validé (Politiques configurables via UI/CLI) |
| **REQ-T01 (Optimisation)** | Validé (Déduplication globale excellente) | Validé (Déduplication forte) | Validé (Déduplication et compression) |
| **REQ-T02 (Cohérence DB)** | Validé (Sauvegarde les dossiers de dump) | Validé (Sauvegarde les dossiers de dump) | Validé (Sauvegarde les dossiers de dump) |
| **REQ-T03 (Cloud)** | Évaluation (Nécessite PBS Sync ou outil tiers pour S3/WebDAV) | Validé (Support natif de multiples backends cloud) | Validé (Support natif de multiples backends cloud) |

## 4. Solution proposée

La solution proposée pour l'infrastructure est **Proxmox Backup Server (PBS)** complété par une orchestration **Ansible** pour les équipements non gérés nativement.

**Intégration architecturale :**

PBS sera déployé directement sur le nœud disposant d'une capacité de 1To. L'architecture se décline en trois axes :

1. **Machines Virtuelles :** Proxmox VE est connecté nativement au PBS. Les sauvegardes s'exécutent en mode *agentless* via l'hyperviseur, exploitant les QEMU dirty bitmaps pour ne transférer que les blocs modifiés depuis la veille.
2. **Fichiers et Bases de données (PostgreSQL/MongoDB) :** Un processus chronométré génère des dumps SQL locaux. L'utilitaire `proxmox-backup-client` installé sur la VM NFS prend ensuite le relais pour envoyer le contenu du serveur de fichiers (incluant les dumps) vers PBS sous forme d'archives compressées et dédupliquées.
3. **Pare-feu OPNsense :** Une VM orchestrateur exécute un playbook Ansible qui requiert l'API d'OPNsense pour télécharger le `config.xml`. Le fichier est ensuite déposé de manière sécurisée (via rebond SSH / ProxyJump) sur le serveur NFS dans un répertoire restreint, avant d'être avalé par la sauvegarde globale du client PBS.

Cette architecture centralise 100% des données au sein d'un seul coffre-fort (datastore) PBS, avec une politique de rétention globale fixée à 7 jours. L'externalisation (vers l'offre étudiante WebDAV ou un bucket S3 économique) sera gérée par une tâche de synchronisation du datastore sous-jacent.

**Justification du rejet des solutions alternatives :**

* **Restic :** Bien que performant pour la gestion des fichiers isolés et du transfert direct vers le Cloud, Restic ne permet pas de sauvegarder l'état complet et à chaud des machines virtuelles (images disques) de manière efficiente. Utiliser Restic obligerait à empiler une solution tierce pour les VM, ce qui viole la contrainte de centralisation et augmente la complexité opérationnelle.
* **Kopia :** Kopia souffre de la même limitation fondamentale que Restic (incapacité à interagir avec l'hyperviseur pour des snapshots de niveau bloc). De plus, l'ajout d'une interface graphique pour la simple sauvegarde de fichiers est considéré comme de la sur-ingénierie dans un contexte où l'infrastructure privilégie les CLI et l'Infrastructure as Code (IaC).