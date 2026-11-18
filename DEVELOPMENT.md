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
The API test suite runs standalone without requiring live MLflow instances or previously trained models:
```bash
pytest tests/test_api.py -v
```
To run data input format checks:
```bash
pytest tests/test_data.py -k "test_text_accepts_unicode or test_valid_sample_texts" -v
```
Full data validation and model behavior tests (`pytest tests/ -v`) require running the DVC data ingestion and training stages first.

## 7. Current Baseline Verification Status
- **API Tests (`tests/test_api.py`)**: Verified passing (29 passed in 1.29s).
- **Data Input Tests (`tests/test_data.py`)**: 5 standalone schema/type tests passed. 13 dataset-split tests skipped as `data/processed/train.csv` is not yet ingested locally.
- **Model Tests (`tests/test_model.py`)**: 1 test failed (`test_model_file_exists: AssertionError: No .pkl files found in models/`) because pipeline model training has not been executed yet in this lightweight bootstrap phase.

## 8. Known Setup & Code Problems
- **Fixed in this fork**: Upstream `requirements.txt` contained an invalid literal `" "` entry on line 45 which caused `pip install` to fail immediately with an `InvalidRequirement` syntax error.
- **Fixed in this fork**: `.gitignore` was updated to exclude `.venv/`, `env/`, and `.pytest_cache/` which were previously missing.
- **Model artifact dependency in tests**: `tests/test_model.py` requires `.pkl` artifacts in `models/` on disk, causing failure when run in fresh clones prior to running `src/train.py`.

## 9. Completed vs. Planned Work

### Completed Work
- [x] Cloned and verified upstream Git history and MIT attribution.
- [x] Fixed invalid syntax bug on line 45 of `requirements.txt`.
- [x] Updated `.gitignore` to cover virtual environments and pytest cache directories.
- [x] Verified 29 FastAPI endpoint tests against local Python environment.
- [x] Added fork provenance notices and local development documentation.

### Planned Personal Extensions
- [ ] Provide an offline test fixture or small dummy pipeline model artifact for `tests/test_model.py` to enable full offline CI testing without running full training.
- [ ] Create a local lightweight sample dataset generator for offline DVC testing.
- [ ] Add Docker Compose health check automation.
