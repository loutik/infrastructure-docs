# Infrastructure - Documentation

![Bannière Loutik](https://raw.githubusercontent.com/loutik/design-assets/main/banniere_loutik.png)

## Contexte

Ce dépôt centralise l'ensemble des procédures, architectures et documentations techniques du projet Loutik. L'objectif est de maintenir une base de connaissances fiable, versionnée et structurée via MkDocs pour assurer la pérennité, la compréhension et la bonne gestion des services de l'infrastructure LoutikCLOUD.

---

## Structure du dépôt

L’organisation du dépôt suit la logique suivante :

```text
.
├── .github/
├── docs/
│   ├── 01-architecture/
│   ├── 02-infrastructure-core/
│   ├── 03-edge/
│   ├── 04-securite/
│   ├── 05-services/
│   ├── assets/
│   ├── favicon.ico
│   └── index.md
├── overrides/
├── template/
├── .gitignore
├── mkdocs.yml
└── requirements.txt
```

* **`docs/`** : Contient l'ensemble des fichiers Markdown de la documentation, segmentés par domaine technique logique.
* **`.github/`** : Héberge les configurations spécifiques à GitHub, notamment les templates de Pull Request et les workflows CI/CD.
* **`mkdocs.yml`** : Fichier de configuration principal qui définit la structure du site, le thème et les plugins MkDocs.
* **`requirements.txt`** : Liste les paquets Python requis pour générer et faire tourner le site de documentation localement.

---

## Utilisation de MkDocs

### 1. Cloner le dépôt localement

La commande `git clone` permet de télécharger une copie locale du dépôt distant, tandis que `cd` permet de naviguer dans le dossier nouvellement créé.

```bash
git clone https://github.com/loutik/infrastructure-docs.git
cd infrastructure-docs
```

### 2. Initialiser et charger l'environnement virtuel

Afin d'isoler les dépendances du projet du reste du système, nous utilisons un environnement virtuel Python. La commande `python -m venv` le crée, et la commande `source` l'active pour la session courante du terminal.

```bash
python -m venv venv
source venv/bin/activate
```

*(Note : Sous Windows, la commande d'activation est `.\venv\Scripts\activate`)*

### 3. Installer les dépendances

La commande `pip install` lit le fichier de configuration et télécharge les paquets nécessaires (comme MkDocs et le thème Material) dans votre environnement virtuel.

```bash
pip install -r requirements.txt
```

### 4. Lancer le serveur de développement

La commande `mkdocs serve` compile les fichiers Markdown en HTML et lance un serveur web local. Ce serveur intègre un rechargement à chaud (hot-reload) : toute modification sauvegardée dans vos fichiers Markdown sera immédiatement visible dans le navigateur.

```bash
mkdocs serve
```

---

## Bonnes pratiques et sécurité

1. **Création d'une branche dédiée** : Ne travaillez jamais directement sur la branche `main`. Pour toute nouvelle procédure, créez une branche de fonctionnalité. La commande `git checkout -b` permet de créer cette branche et de basculer dessus en une seule opération.
2. **Standardisation des Pull Requests (PR)** : Une fois la documentation terminée et la branche poussée (`git push origin <nom-branche>`), ouvrez une Pull Request vers `main`. Vous devez impérativement utiliser et remplir le template de PR fourni par défaut pour garantir la clarté des modifications apportées.

```bash
# Exemple de création d'une branche de travail
git checkout -b docs/ajout-procedure-securite
```

---

## 👨‍💻 Mainteneurs

* **Louis MEDO** | [LinkedIn](https://www.linkedin.com/in/louismedo/) | [Portfolio](https://louis.loutik.fr/) | [GitHub](https://github.com/FireToak) | [louis.medo@loutik.fr](mailto:louis.medo@loutik.fr)

---

<div align="center">
<br>
<small><i>Dernière mise à jour : 17 mai 2026</i></small>
</div>