# 🔄 TP5 : Pipelines MLOps avec ZenML et MLflow


---

## 📋 Objectifs du TP

- ✅ Reprendre le projet CV YOLO tiny du TP4 et l'encapsuler dans un pipeline ZenML
- ✅ Utiliser une stack préconfigurée dans ZenML Server : orchestrateur local, artifact store S3 sur MinIO, MLflow comme experiment tracker
- ✅ Lancer plusieurs runs de pipeline (baseline + variations) et les analyser dans l'UI MLflow
- ✅ Découvrir l'UI ZenML Server (pipelines, stacks, runs)

---

## 🛠️ Prérequis

- **Python 3.11+**
- **Git**
- **Docker Desktop** (pour ZenML Server, MLflow, MinIO)

---

## 📦 Structure du Projet

```
zenml-cv-yolo/
├── src/
│   ├── train_cv.py                      # Script d'entraînement YOLO
│   ├── zenml_pipelines/
│   │   ├── yolo_training_pipeline.py    # Définition du pipeline ZenML
│   │   ├── run_yolo_pipeline_baseline.py
│   │   └── run_yolo_pipeline_grid.py
│   └── zenml_steps/
│       ├── data_steps.py                # Step préparation données
│       ├── train_steps.py               # Step entraînement
│       └── eval_steps.py                # Step évaluation
├── tools/
│   └── make_tiny_person_from_coco128.py
├── reports/templates/
│   └── decision_template_zenml.md
├── docker-compose.yml
├── requirements.txt
└── yolov8n.pt
```

---

## 🚀 Procédure Détaillée

### 0️⃣ Fork & Clone

```bash
git clone <URL_DE_VOTRE_FORK>.git
cd zenml-cv-yolo
```

### 1️⃣ Préparation de l'Environnement
```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
```

### 2️⃣ Gestion des Données (DVC)
```powershell
# Générer le dataset minimal (si nécessaire)
python tools/make_tiny_person_from_coco128.py

# Vérifier le tracking DVC
dvc status
```

### 3️⃣ Démarrage de l'Infrastructure (Docker)
```powershell
docker compose up -d
docker compose ps

# Accès aux interfaces :
# - MLflow : http://localhost:5000
# - MinIO  : http://localhost:9001
# - ZenML  : http://localhost:8080
```

### 4️⃣ Configuration de la Stack ZenML (Admin Server)
*À exécuter une seule fois pour configurer les composants du serveur.*
```powershell
docker exec -it zenml-server bash

# -- À l'intérieur du conteneur --
# 1. Register MLflow Experiment Tracker
zenml experiment-tracker register mlflow_tracker --flavor=mlflow --tracking_uri=http://mlflow:5000 --tracking_token="dummy-token"

# 2. Créer le secret pour MinIO
zenml secret create minio_zenml_secret --aws_access_key_id='minio' --aws_secret_access_key='minio12345'

# 3. Register Artifact Store (S3/MinIO)
zenml artifact-store register minio_artifacts --flavor=s3 --path='s3://zenml-artifacts' --authentication_secret=minio_zenml_secret --client_kwargs='{"endpoint_url": "http://minio:9000"}'

# 4. Register et activer la Stack
zenml stack register mlflow_stack -o default -a minio_artifacts -e mlflow_tracker
zenml stack set mlflow_stack
exit
```

### 5️⃣ Connexion Client & Exécution
```powershell
# Connexion au serveur depuis votre machine
zenml connect http://localhost:8080
zenml init

# Sélectionner la stack
zenml stack set mlflow_stack

# Lancer le pipeline Baseline
python -m src.zenml_pipelines.run_yolo_pipeline_baseline

# Lancer la Grille de runs
python -m src.zenml_pipelines.run_yolo_pipeline_grid
```

---

## 📊 Analyse des Runs

### Dans MLflow (http://localhost:5000)
- Comparez les métriques : `mAP@50`, `mAP@50-95`, `precision`, `recall`
- Examinez les artefacts : `results.png`, `confusion_matrix.png`

### Dans ZenML Server (http://localhost:8080)
- Visualisez le DAG du pipeline
- Inspectez les métadonnées des runs



## 📝 Questions de Compréhension

### 1. À quoi servent concrètement les décorateurs `@step` et `@pipeline` ?
*   **`@step`** : Ce décorateur transforme une fonction Python en une étape de pipeline ZenML. Concrètement, il permet à ZenML de suivre les entrées/sorties (artifacts), de gérer le **caching** (ne pas recalculer si les entrées n'ont pas changé) et de faciliter la robustesse (exécution isolée).
*   **`@pipeline`** : Ce décorateur définit la structure globale (le DAG - Directed Acyclic Graph) du processus MLOps. Il orchestre l'ordre d'exécution des étapes décorées avec `@step` et assure le flux de données entre elles.

### 2. Quels sont les artefacts principaux produits par chaque step ?
*   **Step `prepare_tiny_coco_dataset`** : Produit un artefact de type **données** (le chemin vers le dataset validé par DVC).
*   **Step `train_yolo_tiny`** : Produit un artefact de type **modèle** (le fichier `.pt` d'Ultralytics) et des logs d'entraînement.
*   **Step `summarize_yolo_experiment`** : Produit un artefact de type **rapport/métadonnées** récapitulant les performances du run.

### 3. Qu'est-ce qui est stocké dans ZenML Server / MinIO vs dans MLflow ?
*   **ZenML Server & MinIO** :
    *   **ZenML Server** stocke les **métadonnées** : noms des runs, configurations des stacks, historique des exécutions, et les relations entre les artefacts.
    *   **MinIO** (Artifact Store) stocke les **fichiers réels** : le dataset préparé et les poids du modèle fournis par ZenML.
*   **MLflow** (Experiment Tracker) :
    *   Stocke les **métriques scientifiques** (courbes de perte/loss, précision, mAP au fil des époques).
    *   Stocke les **artefacts de visualisation** (matrices de confusion, images de prédiction d'exemple) pour faciliter l'analyse comparative visuelle.

---

## 💡 Réflexion MLOps
L'usage de ZenML apporte une **reproductibilité** totale. Pour GitLab CI, il suffirait d'ajouter un runner avec accès réseau au serveur ZenML et d'utiliser une API Key pour automatiser ces exécutions.

---

