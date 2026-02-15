# Projet Final Ansible - Application Web 3-Tiers

## Description

Ce projet déploie automatiquement une application web complète en utilisant Ansible. L'application est organisée en 3 couches (frontend, backend, base de données) sur des conteneurs Docker.

### Conteneurs

- **web1** (SSH port 2221) : Frontend Nginx
- **web2** (SSH port 2222) : Frontend Nginx (haute disponibilité)
- **db1** (SSH port 2223) : Backend API Node.js + PostgreSQL

## Prérequis

- Ansible (version 2.9 ou supérieure)
- Docker et Docker Compose
- sshpass (pour l'authentification SSH)

### Accès SSH aux conteneurs

- **Utilisateur** : `root`
- **Mot de passe** : `ansible`
- **Hôte** : `127.0.0.1`
- **Ports** : 2221 (web1), 2222 (web2), 2223 (db1)

## Structure du Projet

```
projet-ansible-final/
├── README.md
├── ansible.cfg
├── .vault_pass
├── inventory/
│   └── hosts
├── group_vars/
│   └── all/
│       ├── vars.yml
│       └── vault.yml
├── playbooks/
│   ├── site.yml
│   └── tests.yml
└── roles/
    ├── common/
    ├── nginx/
    ├── nodejs/
    └── postgresql/
```

## Installation

### 1. Configuration du vault

Créer le fichier `.vault_pass` avec le mot de passe :

```bash
echo "ansible123" > .vault_pass
chmod 600 .vault_pass
```

### 2. Vérifier la connectivité

```bash
ansible all -m ping
```

### 3. Déployer l'application

```bash
# Vérifier la syntaxe
ansible-playbook playbooks/site.yml --syntax-check

# Déploiement complet
ansible-playbook playbooks/site.yml
```

## Utilisation

### Accéder à l'application

**Frontend (Nginx)** :
```bash
curl http://localhost:80
```

**API Backend** :
```bash
# Endpoint racine
curl http://localhost:3000/

# Liste des utilisateurs
curl http://localhost:3000/users
```

**Via le proxy Nginx** :
```bash
curl http://localhost/api/
curl http://localhost/api/users
```

## Tests

### Tests automatisés

```bash
ansible-playbook playbooks/tests.yml
```

### Tests manuels

```bash
# Vérifier les pages web
ansible webservers -m uri -a "url=http://localhost status_code=200"

# Vérifier l'API
ansible appservers -m uri -a "url=http://localhost:3000/ status_code=200"
ansible appservers -m uri -a "url=http://localhost:3000/users status_code=200"

# Vérifier les processus
ansible webservers -m shell -a "pgrep -x nginx >/dev/null"
ansible appservers -m shell -a "pgrep -fa postgres >/dev/null"
ansible appservers -m shell -a "pgrep -f 'node.*app.js'"
```

### Vérifier l'idempotence

Rejouez le playbook - aucune tâche ne devrait être modifiée :

```bash
ansible-playbook playbooks/site.yml
```

## Gestion des Secrets

Les secrets sont stockés dans `group_vars/all/vault.yml` et chiffrés avec Ansible Vault.

**Mot de passe du vault** : `ansible123`

```bash
# Voir le contenu
ansible-vault view group_vars/all/vault.yml

# Éditer les secrets
ansible-vault edit group_vars/all/vault.yml
```

## Rôles Implémentés

### Rôle `common`
- Mise à jour du cache apt
- Installation des packages essentiels (curl, vim, htop, git)

### Rôle `nginx`
- Installation de Nginx
- Configuration d'une page HTML d'accueil
- Proxy reverse vers l'API backend (`/api/` → `http://db1:3000/`)
- Handlers pour le rechargement du service

### Rôle `postgresql`
- Installation de PostgreSQL
- Création de la base de données et de l'utilisateur
- Initialisation de la table `users`

### Rôle `nodejs`
- Installation de Node.js et npm
- Déploiement de l'API Express
- Endpoints : `/` et `/users`
- Connexion à PostgreSQL
