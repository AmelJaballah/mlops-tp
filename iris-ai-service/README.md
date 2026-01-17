# 🐳 TP2 : Docker et Docker Compose pour MLOps


---

## 📋 Objectifs du TP

- ✅ Containeriser un service IA (API FastAPI + Frontend React)
- ✅ Construire et tester chaque image individuellement avant l'orchestration
- ✅ Utiliser Docker Compose pour orchestrer plusieurs conteneurs

---

##  Description 

Application complète de prédiction basée sur le dataset Iris utilisant un modèle RandomForest :

- **API FastAPI** : Backend Python pour les prédictions ML (port 8000)
- **Frontend React** : Interface utilisateur moderne avec Vite (port 5174)
- **Monitoring** : Prometheus + Grafana (optionnel)

---

## 🏗️ Architecture 

```
iris-ai-service/
├── api/
│   ├── Dockerfile              # Image basée sur python:3.11-slim
│   ├── requirements.txt        # Dépendances Python
│   └── app/
│       ├── main.py             # Point d'entrée FastAPI
│       ├── models.py           # Modèles Pydantic
│       ├── db.py               # Gestion base de données
│       └── model/
│           ├── model.joblib    # Modèle ML entraîné
│           └── model_metadata.json
├── frontend/
│   ├── Dockerfile              # Multi-stage: node:20-alpine → nginx:alpine
│   ├── package.json
│   ├── vite.config.js
│   ├── nginx.conf              # Configuration Nginx
│   └── src/
│       ├── App.jsx
│       └── main.jsx
├── monitoring/
│   ├── prometheus.yml          # Configuration Prometheus
│   └── grafana-fastapi-dashboard.json
├── docker-compose.yml          # Orchestration des services
└── README.md
```

---

##  Prérequis

- **Docker Desktop** (Windows/Mac) ou **Docker Engine** (Linux)
- **Docker Compose** v2.0+
- **Git**
- Ports **8000** et **5174** disponibles

---

## 🚀 Procédure Détaillée

### 1️⃣ Fork et Clone du Projet

```bash
# Forkez depuis : https://gitlab.com/mlops_tps/iris-ai-service
# Puis clonez VOTRE fork
git clone git@gitlab.com:<votre_utilisateur>/iris-ai-service.git
cd iris-ai-service
```

### 2️⃣ Build et Test Individuel - API

```bash
cd api
docker build -t iris-api:dev .
docker run -d -p 8000:8000 --name iris-api iris-api:dev

# Vérification
curl -s http://localhost:8000/health
```

**Accès :**
- Healthcheck : http://localhost:8000/health
- Swagger UI : http://localhost:8000/docs

```bash
# Arrêter le conteneur test
docker stop iris-api && docker rm iris-api
```

### 3️⃣ Build et Test Individuel - Frontend

```bash
cd ../frontend
docker build -t iris-frontend:dev .
docker run -d -p 5174:80 --name iris-frontend iris-frontend:dev
```

**Accès :**
- Interface Web : http://localhost:5174

```bash
# Arrêter le conteneur test
docker stop iris-frontend && docker rm iris-frontend
cd ..
```

### 4️⃣ Exécution avec Docker Compose

```bash
# Depuis la racine du projet
docker compose up --build

# OU en mode détaché (arrière-plan)
docker compose up --build -d

# Vérifier l'état des conteneurs
docker compose ps

# Devrait afficher :
# NAME            STATE    PORTS
# iris-api        Up       0.0.0.0:8000->8000/tcp
# iris-frontend   Up       0.0.0.0:5174->80/tcp
```

**Accès final :**
- **API Swagger** : http://localhost:8000/docs
- **API Health** : http://localhost:8000/health
- **Frontend** : http://localhost:5174

---

## 🐳 Dockerfiles Créés

### `api/Dockerfile`

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app/ ./app/
EXPOSE 8000
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### `frontend/Dockerfile`

```dockerfile
# Build stage
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```



## 📊 Variables d'Environnement

| Variable | Service | Description | Valeur |
|----------|---------|-------------|--------|
| `API_PORT` | API | Port d'écoute | 8000 |
| `CORS_ORIGINS` | API | Origines CORS autorisées | http://localhost:5174 |
| `VITE_API_BASE` | Frontend | URL de base de l'API | http://localhost:8000 |

---

## 🧪 Test de l'API

### Health Check

```bash
curl http://localhost:8000/health
```

### Prédiction

```bash
curl -X POST http://localhost:8000/predict \
  -H "Content-Type: application/json" \
  -d '{"sepal_length": 5.1, "sepal_width": 3.5, "petal_length": 1.4, "petal_width": 0.2}'
```

**Réponse attendue :**
```json
{
  "prediction": "setosa",
  "probabilities": {"setosa": 0.95, "versicolor": 0.03, "virginica": 0.02}
}
```

---

## 🔧 Commandes Docker Utiles

```bash
# Démarrer les services
docker compose up -d

# Arrêter les services
docker compose down

# Voir les logs
docker compose logs -f

# Reconstruire sans cache
docker compose build --no-cache

# Accéder au shell du conteneur API
docker compose exec api /bin/bash
```

