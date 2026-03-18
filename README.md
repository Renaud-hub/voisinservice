# 🏘️ VoisinService

> Marketplace de services entre voisins avec paiement **MTN MoMo**  
> Inspiré de Allovoisin & Yoojo — Stack: React + Node.js + MongoDB

---

## 📋 Table des matières

- [Fonctionnalités](#-fonctionnalités)
- [Architecture](#-architecture)
- [Installation rapide](#-installation-rapide)
- [Variables d'environnement](#-variables-denvironnement)
- [MTN MoMo API](#-mtn-momo-api---configuration)
- [Déploiement Docker](#-déploiement-docker)
- [Déploiement Production](#-déploiement-production)
- [Structure des fichiers](#-structure-des-fichiers)
- [API Routes](#-api-routes)
- [Comptes de test](#-comptes-de-test)

---

## ✨ Fonctionnalités

### 🌍 Côté Public
- ✅ Liste de services avec **filtres avancés** (catégorie, ville, prix, note, PRO)
- ✅ Recherche full-text
- ✅ Tri : récents, prix, popularité, note
- ✅ Pagination
- ✅ Page détail service avec avis
- ✅ Inscription / Connexion JWT

### 📅 Réservations
- ✅ Réservation avec date, durée, adresse
- ✅ Workflow : En attente → Confirmée → En cours → Terminée
- ✅ Annulation client / Refus prestataire
- ✅ Notifications temps-réel (Socket.io)

### 💛 Paiement MTN MoMo
- ✅ Intégration API officielle MTN MoMo Collections
- ✅ Request to Pay (push notification mobile)
- ✅ Polling automatique du statut (5s × 12 tentatives)
- ✅ Webhook callback
- ✅ Remboursement via Disbursements API
- ✅ Validation numéro MoMo
- ✅ Commission plateforme configurable (10% par défaut)
- ✅ Paiement cash / carte également supportés

### ⚙️ Panel Admin
- ✅ Dashboard KPI (revenus, transactions, utilisateurs, services)
- ✅ Gestion services : approbation / rejet / suppression
- ✅ Création directe de services
- ✅ Gestion utilisateurs (rôles, activation/désactivation)
- ✅ Suivi transactions MTN MoMo avec remboursement
- ✅ Gestion réservations
- ✅ CRUD catégories

### 🔧 Prestataire
- ✅ Soumission de service pour validation
- ✅ Dashboard prestations reçues
- ✅ Accepter / Refuser / Terminer une réservation

---

## 🏗️ Architecture

```
voisinservice/
├── backend/                    # Node.js + Express
│   ├── server.js              # Point d'entrée + Socket.io
│   ├── models/
│   │   ├── User.js            # Utilisateurs (client/provider/admin)
│   │   ├── Service.js         # Services proposés
│   │   ├── Booking.js         # Réservations
│   │   ├── Payment.js         # Transactions MTN MoMo
│   │   └── index.js           # Category, Review, Notification
│   ├── routes/
│   │   ├── auth.js            # Register, Login, Profile
│   │   ├── services.js        # CRUD services + filtres
│   │   ├── bookings.js        # Réservations + workflow statuts
│   │   ├── payments.js        # MTN MoMo (initiate/callback/refund)
│   │   ├── admin.js           # Panel admin complet
│   │   ├── reviews.js         # Avis clients
│   │   ├── notifications.js   # Notifications
│   │   ├── categories.js      # Catégories publiques
│   │   └── users.js           # Profils utilisateurs
│   ├── middleware/
│   │   └── auth.js            # JWT protect + authorize
│   └── config/
│       ├── momo.js            # Service MTN MoMo API
│       └── seeder.js          # Données de test
│
├── frontend/                   # React 18
│   └── src/
│       ├── App.js             # Routes React Router v6
│       ├── context/
│       │   └── AuthContext.js # Authentification globale
│       ├── utils/
│       │   └── api.js         # Axios + tous les appels API
│       ├── components/
│       │   ├── Navbar.js      # Navigation
│       │   ├── ServiceCard.js # Carte service
│       │   └── MomoModal.js   # Modal paiement MTN MoMo
│       └── pages/
│           ├── HomePage.js    # Accueil + filtres + grille
│           ├── ServiceDetailPage.js
│           ├── BookingPage.js # Formulaire réservation
│           ├── DashboardPage.js
│           ├── LoginPage.js
│           ├── RegisterPage.js
│           ├── OfferServicePage.js
│           └── admin/
│               ├── AdminLayout.js
│               ├── AdminDashboard.js
│               ├── AdminServices.js
│               ├── AdminAddService.js
│               ├── AdminUsers.js
│               ├── AdminTransactions.js
│               ├── AdminBookings.js
│               └── AdminCategories.js
│
├── docker-compose.yml
├── Dockerfile
├── .env.example
└── README.md
```

---

## 🚀 Installation rapide

### Prérequis
- Node.js 18+
- MongoDB (local ou Atlas)
- Compte MTN MoMo Developer

### 1. Cloner et configurer

```bash
git clone https://github.com/votre-user/voisinservice.git
cd voisinservice

# Copier et remplir les variables d'environnement
cp .env.example .env
nano .env
```

### 2. Installer les dépendances

```bash
# Backend
cd backend && npm install && cd ..

# Frontend
cd frontend && npm install && cd ..
```

### 3. Peupler la base de données

```bash
cd backend
node config/seeder.js
```

### 4. Lancer en développement

```bash
# Terminal 1 — Backend
cd backend && npm run dev   # http://localhost:5000

# Terminal 2 — Frontend
cd frontend && npm start    # http://localhost:3000
```

---

## 🔐 Variables d'environnement

Copiez `.env.example` en `.env` et remplissez :

| Variable | Description | Exemple |
|----------|-------------|---------|
| `MONGODB_URI` | URL MongoDB | `mongodb://localhost:27017/voisinservice` |
| `JWT_SECRET` | Clé secrète JWT | Chaîne aléatoire 32+ caractères |
| `MTN_MOMO_BASE_URL` | URL API MoMo | `https://sandbox.momodeveloper.mtn.com` |
| `MTN_MOMO_COLLECTIONS_PRIMARY_KEY` | Clé primaire collections | Depuis portail MTN |
| `MTN_MOMO_API_USER` | UUID utilisateur API | UUID généré |
| `MTN_MOMO_API_KEY` | Clé API | Depuis portail MTN |
| `MTN_MOMO_ENVIRONMENT` | Environnement | `sandbox` ou `production` |
| `MTN_MOMO_CALLBACK_URL` | URL webhook | `https://votreapp.com/api/payments/momo/callback` |
| `COMMISSION_RATE` | Taux commission | `0.10` (= 10%) |

---

## 💛 MTN MoMo API — Configuration

### Étape 1 : Créer un compte développeur
1. Aller sur [https://momodeveloper.mtn.com](https://momodeveloper.mtn.com)
2. S'inscrire et créer un profil développeur
3. Souscrire au produit **Collection** (et optionnellement **Disbursement**)
4. Copier votre **Primary Key** et **Secondary Key**

### Étape 2 : Créer un utilisateur API (sandbox)

```bash
# Générer un UUID pour votre API User
UUID=$(uuidgen)  # ou https://www.uuidgenerator.net/

# Créer l'utilisateur API
curl -X POST https://sandbox.momodeveloper.mtn.com/v1_0/apiuser \
  -H "X-Reference-Id: $UUID" \
  -H "Ocp-Apim-Subscription-Key: VOTRE_PRIMARY_KEY" \
  -H "Content-Type: application/json" \
  -d '{"providerCallbackHost": "https://votreapp.com"}'

# Obtenir la clé API
curl -X POST https://sandbox.momodeveloper.mtn.com/v1_0/apiuser/$UUID/apikey \
  -H "Ocp-Apim-Subscription-Key: VOTRE_PRIMARY_KEY"
```

### Étape 3 : Remplir le `.env`

```env
MTN_MOMO_API_USER=<UUID généré ci-dessus>
MTN_MOMO_API_KEY=<Clé retournée par l'API>
MTN_MOMO_COLLECTIONS_PRIMARY_KEY=<Votre Primary Key>
```

### Numéros de test (sandbox)
| Numéro | Comportement |
|--------|-------------|
| `46733123450` | Paiement SUCCESSFUL |
| `46733123451` | Paiement FAILED |
| `46733123452` | Timeout / PENDING |

### Flux de paiement
```
Client → [Saisit N° MoMo]
   ↓
POST /api/payments/momo/initiate
   ↓
MTN API → requestToPay → referenceId retourné
   ↓
Client reçoit notification mobile MTN
   ↓
Client confirme (ou refuse) sur son téléphone
   ↓
MTN API → POST /api/payments/momo/callback (webhook)
   OR
Frontend → polling GET /api/payments/momo/status/:refId (toutes les 5s)
   ↓
Si SUCCESSFUL → Réservation confirmée + notifications envoyées
Si FAILED → Erreur affichée + possibilité de réessayer
```

---

## 🐳 Déploiement Docker

```bash
# Construire et lancer tous les services
docker-compose up --build -d

# Voir les logs
docker-compose logs -f backend

# Peupler la base de données
docker-compose exec backend node config/seeder.js

# Arrêter
docker-compose down
```

Services exposés :
- **Frontend** : http://localhost:80
- **Backend API** : http://localhost:5000
- **MongoDB** : localhost:27017

---

## 🌐 Déploiement Production

### Option A — VPS (Ubuntu)

```bash
# 1. Installer Docker
curl -fsSL https://get.docker.com | sh

# 2. Cloner le projet
git clone https://github.com/votre-user/voisinservice.git
cd voisinservice

# 3. Configurer l'environnement production
cp .env.example .env
# Éditer .env : NODE_ENV=production, vraies clés MoMo, MONGODB Atlas...

# 4. Lancer
docker-compose up -d --build

# 5. Seeder (première fois)
docker-compose exec backend node config/seeder.js
```

### Option B — Render.com (Backend gratuit)

1. Créer un compte sur [render.com](https://render.com)
2. **New Web Service** → connecter votre repo GitHub
3. Build command: `cd backend && npm install`
4. Start command: `node backend/server.js`
5. Ajouter toutes les variables d'environnement
6. **New Static Site** pour le frontend avec `cd frontend && npm run build`

### Option C — Railway.app

```bash
# Installer Railway CLI
npm install -g @railway/cli
railway login
railway up
```

### Option D — Heroku

```bash
heroku create voisinservice
heroku addons:create mongolab:sandbox
heroku config:set NODE_ENV=production JWT_SECRET=xxx ...
git push heroku main
heroku run node backend/config/seeder.js
```

### HTTPS avec Let's Encrypt (VPS)

```bash
# Installer Certbot
apt install certbot python3-certbot-nginx

# Obtenir un certificat
certbot --nginx -d votredomaine.com

# Renouvellement automatique
echo "0 12 * * * /usr/bin/certbot renew --quiet" | crontab -
```

---

## 📡 API Routes

### Auth
| Méthode | Route | Description |
|---------|-------|-------------|
| POST | `/api/auth/register` | Inscription |
| POST | `/api/auth/login` | Connexion |
| GET  | `/api/auth/me` | Profil connecté |
| PUT  | `/api/auth/profile` | Modifier profil |
| PUT  | `/api/auth/password` | Changer mot de passe |

### Services
| Méthode | Route | Description |
|---------|-------|-------------|
| GET  | `/api/services?q=&category=&city=&minPrice=&maxPrice=&minRating=&sort=&page=` | Liste filtrée |
| GET  | `/api/services/:id` | Détail service |
| POST | `/api/services` | Créer service (provider) |
| PUT  | `/api/services/:id` | Modifier service |
| DELETE | `/api/services/:id` | Supprimer service |

### Paiement MTN MoMo
| Méthode | Route | Description |
|---------|-------|-------------|
| POST | `/api/payments/momo/initiate` | Initier un paiement |
| POST | `/api/payments/momo/callback` | Webhook MTN (automatique) |
| GET  | `/api/payments/momo/status/:refId` | Vérifier statut |
| POST | `/api/payments/momo/refund` | Rembourser |
| POST | `/api/payments/validate-number` | Valider numéro MoMo |
| GET  | `/api/payments/balance` | Solde compte (admin) |

### Admin
| Méthode | Route | Description |
|---------|-------|-------------|
| GET  | `/api/admin/stats` | KPIs dashboard |
| GET/POST | `/api/admin/services` | Gestion services |
| PATCH | `/api/admin/services/:id/approve` | Approuver/rejeter |
| GET/PATCH | `/api/admin/users` | Gestion utilisateurs |
| GET  | `/api/admin/transactions` | Transactions MoMo |
| GET  | `/api/admin/bookings` | Toutes réservations |
| CRUD | `/api/admin/categories` | Catégories |

---

## 👤 Comptes de test

Après avoir lancé `node config/seeder.js` :

| Rôle | Email | Mot de passe |
|------|-------|--------------|
| 👑 Admin | admin@voisinservice.com | Admin@1234 |
| 👤 Client | client@test.com | Test1234 |
| 🔧 Prestataire | brice@test.com | Test1234 |
| 🔧 Prestataire | awa@test.com | Test1234 |

---

## 🔧 Technologies utilisées

### Backend
- **Node.js** + **Express** — Serveur API REST
- **MongoDB** + **Mongoose** — Base de données
- **JWT** — Authentification
- **Socket.io** — Notifications temps-réel
- **MTN MoMo API** — Paiement mobile
- **Helmet** + **Rate Limiting** — Sécurité
- **Bcryptjs** — Hash mots de passe

### Frontend
- **React 18** — Interface utilisateur
- **React Router v6** — Navigation SPA
- **Axios** — Appels API
- **React Hot Toast** — Notifications
- **Socket.io-client** — Temps-réel

### DevOps
- **Docker** + **Docker Compose** — Conteneurisation
- **Nginx** — Reverse proxy + serveur statique
- **MongoDB Atlas** compatible — Cloud DB

---

## 📞 Support

Pour toute question sur l'intégration MTN MoMo :
- Documentation officielle : https://momodeveloper.mtn.com/docs
- Forum développeurs : https://momodeveloper.mtn.com/forums

---

*VoisinService — Fait avec ❤️ pour les communautés africaines*
