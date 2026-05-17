# 🚀 Edge

![Bannière LoutikCLOUD](../assets/banniere_loutikcloud.png)

## 📝 Vue d'ensemble
Cette section documente la couche de périphérie exposée aux réseaux publics. Elle gère le point d'entrée unique de l'infrastructure, assurant le routage des flux entrants, la mitigation des menaces et l'accès distant sécurisé.

## 🎯 Périmètre et objectifs
* **Reverse Proxy** : Assurer la terminaison TLS et le routage applicatif des requêtes entrantes vers les services internes appropriés.
* **Security** : Protéger les points de terminaison via la détection d'intrusions, le pare-feu applicatif (WAF) et le contrôle d'accès.
* **VPN** : Établir des tunnels chiffrés et authentifiés pour l'administration sécurisée de l'infrastructure depuis l'extérieur.

## 🗺️ Navigation
* **[Reverse Proxy](./reverse-proxy)** - Configurations de routage HTTP/HTTPS, gestion des certificats et règles d'exposition des services.
* **[Security](./security)** - Politiques de filtrage, intégrations CrowdSec, et mécanismes de protection des accès.
* **[VPN](./vpn)** - Déploiement des tunnels distants, gestion des clés cryptographiques et règles de routage client-à-site.