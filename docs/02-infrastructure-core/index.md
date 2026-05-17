# 🚀 Infrastructure Core

![Bannière LoutikCLOUD](../assets/banniere_loutikcloud.png)

## 📝 Vue d'ensemble
Cette section documente les fondations matérielles et logiques de l'environnement, englobant la virtualisation, l'orchestration des conteneurs et la topologie réseau. Elle constitue le socle garantissant la haute disponibilité, l'isolation et l'exécution sécurisée de l'ensemble des services hébergés.

## 🎯 Périmètre et objectifs
* **Hyperviseur** : Fournir, allouer et gérer les ressources de calcul virtualisées (Proxmox VE) en assurant la résilience et l'optimisation matérielle.
* **Orchestrateur** : Automatiser le déploiement, la mise à l'échelle et le maintien en condition opérationnelle des charges de travail conteneurisées (K3s).
* **Réseau** : Assurer le routage interne/externe, la segmentation stricte des flux et la sécurisation du périmètre de l'infrastructure (OPNsense).

## 🗺️ Navigation
* **[Hyperviseur](./hyperviseur)** - Documentation des nœuds de virtualisation, templates de machines virtuelles/LXC et gestion du stockage.
* **[Orchestrateur](./orchestrateur)** - Architecture du cluster, gestion des nœuds (control-plane/workers) et standards de déploiement des pods.
* **[Réseau](./reseau)** - Cartographie de la topologie, règles de filtrage du pare-feu, configurations des VLANs et politiques de routage.