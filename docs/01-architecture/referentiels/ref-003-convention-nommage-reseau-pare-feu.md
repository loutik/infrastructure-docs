---
id: 003
title: Convention de nommage réseau et pare-feu
category: Architecture
date: 2026-06-23
author: Louis MEDO
tags: [referentiel, norme, standard, convention]
---

# REF-{{ page.meta.id }} : {{ page.meta.title }}

![Bannière LoutikCLOUD](/assets/banniere_loutikcloud.png)

!!! info "Informations du Référentiel"
    * **Date d'application** : {{ page.meta.date }}
    * **Catégorie** : {{ page.meta.category }}
    * **Garant de la norme** : {{ page.meta.author }}

## 1. Objectif et périmètre
L'objectif de cette norme est de structurer et d'harmoniser le nommage au sein des équipements réseau (switchs, routeurs) et spécifiquement du pare-feu OPNsense de l'infrastructure LoutikCLOUD. Elle définit les conventions applicables aux interfaces (physiques et logiques), aux alias (hôtes, réseaux, ports), ainsi qu'aux règles et catégories de filtrage. Ce standard est indispensable pour maintenir une lisibilité optimale dans les IHM restreintes, respecter le principe KISS au quotidien, et garantir la prédictibilité des objets pour l'automatisation de la matrice de flux.

## 2. Conventions

### 2.1. Interfaces (Physiques et VLANs)
Le nommage des interfaces (`Description` / `Identifier` dans OPNsense) doit être extrêmement concis pour éviter la troncature dans les tableaux de règles de pare-feu. L'identifiant matériel (`Device`, ex: `vlan0.12`) conserve sa nomenclature système par défaut pour faciliter le débogage L2.

* **Format strict** : `[ZONE]_[FONCTION]_[ID]` 
* **Casse** : Majuscules obligatoires.

| Segment | Rôle | Valeurs autorisées |
| :--- | :--- | :--- |
| `ZONE` | Identifie la zone réseau logique ou physique | `WAN`, `LAN`, `DMZE` (DMZ Externe), `DMZI` (DMZ Interne), `ZDS` (Diffusion Restreinte), `DEV` |
| `FONCTION` | Décrit le rôle du réseau | `GW`, `BASTION`, `SVC`, `INFRA`, `DB`, `USERS` |
| `ID` | Décrit l'ID du VLAN | `12`, `17` |

### 2.2. Alias et objets Logiques
Les alias OPNsense abstraient les adresses IP et les ports. Ils doivent systématiquement typer l'objet visé pour éviter les erreurs d'assignation dans les règles.

* **Format strict** : `[TYPE]_[DESCRIPTION]`
* **Casse** : Préfixe en majuscules, description en majuscule et [snake_case](https://en.wikipedia.org/wiki/Snake_case).

| Type (`TYPE_`) | Rôle de l'objet logique | Exemples d'application |
| :--- | :--- | :--- |
| `NET_` | Réseaux ou sous-réseaux (CIDR complets) | `NET_ZDS_DATABASE`, `NET_ADM_POSTS` |
| `HOST_` | Machines spécifiques ou adresses IP uniques | `HOST_K3S_MASTER_01`, `HOST_ANSIBLE_RUNNER` |
| `PORT_` | Ports TCP/UDP individuels ou groupés | `PORT_WEB`, `PORT_KUBE_API` |

### 2.3. Catégories et règles de Filtrage
* **Catégories (Tags)** : Doivent correspondre au flux métier ou à la zone de destination (ex: `Inter-VLAN`, `Internet-Access`, `Management`).
* **Description des règles** : Doit obligatoirement contextualiser la source, la destination et l'action. Format exigé : `[Flux] Description métier`.

## 3. Exemples pratiques

### Nommage des interfaces
* ✅ **Conforme** : 
    * `DMZE_GW_10` (VLAN 10, Passerelle DMZ Externe)
    * `ZDS_DB_17` (VLAN 17, Zone de Diffusion Restreinte Base de données)
    * `LAN_USERS_19` (VLAN 19, Réseau local utilisateurs)
* ❌ **Non conforme** : 
    * `mlt1-dmzi-vlan-service-12` (Format IPAM, beaucoup trop long pour l'IHM OPNsense)
    * `vlan_12_dmz` (Casse incorrecte, ordre des segments inversé)

### Alias et objets logiques
* ✅ **Conforme** : 
    * `NET_DMZI_SERVICES` (Groupe réseau pour 10.0.12.0/24)
    * `HOST_REVERSE_PROXY` (Machine unique gérant les flux entrants)
    * `PORT_SSH_ADMIN` (Port 22 ou port SSH custom)
* ❌ **Non conforme** : 
    * `K3s_Servers` (Préfixe de type `NET_` ou `HOST_` manquant)
    * `PORTS_80_443` (La description doit indiquer le service, pas les numéros de port)

### Règles de filtrage
* ✅ **Conforme** : 
    * `[Ansible -> DMZI] Autorisation SSH pour le provisioning`
    * `[LAN -> ZDS] Accès lecture base de données PostgreSQL`
* ❌ **Non conforme** : 
    * `Allow ping` (Manque de contexte de flux et description trop vague)

## 4. Exceptions
* **Interfaces virtuelles dynamiques** : Les interfaces générées automatiquement par des plugins (VPN Tailscale `tailscale0`, interfaces WireGuard `wg0`) conservent le nommage matériel généré par le binaire, mais l'assignation logique (Nom OPNsense) doit s'adapter au format métier, par exemple `VPN_TAILSCALE`.
* **Interfaces non routées** : Les interfaces physiques dédiées exclusivement au management hors-bande (IPMI, iLO) qui ne traversent pas le plan de routage d'OPNsense peuvent conserver l'appellation d'origine du constructeur.

## 5. Validation
* L'outil interne **Nomina** est utilisé pour la génération stricte des noms et la vérification de la conformité des identifiants au sein de l'infrastructure LoutikCLOUD.
* Tout nouvel objet réseau (Interface, Alias) intégré via Terraform, Ansible ou manuellement doit passer par l'approbation du référentiel Nomina. Des audits périodiques vérifieront la correspondance entre les objets OPNsense et l'état attendu dans la documentation.