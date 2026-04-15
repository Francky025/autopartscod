# autopartscod
application web de vente de piece automobile paiement a la livraison de commerce électronique pour la vente de pièces mécaniques automobiles
**Université du Québec en Outaouais (UQO) — Projet de Synthèse, Hiver 2026**  
Auteur : Francky Daniel Fokouo Ngaintchueng  
Coordinateur : Mr Ekwelle Epale Thomas Martial

---

## Table des matières

- [À propos du projet](#à-propos-du-projet)
- [Fonctionnalités](#fonctionnalités)
- [Architecture](#architecture)
- [Stack technique](#stack-technique)
- [Installation](#installation)
- [Configuration](#configuration)
- [Base de données](#base-de-données)
- [API REST — Endpoints](#api-rest--endpoints)
- [Modèles de données](#modèles-de-données)
- [Workflow COD](#workflow-cod)
- [Tests](#tests)
- [Structure du projet](#structure-du-projet)
- [Contribuer](#contribuer)
- [Licence](#licence)

---

## À propos du projet

**AutoPartsCOD** est une plateforme e-commerce spécialisée dans la vente de pièces mécaniques automobiles. Elle résout trois problèmes concrets :

- **Compatibilité** : recherche de pièces par référence, marque, modèle ou véhicule
- **Confiance** : paiement uniquement à la réception (modèle Cash on Delivery)
- **Traçabilité** : suivi complet du cycle de vie de chaque commande

---

## Fonctionnalités

| Acteur | Fonctionnalités |
|---|---|
| **Client** | Recherche de pièces, gestion du panier, passage de commande, suivi en temps réel |
| **Vendeur** | Gestion du catalogue, mise à jour du stock, validation des commandes |
| **Livreur** | Consultation des livraisons assignées, confirmation ou refus à la livraison |
| **Administrateur** | Gestion des utilisateurs, supervision globale, rapports |

---

## Architecture

```
Client React (HTTPS/JSON)
        │
        ▼
API REST Django (DRF)
  ├─ Authentification JWT
  ├─ Gestion catalogue / stock
  ├─ Cycle de vie commandes COD
  └─ Gestion livraisons / paiements
        │
        ▼
Base de données PostgreSQL
        │
        ▼
Service Livraison (API externe ou interne)
```

Architecture trois couches : **Présentation → Application → Données**

---

## Stack technique

| Couche | Technologie |
|---|---|
| Backend | Python 3.11+, Django 4.x, Django REST Framework |
| Base de données | PostgreSQL 15+ |
| Authentification | JWT via `djangorestframework-simplejwt` |
| Frontend *(à venir)* | React.js |
| Hébergement cible | Render (cloud) |
| Gestion dépendances | pip + virtualenv |

---

## Installation

### Prérequis

- Python 3.11+
- PostgreSQL 15+
- Git

### Étapes

```bash
# 1. Cloner le dépôt
git clone https://github.com/TON_USERNAME/autopartscod.git
cd autopartscod

# 2. Créer et activer l'environnement virtuel
python -m venv venv

# Windows
venv\Scripts\activate

# macOS / Linux
source venv/bin/activate

# 3. Installer les dépendances
pip install -r requirements.txt

# 4. Configurer les variables d'environnement
cp .env.example .env
# Éditer .env avec vos valeurs

# 5. Appliquer les migrations
python manage.py migrate

# 6. Créer un superutilisateur
python manage.py createsuperuser

# 7. Lancer le serveur
python manage.py runserver
```

---

## Configuration

Créez un fichier `.env` à la racine du projet (ne jamais committer ce fichier) :

```env
SECRET_KEY=django-insecure-changeme-autopartscod-2026
DEBUG=True

DB_NAME=autopartscod_db
DB_USER=postgres
DB_PASSWORD=votre_mot_de_passe
DB_HOST=localhost
DB_PORT=5432
```

Un fichier `.env.example` est fourni comme modèle.

---

## Base de données

Créer la base PostgreSQL avant les migrations :

```sql
CREATE DATABASE autopartscod_db;
CREATE USER autoparts_user WITH PASSWORD 'votre_mot_de_passe';
GRANT ALL PRIVILEGES ON DATABASE autopartscod_db TO autoparts_user;
```

---

## API REST — Endpoints

### Authentification

| Méthode | Endpoint | Description |
|---|---|---|
| `POST` | `/api/token/` | Obtenir un token JWT |
| `POST` | `/api/token/refresh/` | Rafraîchir le token |

### Catalogue

| Méthode | Endpoint | Description |
|---|---|---|
| `GET` | `/api/pieces/` | Lister toutes les pièces |
| `GET` | `/api/pieces/?marque=Toyota` | Filtrer par marque / modèle |
| `GET` | `/api/pieces/<id>/` | Détail d'une pièce |
| `POST` | `/api/pieces/` | Ajouter une pièce *(vendeur)* |
| `PUT` | `/api/pieces/<id>/` | Modifier une pièce *(vendeur)* |

### Commandes

| Méthode | Endpoint | Description |
|---|---|---|
| `POST` | `/api/commandes/` | Créer une commande |
| `GET` | `/api/commandes/` | Lister mes commandes |
| `GET` | `/api/commandes/<id>/` | Détail d'une commande |
| `PATCH` | `/api/commandes/<id>/statut/` | Changer le statut |

### Livraisons

| Méthode | Endpoint | Description |
|---|---|---|
| `GET` | `/api/livraisons/` | Lister les livraisons |
| `POST` | `/api/livraisons/<id>/confirmer/` | Confirmer la livraison |
| `POST` | `/api/livraisons/<id>/refuser/` | Signaler un refus |

### Paiements

| Méthode | Endpoint | Description |
|---|---|---|
| `POST` | `/api/paiements/<id>/enregistrer/` | Enregistrer le paiement COD |
| `GET` | `/api/paiements/<id>/` | Détail d'un paiement |

---

## Modèles de données

```
Utilisateur (AbstractUser)
 ├─ Client
 ├─ Vendeur
 ├─ Livreur
 └─ Administrateur

Piece
 └─ référence · marque · modèle véhicule · stock · prix

Commande  ──1:N──  LigneCommande  ──N:1──  Piece
    │
   1:1
    │
Livraison
    │
   1:1
    │
Paiement (type: Cash on Delivery)
```

---

## Workflow COD

Le cycle de vie d'une commande suit ces états :

```
Créée → Validée → Préparée → Expédiée → Livrée ✓
                                    └──→ Refusée ✗
```

Le paiement est enregistré **uniquement** après confirmation de livraison par le livreur. En cas de refus client, la commande passe à l'état `Refusée` et le paiement reste `annulé`.

---

## Tests

```bash
# Lancer tous les tests
python manage.py test

# Avec pytest
pip install pytest pytest-django
pytest

# Tester un module spécifique
pytest orders/tests.py -v
```

Couverture cible : cycle de vie commande, refus livraison, calcul total, compatibilité pièces.

---

## Structure du projet

```
autopartscod/
├── config/                  # Configuration Django
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── users/                   # Gestion des utilisateurs et rôles
├── catalog/                 # Pièces automobiles et stock
├── orders/                  # Commandes et lignes de commande
├── delivery/                # Livraisons
├── payments/                # Paiements COD
├── .env                     # Variables d'environnement (non commité)
├── .env.example             # Modèle de configuration
├── .gitignore
├── manage.py
├── requirements.txt
└── README.md
```

---

## Contribuer

1. Fork le projet
2. Crée une branche : `git checkout -b feature/nom-de-la-fonctionnalite`
3. Commit tes changements : `git commit -m "Ajout de la fonctionnalité X"`
4. Push : `git push origin feature/nom-de-la-fonctionnalite`
5. Ouvre une Pull Request

---

## Licence

Projet académique — Université du Québec en Outaouais (UQO), Hiver 2026.  
Tous droits réservés © Francky Daniel Fokouo Ngaintchueng.
