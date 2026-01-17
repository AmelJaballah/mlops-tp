# 🔬 TP4 : Experiment Tracking avec MLflow


---

## 📋 Objectifs du TP

- ✅ Utiliser MLflow pour tracer plusieurs runs de détection d'objets (YOLO tiny) sur un dataset ultra-léger
- ✅ Comparer les runs dans l'UI MLflow, analyser les métriques (mAP, précision, rappel)
- ✅ Consigner la décision de promotion
- ✅ (Optionnel) Enregistrer le modèle choisi dans le Model Registry (Stage : Staging/Production)

---

## 🛠️ Prérequis

- **Python 3.11+**
- **Git**
- **Docker Desktop** (pour MLflow et MinIO)
- **PowerShell** (Windows) ou **bash** (Linux/macOS)

---

## 📦 Structure du Projet

```
mlflow-cv-yolo/
├── src/
│   ├── train_cv.py           # Script d'entraînement YOLO
│   └── utils.py              # Utilitaires
├── tools/
│   └── make_tiny_person_from_coco128.py  # Génération du mini-dataset
├── scripts/
│   ├── run_grid.sh           # Grille de runs (Linux/macOS)
│   ├── run_grid.ps1          # Grille de runs (PowerShell)
│   └── run_grid.cmd          # Grille de runs (CMD)
├── data/
│   └── tiny_coco.yaml        # Configuration dataset
├── runs/                     # Résultats des entraînements
├── reports/
│   └── templates/
│       └── decision_template.md  # Gabarit de décision
├── docker-compose.yml        # MLflow + MinIO
├── mlflow.env                # Variables d'environnement MLflow
├── requirements.txt
└── yolov8n.pt               # Poids YOLO pré-entraînés
```

---

## 🚀 Procédure Détaillée

### 0️⃣ Fork & Clone

```bash
# Forkez depuis GitLab/GitHub : MLflow-CV-Yolo
# Puis clonez VOTRE fork
git clone <URL_DE_VOTRE_FORK>.git
cd mlflow-cv-yolo
```

### 1️⃣ Préparation de l'Environnement Python

**Windows PowerShell :**
```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
```

**Linux/macOS :**
```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
```

### 2️⃣ Génération du Mini-Dataset (COCO128 → 1 classe person)

```bash
python tools/make_tiny_person_from_coco128.py
```

**Résultat attendu :**
- 60 images au total (40 train / 10 val / 10 test)
- 1 seule classe : **person**

#### (Optionnel) Tracker avec DVC

```bash
dvc init
dvc add data/tiny_coco -R
git add data/tiny_coco.dvc .gitignore .dvc/
git commit -m "Track dataset tiny_coco with DVC"
```

### 3️⃣ Démarrer MLflow (backend SQLite + artefacts MinIO)

```bash
docker compose up -d
docker compose ps
docker compose logs -f mlflow  # Ctrl+C pour sortir
```

**Accès aux interfaces :**
- **UI MLflow** : http://localhost:5000
- **Console MinIO** : http://localhost:9001 (minio / minio12345)

### 4️⃣ Configurer la Variable MLflow

**Linux/macOS :**
```bash
export MLFLOW_TRACKING_URI=http://localhost:5000
```

**Windows PowerShell :**
```powershell
$env:MLFLOW_TRACKING_URI = "http://localhost:5000"
```

**Windows CMD :**
```cmd
set MLFLOW_TRACKING_URI=http://localhost:5000
```

### 5️⃣ Lancer un Run Baseline

```bash
# Mode package (recommandé)
python -m src.train_cv --epochs 3 --imgsz 320 --exp-name cv_yolo_tiny
```

**Durée estimée** : 1-2 minutes

✅ Vérifiez dans l'UI MLflow qu'un nouveau run apparaît

### 6️⃣ Générer une Grille de Runs (8 runs)

**Linux/macOS :**
```bash
chmod +x scripts/run_grid.sh
bash scripts/run_grid.sh
```

**Windows PowerShell :**
```powershell
powershell -ExecutionPolicy Bypass -File scripts\run_grid.ps1
```

**Windows CMD :**
```cmd
scripts\run_grid.cmd
```

**Configurations testées :**
- Tailles d'image : 320, 416
- Learning rates : 0.005, 0.01
- Seeds : 1, 42

### 7️⃣ Comparaison dans l'UI MLflow

1. Ouvrir http://localhost:5000
2. Menu **Experiments** → `cv_yolo_tiny`
3. Sélectionner plusieurs runs
4. Cliquer sur **Compare**

**Métriques à examiner :**
- `mAP@50` : Précision moyenne à 50% IoU
- `mAP@50-95` : Précision moyenne sur différents seuils
- `precision` / `recall`

**Artefacts disponibles :**
- `results.png` : Courbes d'entraînement
- `confusion_matrix.png` : Matrice de confusion
- `weights/best.pt` : Meilleur modèle
