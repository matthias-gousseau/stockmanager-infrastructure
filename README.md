# StockManager Infrastructure

Infrastructure et configuration Docker Compose pour le déploiement de l'application StockManager (Frontend Angular + Backend Spring Boot + PostgreSQL).

## 📁 Structure

```
stockmanager-infrastructure/
├── docker-compose.dev.yml   # Configuration développement
├── docker-compose.prod.yml  # Configuration production
├── .env.example             # Template des variables d'environnement
└── README.md               # Ce fichier
```

## 🚀 Déploiement

### Prérequis

- Docker et Docker Compose installés sur le serveur
- Accès aux images Docker sur GitHub Container Registry (GHCR)
- **Secrets configurés dans GitHub** (pas de fichier .env nécessaire)

### Configuration des secrets GitHub

Dans le repo `stockmanager-infrastructure`, allez dans **Settings → Secrets → Actions** et ajoutez :

| Secret | Description | Exemple |
|--------|-------------|---------|
| `VPS_HOST` | IP du VPS | `123.45.67.89` |
| `VPS_SSH_KEY` | Clé SSH privée | `-----BEGIN RSA PRIVATE KEY-----...` |
| `DB_USER_DEV` | User PostgreSQL dev | `postgres` |
| `DB_PASSWORD_DEV` | Password PostgreSQL dev | `postgres` |
| `JWT_SECRET_DEV` | Secret JWT dev | `dev_secret` |
| `DB_USER_PROD` | User PostgreSQL prod | `stockmanager_user` |
| `DB_PASSWORD_PROD` | Password PostgreSQL prod | `VotreMotDePasse123!` |
| `JWT_SECRET_PROD` | Secret JWT prod | Générer avec `openssl rand -base64 32` |

### Installation initiale sur le VPS

```bash
# 1. Cloner le repository
cd ~
git clone https://github.com/matthias-gousseau/stockmanager-infrastructure.git stockmanager
cd stockmanager

# 2. Login à GHCR (si les images sont privées)
echo "YOUR_GITHUB_TOKEN" | docker login ghcr.io -u matthias-gousseau --password-stdin

# 3. Pas besoin de créer un fichier .env !
# Les secrets sont injectés automatiquement par les workflows GitHub Actions
```

### Déploiement via GitHub Actions (Recommandé ✅)

Le déploiement se fait automatiquement via GitHub Actions :

1. **Pour le frontend** : Merge sur `dev` ou `main` → déploiement automatique
2. **Pour le backend** : Merge sur `dev` ou `main` → déploiement automatique
3. **Pour l'infrastructure** : Actions → "Deploy Infrastructure" → Run workflow

#### Déploiement manuel de l'infrastructure

Allez dans l'onglet **Actions** du repo, sélectionnez "Deploy Infrastructure", puis **Run workflow** en choisissant l'environnement (dev, prod, ou both).

### Déploiement manuel (si besoin)

⚠️ **Attention** : Le déploiement manuel nécessite d'exporter les variables d'environnement.

```bash
cd ~/stockmanager

# Définir les secrets manuellement (pas recommandé)
export DB_USER_DEV="postgres"
export DB_PASSWORD_DEV="postgres"
export JWT_SECRET_DEV="dev_secret"
export DB_USER_PROD="votre_user"
export DB_PASSWORD_PROD="votre_password"
export JWT_SECRET_PROD="votre_jwt_secret"

# Démarrer les services
docker compose -f docker-compose.dev.yml up -d
docker compose -f docker-compose.prod.yml up -d
```

**Recommandation** : Utilisez toujours le déploiement via GitHub Actions pour éviter d'exposer les secrets.

### Lancer l'environnement de développement

```bash
cd ~/stockmanager

# Pull les dernières images
docker compose -f docker-compose.dev.yml pull

# Démarrer les services
docker compose -f docker-compose.dev.yml up -d

# Vérifier les logs
docker compose -f docker-compose.dev.yml logs -f

# Vérifier le statut
docker compose -f docker-compose.dev.yml ps
```

**Accès :**
- Frontend : http://YOUR_VPS_IP:4201
- Backend : http://YOUR_VPS_IP:8080
- PostgreSQL : localhost:5432

### Lancer l'environnement de production

```bash
cd ~/stockmanager

# Pull les dernières images
docker compose -f docker-compose.prod.yml pull

# Démarrer les services
docker compose -f docker-compose.prod.yml up -d

# Vérifier les logs
docker compose -f docker-compose.prod.yml logs -f

# Vérifier le statut
docker compose -f docker-compose.prod.yml ps
```

