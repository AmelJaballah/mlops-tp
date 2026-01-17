


# 🚀 MLOps Course Projects Portfolio

**Master MLOps 2025-2026 - Dr. Salah Gontara**

Ce repository contient l'ensemble des travaux pratiques réalisés dans le cadre du cours MLOps. Chaque projet couvre un pilier essentiel du cycle de vie MLOps.

---

## 📂 Structure des TPs

| TP | Projet | Description | Port(s) |
|:--:|--------|-------------|---------|
| **TP2** | [iris-ai-service](./iris-ai-service) | Docker & Docker Compose | 8000, 5174 |
| **TP3** | [mlops-dvc-iris](./mlops-dvc-iris) | Data Versioning avec DVC | 9001 (MinIO) |
| **TP4** | [mlflow-cv-yolo](./mlflow-cv-yolo) | Experiment Tracking MLflow | 5000, 9001 |
| **TP5** | [zenml-cv-yolo](./zenml-cv-yolo) | Pipelines ZenML | 8080, 5000, 9001 |
| **TP6** | [optuna-cv-yolo](./optuna-cv-yolo) | Optimisation Optuna | 8080, 5000, 9001 |
| **TP7** | [deploy-cv-yolo](./deploy-cv-yolo) | Déploiement TorchServe | 8080, 8081 |

---

## 🚀 Commandes Rapides pour Chaque TP

### 🐳 TP2 : Docker & Docker Compose (iris-ai-service)

```powershell
cd c:\Users\q\Desktop\mlops-tp\iris-ai-service
docker compose up -d --build
```

**Interfaces :**
- 🌐 Frontend : http://localhost:5174
- 📡 API Swagger : http://localhost:8000/docs

---

### 📊 TP3 : DVC + MinIO (mlops-dvc-iris)

```powershell
cd c:\Users\q\Desktop\mlops-tp\mlops-dvc-iris
docker compose up -d

# Variables d'environnement
$env:AWS_ACCESS_KEY_ID = "minio"
$env:AWS_SECRET_ACCESS_KEY = "minio12345"

# Commandes DVC
dvc status
dvc push
```

**Interfaces :**
- 🗄️ MinIO Console : http://localhost:9001 (minio / minio12345)

---

### 🔬 TP4 : MLflow (mlflow-cv-yolo)

```powershell
cd c:\Users\q\Desktop\mlops-tp\mlflow-cv-yolo
docker compose up -d

# Créer l'environnement Python
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt

# Variables MLflow
$env:MLFLOW_TRACKING_URI = "http://localhost:5000"

# Lancer un entraînement
python -m src.train_cv --epochs 3 --imgsz 320 --exp-name cv_yolo_tiny

# Lancer la grille
.\scripts\run_grid.ps1
```

**Interfaces :**
- 📊 MLflow UI : http://localhost:5000
- 🗄️ MinIO Console : http://localhost:9001

---

### 🔄 TP5 : ZenML (zenml-cv-yolo)

```powershell
cd c:\Users\q\Desktop\mlops-tp\zenml-cv-yolo
docker compose up -d

# Environnement Python
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt

# Connexion ZenML
zenml connect http://localhost:8080
zenml init
zenml stack set mlflow_stack

# Lancer le pipeline
python -m src.zenml_pipelines.run_yolo_pipeline_baseline
python -m src.zenml_pipelines.run_yolo_pipeline_grid
```

**Interfaces :**
- 🔄 ZenML Server : http://localhost:8080 (admin / zenml)
- 📊 MLflow UI : http://localhost:5000
- 🗄️ MinIO Console : http://localhost:9001

---

### ⚡ TP6 : Optuna (optuna-cv-yolo)

```powershell
cd c:\Users\q\Desktop\mlops-tp\optuna-cv-yolo
docker compose up -d

# Environnement Python
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt

# Variables d'environnement
$env:MLFLOW_TRACKING_URI = "http://localhost:5000"
$env:AWS_ACCESS_KEY_ID = "minio"
$env:AWS_SECRET_ACCESS_KEY = "minio12345"

# Lancer Optuna
.\scripts\run_optuna.ps1
```

**Interfaces :**
- 🔄 ZenML Server : http://localhost:8080
- 📊 MLflow UI : http://localhost:5000
- 🗄️ MinIO Console : http://localhost:9001

---

### 🚢 TP7 : Déploiement TorchServe (deploy-cv-yolo)

```powershell
cd c:\Users\q\Desktop\mlops-tp\deploy-cv-yolo
docker compose up -d
```

**Interfaces :**
- 🌐 API Gateway : http://localhost:8080/docs
- 🤖 TorchServe : http://localhost:8081

---

## 📋 Récapitulatif des URLs

| Service | URL | Credentials |
|---------|-----|-------------|
| **MinIO Console** | http://localhost:9001 | minio / minio12345 |
| **MLflow UI** | http://localhost:5000 | - |
| **ZenML Server** | http://localhost:8080 | admin / zenml |
| **Iris Frontend** | http://localhost:5174 | - |
| **Iris API** | http://localhost:8000/docs | - |

---

## 🛠️ Commandes Docker Utiles

```powershell
# Voir les conteneurs en cours
docker ps

# Arrêter tous les conteneurs d'un projet
docker compose down

# Voir les logs
docker compose logs -f

# Nettoyer les volumes
docker compose down -v

# Tout reconstruire
docker compose up -d --build --force-recreate
```

---

## 🛠️ Stack Technologique

| Catégorie | Outils |
|:----------|:-------|
| **Conteneurisation** | Docker, Docker Compose |
| **Versioning Données** | DVC, Git |
| **Stockage** | MinIO (S3 Compatible) |
| **Experiment Tracking** | MLflow |
| **Orchestration** | ZenML |
| **Optimisation** | Optuna |
| **Serving** | TorchServe, FastAPI |
| **Frameworks ML** | PyTorch, YOLOv8, Scikit-Learn |
| **Frontend** | React, Vite, Nginx |

---

