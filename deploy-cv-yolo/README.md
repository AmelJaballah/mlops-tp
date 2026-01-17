# 🚀 TP7 : Déploiement de Modèles avec TorchServe

---

## 📋 Objectifs du TP

- ✅ Packager un modèle YOLO en archive `.mar` pour TorchServe
- ✅ Déployer le modèle avec TorchServe et Docker
- ✅ Créer une API Gateway (FastAPI) pour exposer le service
- ✅ Gérer les versions de modèles (v1/v2/rollback)

---

## 📦 Structure du Projet

```
deploy-cv-yolo/
├── api-gateway/
│   ├── Dockerfile
│   ├── requirements.txt
│   └── app/
│       └── main.py              # API Gateway FastAPI
├── serving/
│   └── torchserve/
│       ├── config.properties    # Configuration TorchServe
│       ├── requirements.txt
│       └── yolo_handler.py      # Handler personnalisé YOLO
├── models/
│   └── weights/
│       ├── best.pt              # Poids du modèle
│       └── best.onnx
├── backup/
│   └── yolo_v1.mar              # Archive modèle packagé
├── scripts/
│   ├── package_mar.sh           # Script packaging (Linux)
│   ├── package_mar.ps1          # Script packaging (PowerShell)
│   └── smoke_test.sh            # Test du service
├── samples/                     # Images de test
├── docker-compose.yml
└── README.md
```

---

## 🚀 Commandes d'Exécution

### 1️⃣ Lancer l'Infrastructure

```powershell
cd c:\Users\q\Desktop\mlops-tp\deploy-cv-yolo
docker compose up -d
docker compose ps
```

### 2️⃣ Vérifier les Services

**API Gateway** : http://localhost:8080
- Swagger UI : http://localhost:8080/docs

**TorchServe** :
- Inference API : http://localhost:8081
- Management API : http://localhost:8082

### 3️⃣ Packager le Modèle (si besoin)

**Windows PowerShell :**
```powershell
.\scripts\package_mar.ps1
```

**Linux/macOS :**
```bash
chmod +x scripts/package_mar.sh
./scripts/package_mar.sh
```

### 4️⃣ Tester l'Inférence

```powershell
# Via l'API Gateway
Invoke-RestMethod -Uri "http://localhost:8080/predict" -Method POST -InFile "samples/test.jpg"

# Directement via TorchServe
curl -X POST http://localhost:8081/predictions/yolo -T samples/test.jpg
```

### 5️⃣ Arrêter les Services

```powershell
docker compose down
```

---

## 📊 Endpoints de l'API

| Endpoint | Méthode | Description |
|----------|---------|-------------|
| `/health` | GET | Health check |
| `/predict` | POST | Prédiction sur une image |
| `/models` | GET | Liste des modèles chargés |
| `/models/{name}/version` | GET | Version du modèle |

---

## 🐳 Docker Compose

```yaml
services:
  torchserve:
    image: pytorch/torchserve:latest
    ports:
      - "8081:8080"    # Inference API
      - "8082:8081"    # Management API
    volumes:
      - ./models:/home/model-server/model-store
      - ./serving/torchserve/config.properties:/home/model-server/config.properties

  api-gateway:
    build: ./api-gateway
    ports:
      - "8080:8080"
    depends_on:
      - torchserve
```

---

## 📝 Questions de Compréhension

### 1. Qu'est-ce qu'un fichier `.mar` ?
Un fichier `.mar` (Model Archive) est le format de packaging standard de TorchServe. Il contient :
- Les poids du modèle (`best.pt`)
- Le handler personnalisé (`yolo_handler.py`)
- Les dépendances et métadonnées

### 2. Comment fonctionne l'API Gateway ?
L'API Gateway (FastAPI) sert d'interface unifiée entre les clients et TorchServe :
- Expose une API REST simple (`/predict`)
- Gère l'authentification et la validation
- Route les requêtes vers TorchServe

### 3. Comment gérer plusieurs versions de modèles ?
TorchServe supporte nativement le versioning :
```bash
# Lister les modèles
curl http://localhost:8082/models

# Enregistrer une nouvelle version
curl -X POST "http://localhost:8082/models?url=yolo_v2.mar"

# Rollback vers une ancienne version
curl -X PUT "http://localhost:8082/models/yolo?default_version=1.0"
```

---

## 🔧 Dépannage

| Problème | Solution |
|----------|----------|
| TorchServe ne démarre pas | Vérifier les logs : `docker compose logs torchserve` |
| Modèle non trouvé | Vérifier que le `.mar` est dans `models/` |
| Erreur de prédiction | Vérifier le format de l'image (JPEG/PNG) |
| Port déjà utilisé | Changer les ports dans `docker-compose.yml` |

---

## 👨‍🏫 Auteur

**Cours MLOps 2025-26 - Dr. Salah Gontara**
