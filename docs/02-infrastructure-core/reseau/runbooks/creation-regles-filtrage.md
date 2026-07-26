---
title: Création des règles de filtrage
service: OPNsense
date: 2026-06-30
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
Cette procédure décrit l'écriture et l'application des règles de sécurité au sein de la nouvelle interface de filtrage d'OPNsense (`Rules [new]`). Ce framework moderne basé sur l'API MVC structure la matrice de flux de manière granulaire. L'objectif est d'implémenter une politique de sécurité de type *Zero-Trust* inter-VLAN en exploitant les alias et catégories configurés précédemment. Cette approche garantit la compatibilité directe avec le code des playbooks Ansible.

## 2. Prérequis

* La procédure de création des VLANs et interfaces ([Création des VLANs et interfaces](./creation-vlan-opnsense.md)) est validée.
* La procédure de déclaration des alias et catégories ([Création des objets logiques et catégories de filtrage](./creation-objets-logiques-et-categories-filtrage.md)) est finalisée.
* Consultation du référentiel **REF-003 ([Convention de nommage réseau et pare-feu](../../../01-architecture/referentiels/ref-003-convention-nommage-reseau-pare-feu.md))**, Section 2.3 pour la nomenclature obligatoire des descriptions.

## 3. Création d'une règle de filtrage dans l'interface moderne

3.1. **Ajout d'une règle unitaire.** Depuis l'interface Web, naviguez dans `Firewall > Rules [new]` et cliquez sur le bouton "+" rouge en bas à droite pour ouvrir la fenêtre de dialogue "Edit rule".

  ```bash
  Enabled : Coché
  Categories : Management
  Description : [DMZI -> Management] Autorisation flux SSH vers le reverse proxy
  Interface : DMZI_SVC_12
  Quick : Coché
  Action : Pass
  Direction : In
  Version : IPv4
  Protocol : TCP
  Source : NET_DMZI_SERVICES
  Source Port : any
  Destination : HOST_REVERSE_PROXY
  Destination Port : 22
  Gateway : None
  ```

  `Enabled` : Active la règle de filtrage.

  `Categories` : Association à l'étiquette organisationnelle (ex: Management, Inter-VLAN).
  
  `Description` : Contextualisation humaine respectant scrupuleusement le format strict [Flux] Description métier du REF-003, Section 2.3 ([Convention de nommage réseau et pare-feu](../../../01-architecture/referentiels/ref-003-convention-nommage-reseau-pare-feu.md)).

  `Interface` : Segment réseau physique ou logique (VLAN) sur lequel s'applique le contrôle d'accès.

  `Quick` : Si coché, applique immédiatement la règle dès qu'un paquet correspond, sans évaluer les règles suivantes.

  `Action` : Détermine le traitement du flux (Pass pour autoriser, Drop pour rejeter silencieusement)[cite: 8].

  `Direction` : Sens du trafic par rapport à l'interface (In pour intercepter le trafic entrant dans le pare-feu depuis un segment)[cite: 8].

  `Version` : Sélectionne la version du protocole IP (IPv4 ou IPv6).

  `Protocol` : Sélectionne le protocole de transport ciblé (ex: TCP, UDP, ICMP).

  `Source / Destination` : Sélection stricte des alias de réseaux (NET_) ou d'hôtes (HOST_) définis selon la norme Nomina.
  
  `Source Port / Destination Port` : Spécification des ports sources ou des alias de services (PORT_) cibles.
  
  `Gateway` : Permet de forcer un routage spécifique via une passerelle de sortie (Laisser à None par défaut).

## 4. Application et validation de la politique

4.1. **Rechargement des configurations en mémoire.** Pour appliquer les modifications et injecter les règles au sein du moteur de filtrage `pf`, appliquez les changements.

  ```bash
  Action : Clic sur le bouton "Apply Changes"
  ```
    
  `Apply Changes` : Valide et recharge la politique de sécurité globale du pare-feu.

## Annexe

- [REF-003 - Convention de nommage réseau et pare-feu](../../../01-architecture/referentiels/ref-003-convention-nommage-reseau-pare-feu.md)