**Accès :**
- Frontend : http://YOUR_VPS_IP:4200
- Backend : http://YOUR_VPS_IP:8081
- PostgreSQL : localhost:5433

## 🔄 Mise à jour

### Mise à jour des configurations

```bash
cd ~/stockmanager
git pull origin main
```

### Mise à jour des applications (nouvelles images Docker)

```bash
cd ~/stockmanager

# Pour dev
docker compose -f docker-compose.dev.yml pull
docker compose -f docker-compose.dev.yml up -d

# Pour prod
docker compose -f docker-compose.prod.yml pull
docker compose -f docker-compose.prod.yml up -d
```

## 🛠️ Commandes utiles

### Gestion des services

```bash
# Arrêter les services
docker compose -f docker-compose.dev.yml down
docker compose -f docker-compose.prod.yml down

# Redémarrer un service spécifique
docker compose -f docker-compose.dev.yml restart frontend-dev
docker compose -f docker-compose.prod.yml restart backend-prod

# Voir les logs d'un service
docker compose -f docker-compose.dev.yml logs -f frontend-dev
docker compose -f docker-compose.prod.yml logs -f backend-prod

# Recréer les conteneurs (force update)
docker compose -f docker-compose.dev.yml up -d --force-recreate
```

### Gestion de la base de données

```bash
# Backup de la base de données de dev
docker exec postgres-stockmanager-dev pg_dump -U postgres stockmanager_db > backup_dev.sql

# Backup de la base de données de prod
docker exec postgres-stockmanager-prod pg_dump -U $DB_USER stockmanager_db > backup_prod.sql

# Restore d'un backup
docker exec -i postgres-stockmanager-prod psql -U $DB_USER stockmanager_db < backup_prod.sql

# Accéder au shell PostgreSQL
docker exec -it postgres-stockmanager-dev psql -U postgres -d stockmanager_db
docker exec -it postgres-stockmanager-prod psql -U $DB_USER -d stockmanager_db
```

### Nettoyage

```bash
# Supprimer les images inutilisées
docker image prune -f

# Supprimer les volumes orphelins (ATTENTION : perte de données)
docker volume prune -f

# Tout nettoyer (DANGER)
docker system prune -a --volumes
```

## 🔒 Sécurité

### Variables d'environnement sensibles

**Ne jamais commit le fichier `.env` dans Git !**

Le fichier `.env` contient des secrets et doit rester uniquement sur le serveur.

### Générer un JWT Secret sécurisé

```bash
openssl rand -base64 32
```

### Changer les mots de passe par défaut

Pour la production, assurez-vous de :
- ✅ Utiliser des mots de passe forts pour la base de données
- ✅ Générer un JWT secret unique
- ✅ Ne jamais utiliser les credentials par défaut

## 📊 Monitoring

### Vérifier la santé des services

```bash
# Voir tous les conteneurs
docker ps

# Voir l'utilisation des ressources
docker stats

# Vérifier les healthchecks
docker inspect stockmanager-frontend-dev | grep -A 10 Health
docker inspect stockmanager-backend-prod | grep -A 10 Health
```

## 🌐 Configuration Nginx (Optionnel)

Pour exposer les services via des noms de domaine :

```nginx
# /etc/nginx/sites-available/stockmanager-dev
server {
    listen 80;
    server_name stockmanager-dev.matthiasgousseau.fr;

    location / {
        proxy_pass http://localhost:4201;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /api {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

## 🐛 Troubleshooting

### Le frontend ne se connecte pas au backend

Vérifiez que `API_URL` dans le docker-compose pointe vers la bonne URL.

### Erreur "manifest unknown" lors du pull

L'image n'existe pas encore dans GHCR. Assurez-vous que les workflows CI/CD des repos frontend/backend ont bien pushé les images.

### Base de données ne démarre pas

Vérifiez les logs : `docker compose -f docker-compose.dev.yml logs db-dev`

Vérifiez que le volume n'est pas corrompu.

### Port déjà utilisé

Un autre service utilise le port. Modifiez les ports dans le docker-compose ou arrêtez le service concurrent.

## 📝 Notes

- **Dev et Prod peuvent tourner en même temps** grâce aux ports et noms différents
- Les volumes PostgreSQL sont persistants et survivent aux redémarrages
- Les healthchecks permettent de s'assurer que les services sont opérationnels

## 🔗 Liens utiles

- [Frontend Repository](https://github.com/matthias-gousseau/stockmanager-frontend)
- [Backend Repository](https://github.com/matthias-gousseau/stockmanager-backend)
- [Docker Compose Documentation](https://docs.docker.com/compose/)

## 📧 Support

Pour toute question : contact@matthiasgousseau.fr