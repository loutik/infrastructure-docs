---
id: REF-001
title: Convention de nommage infrastructure
category: Architecture
date: 2026-05-18
author: Louis MEDO
tags: [convention]
---

# {{ page.meta.id }} : {{ page.meta.title }}

![Bannière LoutikCLOUD](/assets/banniere_loutikcloud.png)

!!! info "Informations"
    * **Date d'application** : {{ page.meta.date }}
    * **Catégorie** : {{ page.meta.category }}
    * **Garant de la norme** : {{ page.meta.author }}

## 1. Objectif et périmètre
L'objectif de cette norme est d'harmoniser l'identification de l'ensemble des ressources physiques et virtuelles de l'infrastructure LoutikCLOUD. Ce standard garantit la prédictibilité des noms d'hôtes pour permettre une automatisation robuste via l'API Netbox et les playbooks Ansible (parsing via `split('-')`).

Cette norme s'applique de manière stricte aux : sites physiques, baies de brassage, équipements réseau (switchs, routeurs, firewalls), serveurs bare-metal, machines virtuelles (On-Premise et Cloud), et ressources logiques (VLANs, IPAM). Elle s'applique également aux enregistrements DNS d'infrastructure (A/AAAA).

## 2. Conventions
La structure globale repose sur un format strict composé de **5 segments** séparés par des tirets. L'utilisation de majuscules est proscrite pour les noms d'hôtes et le DNS.

* **Format d'expression régulière (Regex)** : `^[a-z0-9]+-[a-z0-9]+-[a-z0-9]+-(prd|stg|dev|lab)-\d{2}$`
* **Structure globale** : `<loc>-<rol>-<typ>-<env>-<id>`

| Segment | Rôle | Valeurs autorisées |
| :--- | :--- | :--- |
| **`loc`** (Localisation) | Identifie l'emplacement physique ou le cloud provider. | **On-Premise :** `trs1` (Tours), `mtl1` (Montlouis).<br>**Cloud :** `ovhgra1` (OVH Gravelines), `infgva1` (Infomaniak Genève), `awspar1` (AWS Paris).<br>**Global :** `glb` (Ressources transverses/logiques). |
| **`rol`** (Rôle) | Définit la fonction de l'équipement ou du service. | `k3s`, `pve` (Proxmox), `opn` (OPNsense), `core` (Routage central), `edge` (Bordure), `web`, `db`, `runner`. |
| **`typ`** (Type) | Précise la nature matérielle ou logique. | `bm` (Bare-Metal), `vm` (Virtual Machine), `sw` (Switch), `fw` (Firewall), `rt` (Routeur), `vlan`, `ipam`. |
| **`env`** (Environnement) | Délimite le contexte d'exécution (Trigramme strict). | `prd` (Production), `stg` (Staging), `dev` (Développement), `lab` (Laboratoire). |
| **`id`** (Identifiant) | Incrément numérique unique sur 2 chiffres. | De `01` à `99`. |

!!! note "Enregistrements DNS d'Infrastructure"
    Le FQDN d'une machine doit correspondre **exactement** à son nom Netbox complet. Le format est `<loc>-<rol>-<typ>-<env>-<id>.infra.loutikcloud.fr`. Des CNAME applicatifs (plus courts) pourront pointer vers ces enregistrements de base.

## 3. Exemples pratiques

* ✅ **Conforme** : 
    * `trs1-pve-bm-prd-01` (Serveur physique hyperviseur Proxmox, Production, Tours)
    * `mtl1-opn-fw-lab-01` (Firewall OPNsense physique, Laboratoire, Montlouis)
    * `infgva1-k3s-vm-prd-02` (VM nœud K3s, Production, Infomaniak Genève)
    * `trs1-vlan-mgt-10` (VLAN logique Management, ID 10, Tours - Adaptation du format logique)
* ❌ **Non conforme** : 
    * `TRS1-pve-bm-prd-01` (Invalide : Contient des majuscules).
    * `k3s-m-prod-01` (Invalide : Il manque le segment `loc`, `prod` au lieu de `prd`, et `m` n'est pas un `typ` valide).
    * `ovhgra1_db_vm_dev_01` (Invalide : Utilise des underscores au lieu de tirets).

## 4. Exceptions

* **Ressources logiques pures :** Le champ `env` peut être omis ou remplacé par une valeur métier uniquement pour les VLANs (ex: `trs1-vlan-mgt-10` où `10` remplace `id` et `mgt` remplace `env`).

* **Objets d'organisation Netbox :** Les noms des objets "Sites" et "Racks" dans Netbox (qui ne sont pas des noms d'hôtes DNS) peuvent conserver des majuscules pour la lisibilité humaine (ex: `DC-TRS1`, `TRS1-RK01`).

* **Clusters :** Les noms des clusters sont allégés pour répondre à la problématique de la taille définie par certains orchestrateurs et hyperviseurs. Ils doivent donc utiliser le format suivant : `<loc>-<loc>-<env>` (ex: `trs1-pve-prd`, `mlt1-pve-prd`, `ovhgra1-pve-prd`).

## 5. Validation

Le respect de cette convention est assuré par l'automatisation :

* Validation syntaxique via un script Python exécuté dans la pipeline CI/CD de GitHub Actions lors des requêtes de provisionnement.
* L'utilisation de la commande Ansible `inventory_hostname.split('-')` échouera délibérément lors de l'exécution des playbooks si la matrice de 5 segments n'est pas respectée, bloquant le déploiement.