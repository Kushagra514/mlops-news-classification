# Local Development and Verification Guide

## 1. Upstream Metadata & Licence
- **Original Upstream Repository**: [MOHD-OMER/mlops-pipeline](https://github.com/MOHD-OMER/mlops-pipeline)
- **Original Licence**: MIT License (Copyright (c) 2026 MOHD-OMER)
- **Fork Target**: [Kushagra514/mlops-news-classification](https://github.com/Kushagra514/mlops-news-classification)

## 2. Project Overview
This project implements an end-to-end MLOps pipeline for multi-class text classification on the AG News dataset (`World`, `Sports`, `Business`, `Sci/Tech`). The system integrates:
- **Data Versioning & Pipelines**: DVC (`dvc.yaml`, `params.yaml`)
- **Experiment Tracking & Model Registry**: MLflow with automated promotion gates
- **API Serving**: FastAPI application (`src/serve.py`) with confidence scores, batch inference, latency metrics, and health checks
- **Drift Monitoring**: Evidently AI (`monitoring/monitor.py`) for data drift detection
- **Automated CI/CD**: Multi-stage GitHub Actions workflow (`.github/workflows/ci.yml`)

## 3. Actual Local Folder Structure
```
mlops-news-classification/
├── .github/
│   └── workflows/
│       └── ci.yml                                                # 4-stage GitHub Actions CI/CD workflow
├── .gitignore                                                    # Cleaned ignore definitions for envs & caches
├── Dockerfile                                                    # Container specification for FastAPI service
├── docker-compose.yml                                            # Multi-service stack (API, MLflow, Prometheus, Grafana)
├── dvc.yaml                                                      # DVC pipeline stages (ingest -> preprocess -> train)
├── params.yaml                                                   # Hyperparameters and pipeline configuration
├── pytest.ini                                                    # Pytest configuration
├── requirements.txt                                              # Pinned Python package dependencies
├── LICENSE                                                       # Original MIT License
├── README.md                                                     # Comprehensive documentation with fork provenance
├── DEVELOPMENT.md                                                # Development instructions, audit, and verification status
├── monitoring/
│   └── monitor.py                                                # Evidently AI data drift monitoring job
├── src/
│   ├── ingest.py                                                 # Dataset ingestion (AG News from HuggingFace)
│   ├── preprocess.py                                             # Cleaning, tokenization, and train/val/test splits
│   ├── train.py                                                  # Model training (TF-IDF + LR/SVM or DistilBERT)
│   ├── evaluate.py                                               # Evaluation metrics, confusion matrix, threshold gate
│   └── serve.py                                                  # Production FastAPI REST application
└── tests/
    ├── test_api.py                                               # FastAPI mock-pipeline endpoint integration tests
    ├── test_data.py                                              # Data integrity and schema validation tests
    └── test_model.py                                             # Scikit-learn pipeline inference tests
```

## 4. Main Execution Entry Points
- **FastAPI Service**:
  ```bash
  uvicorn src.serve:app --host 0.0.0.0 --port 8000
  ```
- **DVC Pipeline Execution**:
  ```bash
  dvc repro
  ```
- **Individual Pipeline Steps**:
  ```bash
  python src/ingest.py
  python src/preprocess.py
  python src/train.py
  python src/evaluate.py
  ```
- **Data Drift Monitor**:
  ```bash
  python monitoring/monitor.py
  ```

## 5. Required Dependencies
- Python 3.10+
- `scikit-learn`, `numpy`, `pandas`, `scipy`
- `fastapi`, `uvicorn`, `pydantic`, `httpx`
- `mlflow`, `dvc`, `evidently`
- `pytest`, `pytest-asyncio`
- Optional deep learning: `torch`, `transformers`, `datasets`

## 6. How to Run Available Tests
Run the entire automated test suite (74 tests covering API, Data, and Model behavior):
```bash
pytest tests/ -v
```
Or run individual test modules:
```bash
pytest tests/test_api.py -v    # 29 FastAPI endpoint integration tests
pytest tests/test_data.py -v   # 22 Data integrity and schema validation tests
pytest tests/test_model.py -v  # 23 Scikit-learn pipeline inference tests
```

## 7. Current Verification & Benchmark Status
All 74 automated tests are verified passing:
- **API Tests (`tests/test_api.py`)**: 29 passed.
- **Data Tests (`tests/test_data.py`)**: 22 passed.
- **Model Tests (`tests/test_model.py`)**: 23 passed.
- **Total**: **74 passed, 0 skipped, 0 failed** in 1.30s.

### Pipeline Stage Verification
1. **Data Ingestion (`src/ingest.py`)**:
   - Downloads and validates AG News splits from Hugging Face (`fancyzhx/ag_news`).
   - Train split: 10,000 rows (2,500 per class balanced).
   - Test split: 1,000 rows (250 per class balanced).
   - Outputs: `data/raw/train_raw.csv`, `data/raw/test_raw.csv`.

2. **Preprocessing (`src/preprocess.py`)**:
   - Normalizes text (lowercasing, number masking, URL replacement, HTML strip).
   - Stratified split: Train (7,700 rows, 70%), Val (1,650 rows, 15%), Test (1,650 rows, 15%).
   - Outputs: `data/processed/train.csv`, `val.csv`, `test.csv`.

3. **Model Training & Experiment Tracking (`src/train.py`)**:
   - Local SQLite tracking backend: `sqlite:///mlruns/mlflow.db`.
   - Evaluated 3 hyperparameter runs:
     - `tfidf_svm_bigrams`: **Val Accuracy 0.8903** (Best - Promoted to Production)
     - `tfidf_lr_bigrams`: Val Accuracy 0.8891
     - `tfidf_lr_baseline`: Val Accuracy 0.8855
   - Logged parameters, validation metrics, confusion matrix, and models serialized with cloudpickle.
   - Automatically registered model `news_classifier` v1 and transitioned stage to `Production`.

4. **Production Model Evaluation (`src/evaluate.py`)**:
   - Evaluated Production model on independent test set (1,650 samples):
     - Accuracy: **0.8788**
     - F1-Macro: **0.8785**
     - Precision: **0.8785**
     - Recall: **0.8788**
     - AUC-ROC: **0.9729**
   - Reports generated: `reports/final_metrics.json`, `reports/final_classification_report.txt`, `reports/final_confusion_matrix.png`.

5. **Data Drift Monitoring (`monitoring/monitor.py`)**:
   - Compared training reference (7,700 samples) vs. test set (1,650 samples).
   - Analyzed 8 NLP features (`text_length`, `word_count`, `avg_word_length`, `num_sentences`, `uppercase_ratio`, etc.).
   - Result: 0 / 8 drifted features (Drift share 0.0%, Dataset Drifted: No).
   - Generated HTML report: `reports/drift_report.html`.

6. **Reproducible DVC Pipeline (`dvc repro`)**:
   - Full pipeline executed and recorded in `dvc.lock`.
   - Verified that all stages (`ingest`, `preprocess`, `train`, `evaluate`, `monitor`) are up to date and fully reproducible.

7. **Production Containerization (Docker)**:
   - Built multi-stage Docker image `news-classifier:local`.
   - Verified live container startup, health check (`/health` -> 200 healthy), and prediction inference (`/predict` -> latency ~4ms).

## 8. Technical Challenges Encountered & Resolved
- **HuggingFace Hub URI Format**: Modern `huggingface_hub` (>=2.4) deprecates bare repository identifiers like `ag_news`. Ingestion now includes automated fallback to `fancyzhx/ag_news`.
- **Pandas 3.0 GroupBy Group Keys**: In Pandas 3.0+, `.groupby('label', group_keys=False).apply(...)` drops grouping columns during subsampling. Replaced with robust `pd.concat([g.sample(...) for _, g in df.groupby('label')])` ensuring cross-version stability.
- **Pandas 3.0 StringDtype Compatibility**: Modern Pandas defaults string columns to `StringDtype` rather than `object`. Updated test schema assertions using `pd.api.types.is_string_dtype()`.
- **MLflow Skops Whitelist with Calibrated Classifier**: MLflow 3.x skops serialization blocked `_CalibratedClassifier`. Configured `serialization_format=mlflow.sklearn.SERIALIZATION_FORMAT_CLOUDPICKLE`.
- **FastAPI Model Versioning Type**: Pydantic response validation expected string model versions while MLflow returned integer IDs. Added strict string normalization in `src/serve.py`.
- **Offline / Local MLflow Server Independence**: Configured default `sqlite:///mlruns/mlflow.db` and environment variable overrides (`MLFLOW_TRACKING_URI`) so local runs, Docker, and CI work seamlessly without separate daemon processes.
- **CI/CD Dockerhub Credentials**: Parameterized DockerHub push in GitHub Actions so builds succeed cleanly even without repository push secrets configured.

## 9. Completed vs. Planned Work

### Completed Work
- [x] Cloned and verified upstream Git history and MIT attribution.
- [x] Fixed invalid syntax bug on line 45 of `requirements.txt`.
- [x] Updated `.gitignore` to cover virtual environments, pytest caches, and sqlite databases.
- [x] Created local virtual environment and installed modern MLOps dependencies.
- [x] Executed full DVC pipeline (`ingest` -> `preprocess` -> `train` -> `evaluate` -> `monitor`) and locked with `dvc.lock`.
- [x] Implemented local MLflow experiment tracking and automated model registry promotion with SQLite backend.
- [x] Validated live FastAPI serving for health, single-sample inference, and batch predictions.
- [x] Validated Evidently AI data drift reporting with fallback PSI algorithm.
- [x] Built and verified production Docker image with live container health checks.
- [x] Modernized test assertions and verified all 74 automated tests passing (29 API, 22 Data, 23 Model).
- [x] Hardened GitHub Actions CI/CD workflow for personal fork execution.
