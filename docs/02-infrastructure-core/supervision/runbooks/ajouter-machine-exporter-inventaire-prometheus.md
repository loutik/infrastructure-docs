---
title: Ajouter une machine/exporter dans l'inventaire Prometheus
service: Supervision
date: 2026-08-26
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

## 1. Contexte

Supervision de l'infrastructure Loutik avec la VM supervision dans le VLAN de supervision. Ce runbook détaille la procédure pour ajouter de nouvelles cibles de métriques à collecter. À noter que cette action n'est pas encore automatisée, mais elle a vocation à l'être dans le futur.

## 2. Prérequis

* Accès au dépôt Git : `https://github.com/loutik/infrastructure-ansible`
* Outil `ansible` installé sur la machine d'exécution
* Dépendances des playbooks Ansible installées (collections et rôles)
* Accès SSH avec privilèges d'élévation sur les cibles et la machine de supervision

## 3. Procédure d'exécution

1. **Modifier les variables de groupe** :

    Activer l'exporter correspondant dans le fichier `group_vars` associé à `prometheus_grafana`. Le nom du groupe doit correspondre exactement à celui défini dans l'inventaire `host.yml`.

    ```yaml
    # Fichier : inventories/production/group_vars/prometheus_grafana.yml
    pmt_targets:
      - group: nouveau_groupe
        node_exporter: true
    ```
    
    - **`pmt_targets`** : Variable de type liste contenant des dictionnaires pour définir les cibles.
    - **`group`** : Clé désignant le nom du groupe Ansible ciblé (utilisé pour le tri Grafana).
    - **`node_exporter`** : Clé booléenne qui active le scraping système sur le port 9100.

2. **Exécuter le playbook de mise à jour** :

    Lancer le playbook Ansible `pmt-host-update` pour appliquer la nouvelle configuration.

    ```bash
    ansible-playbook -i inventories/production/host.yml playbooks/pmt-host-update.yml --tags update -J
    ```

    - **`ansible-playbook`** : Binaire permettant d'exécuter un ensemble de tâches définies dans un playbook YAML.
    - **`-i inventories/production/host.yml`** : Paramètre d'inventaire, spécifie le chemin vers le fichier listant les hôtes cibles.
    - **`playbooks/pmt-host-update.yml`** : Chemin relatif vers le fichier de playbook à exécuter.
    - **`--tags update`** : Limite l'exécution aux seules tâches et rôles portant l'étiquette `update`.
    - **`-J`** : Raccourci pour `--ask-become-pass`, force Ansible à demander le mot de passe pour l'élévation de privilèges (sudo).

## 4. Validation

Pour vérifier que la procédure a fonctionné et que le service de supervision a bien pris en compte la modification :

1. Accédez à l'interface web des cibles Prometheus via l'URL : `http://mtl1-mtr-vm-prd-01.infra.loutik.fr:9090/targets`
2. Vérifiez que les nouveaux hôtes ajoutés apparaissent dans la liste et que leur état (State) est **UP**.