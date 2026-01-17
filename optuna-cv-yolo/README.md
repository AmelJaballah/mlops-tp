# ⚡ TP6 : Optimisation des Hyperparamètres avec Optuna



## 📋 Objectifs du TP

- ✅ Reprendre le projet CV YOLO tiny (TP4/TP5) et y ajouter une recherche d'hyperparamètres
- ✅ Comparer rapidement une approche grille (script `run_grid`) à une approche Optuna (random / bayésien)
- ✅ Utiliser Optuna pour lancer plusieurs entraînements YOLO, tout en loggant les runs dans MLflow
- ✅ Analyser dans l'UI MLflow les hyperparamètres testés et les métriques obtenues
- ✅ Rédiger un mini compte-rendu de décision

---

## 🏗️ Architecture Technique

| Composant | Technologie |
|-----------|-------------|
| **Modèle** | YOLOv8 tiny (ultralytics) |
| **Dataset** | Tiny COCO (personnes), versionné avec DVC |
| **Tracking** | MLflow (Expériences, Paramètres, Métriques) |
| **Stockage Artefacts** | MinIO (Compatible AWS S3) |
| **Optimisation** | Optuna (TPE - Tree-structured Parzen Estimator) |
| **Infrastructure** | Docker Compose |

---

## 📦 Structure du Projet

```
optuna-cv-yolo/
├── src/
│   ├── train_cv.py              # Script d'entraînement YOLO
│   └── optuna_yolo.py           # Script Optuna
├── scripts/
│   ├── run_grid.sh/ps1/cmd      # Grille d'hyperparamètres
│   └── run_optuna.sh/ps1/cmd    # Lancement Optuna
├── tools/
│   └── make_tiny_person_from_coco128.py
├── data/
│   └── tiny_coco.yaml
├── runs/                        # Résultats des entraînements
├── reports/templates/
│   └── decision_template.md
├── docker-compose.yml
├── requirements.txt
└── yolov8n.pt
```

---

## 🚀 Procédure Détaillée

### 0️⃣ Fork & Clone

```bash
git clone <URL_DE_VOTRE_FORK>.git
cd optuna-cv-yolo
```

### 1️⃣ Préparation de l'Environnement Python

**Windows PowerShell :**
```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

**Linux/macOS :**
```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### 2️⃣ Dataset Minimal & DVC

```bash
python tools/make_tiny_person_from_coco128.py
dvc status
```

### 3️⃣ Démarrer MLflow + MinIO

```bash
docker compose up -d
docker compose ps
```

**Accès aux interfaces :**
- **UI MLflow** : http://localhost:5000
- **Console MinIO** : http://localhost:9001 (minio / minio12345)

### 4️⃣ Configurer les Variables d'Environnement

**Windows PowerShell :**
```powershell
$env:MLFLOW_TRACKING_URI = "http://localhost:5000"
$env:MLFLOW_S3_ENDPOINT_URL = "http://localhost:9000"
$env:AWS_ACCESS_KEY_ID = "minio"
$env:AWS_SECRET_ACCESS_KEY = "minio12345"
```

**Linux/macOS :**
```bash
export MLFLOW_TRACKING_URI=http://localhost:5000
export MLFLOW_S3_ENDPOINT_URL=http://localhost:9000
export AWS_ACCESS_KEY_ID=minio
export AWS_SECRET_ACCESS_KEY=minio12345
```

---

## 🏃‍♂️ Exécution des Expériences

### 1. Run Baseline

```bash
python -m src.train_cv --epochs 3 --imgsz 320 --exp-name yolo_baseline_optuna
```

### 2. Grid Search (Approche Naïve)

**Windows PowerShell :**
```powershell
.\scripts\run_grid.ps1
```

**Linux/macOS :**
```bash
bash scripts/run_grid.sh
```

### 3. Optuna Search (Approche Avancée)

**Windows PowerShell :**
```powershell
.\scripts\run_optuna.ps1
```

**Linux/macOS :**
```bash
bash scripts/run_optuna.sh
```

---

## 📊 Analyse des Runs dans MLflow

Dans l'UI MLflow (http://localhost:5000) :

1. **Identifiez les runs** :
   - Grille : `yolo_e3_320`, `yolo_e5_416`, etc.
   - Optuna : `optuna_yolo_trial_...`

2. **Comparez** :
   - Hyperparamètres : `epochs`, `imgsz`
   - Métriques : `metrics/mAP50(B)`, `metrics/mAP50-95(B)`


## 📝 Compte-Rendu & Décision

### Questions à traiter

1. **Efficacité** : Pour 5-10 essais, Optuna trouve-t-il de meilleurs hyperparamètres que la grille ?
2. **Avantages Optuna** : Recherche intelligente vs grille naïve
3. **Risques/Limites** : Temps d'entraînement, coût GPU, surapprentissage sur validation

### Décision Recommandée

**Configuration retenue** (exemple) :
- `epochs`: **5**
- `imgsz`: **320**
- **Performance attendue** : mAP50 ≈ **0.168**

### Avantages d'Optuna en MLOps

1. **Efficacité** : TPE apprend des essais précédents, explore seulement les zones prometteuses
2. **Automatisation** : Processus entièrement automatisé
3. **Tracking Unifié** : Intégration avec MLflow pour la traçabilité

---

