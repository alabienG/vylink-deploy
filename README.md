# Vylink — Déploiement

Repo public contenant le `docker-compose.yml` pour déployer **Vylink** sur n'importe quel serveur
avec Docker installé. Les images Docker sont hébergées sur GitHub Container Registry (GHCR) et
sont publiques : aucune authentification requise.

## Prérequis

- Docker ≥ 24 avec le plugin Compose (`docker compose version`)
- Un serveur Linux avec les ports 80 (et optionnel 443) ouverts

## Déploiement en 3 étapes

### 1. Récupérer les fichiers de déploiement

```bash
git clone https://github.com/alabienG/vylink-deploy.git
cd vylink-deploy
```

> Ou sans git :
> ```bash
> mkdir vylink && cd vylink
> curl -fsSL https://raw.githubusercontent.com/alabienG/vylink-deploy/main/docker-compose.yml -o docker-compose.yml
> curl -fsSL https://raw.githubusercontent.com/alabienG/vylink-deploy/main/.env.example -o .env.example
> ```

### 2. Configurer l'environnement

```bash
cp .env.example .env
nano .env   # ou vim, ou tout autre éditeur
```

Les variables **obligatoires** à renseigner :

| Variable | Description |
|----------|-------------|
| `GITHUB_OWNER` | Votre username GitHub (ex: `monorg`) |
| `POSTGRES_PASSWORD` | Mot de passe fort pour PostgreSQL |
| `JWT_SECRET` | Clé secrète JWT (min. 64 caractères) |
| `CORS_ALLOWED_ORIGINS` | URL publique de votre serveur (ex: `http://192.168.1.10`) |

> **Générer un JWT_SECRET sécurisé :**
> ```bash
> openssl rand -hex 32
> ```

### 3. Lancer l'application

```bash
docker compose up -d
```

C'est tout. Docker va :
1. Démarrer **PostgreSQL** et attendre qu'il soit prêt
2. Démarrer le **backend Spring Boot** (migrations Flyway automatiques)
3. Démarrer le **frontend Angular** (Nginx + proxy `/api/` → backend)

L'application est accessible sur `http://votre-serveur` (port 80).

---

## Utiliser une version précise

Par défaut, les images `latest` sont utilisées. Pour épingler des versions spécifiques, éditez `.env` :

```dotenv
BACKEND_VERSION=v1.2.0
FRONTEND_VERSION=v1.2.0
```

Puis rechargez :

```bash
docker compose pull && docker compose up -d
```

> Les versions disponibles sont listées sur
> [ghcr.io/alabienG/vylink-backend](https://github.com/alabienG?tab=packages)

---

## Commandes utiles

```bash
# Voir l'état des conteneurs
docker compose ps

# Voir les logs en temps réel
docker compose logs -f

# Voir les logs d'un seul service
docker compose logs -f backend
docker compose logs -f frontend

# Mettre à jour vers la dernière version
docker compose pull && docker compose up -d

# Arrêter sans supprimer les données
docker compose stop

# Arrêter et supprimer les conteneurs (les données PostgreSQL sont conservées dans le volume)
docker compose down
```

---

## HTTPS avec Caddy (recommandé en production)

Installez **Caddy** sur le serveur pour obtenir un certificat Let's Encrypt automatiquement :

```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https curl
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update && sudo apt install caddy
```

Éditez `/etc/caddy/Caddyfile` :

```
votre-domaine.fr {
    reverse_proxy localhost:80
}
```

```bash
sudo systemctl reload caddy
```

> ⚠️ Pensez à changer `HTTP_PORT=127.0.0.1:80` dans `.env` pour ne pas exposer le port 80 directement
> et laisser Caddy gérer les connexions entrantes.

---

## Mise à jour des images sources

Les images sont automatiquement buildées et publiées sur GHCR par les pipelines CI/CD des repos
privés à chaque création d'un tag Git :

```bash
# Dans le repo backend ou frontend (privé)
git tag v1.2.0
git push origin v1.2.0
```

Le tag `latest` est également mis à jour automatiquement.

---

## Structure

```
vylink-deploy/
├── docker-compose.yml   ← définition des services (pull depuis GHCR)
├── .env.example         ← template de configuration
└── README.md            ← ce fichier
```




