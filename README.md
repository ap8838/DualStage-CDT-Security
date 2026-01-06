DualStage-CDT-Security

A research prototype for Anomaly Detection in Cloud Digital Twins, combining
Autoencoder (AE), GANomaly, Temporal Modeling, and optional Blockchain-based alert logging.

This repository is organized as a versioned research codebase, with each version corresponding to a clear experimental contribution.

🔁 Reproducibility Guarantee

All experimental results are reproducible under the following conditions:

Python: 3.10

OS: Linux / Windows

Training regime: Unsupervised (models trained only on normal samples)

Splitting: Chronological (time-aware)

Random seeds: Fixed across preprocessing and training

Versioning: Each experiment is tagged in Git (v0, v1, v2, v3)

⚠️ Some datasets contain no anomalies in the evaluation window after chronological splitting.
In such cases, ROC-AUC is undefined and anomaly metrics are reported as zero.
This reflects realistic deployment scenarios.

📌 Versioned Experimental Scope
Version	Description
v0 (Baseline)	Point-wise AE & GANomaly, 80/20 chronological split
v1 (Temporal)	Sliding window (w=5), dataset-aware split (Linux/Windows: 60/40)
v2 (Feature-Aware)	Feature-type-aware preprocessing (categorical vs numeric)
v3 (Final)	Dual-threshold / weighted anomaly scoring

🚀 Quickstart (Core Research Experiments)
1️⃣ Environment Setup
python -m venv .venv
source .venv/bin/activate        # Linux/macOS
.venv\Scripts\activate           # Windows PowerShell

pip install -r requirements.txt

2️⃣ Dataset Preprocessing

Generates:

Processed parquet files

Scalers / encoders

Feature metadata

python -m src.data.preprocess


Outputs:

data/processed/
artifacts/preproc/

3️⃣ Model Training
Autoencoder (AE) — Point-wise Baseline
python scripts/run_models.py ae --dataset iot_fridge
# or train all
python scripts/run_models.py ae

GANomaly

v0 (Baseline):

python scripts/run_models.py ganomaly --dataset iot_fridge


v1 (Temporal, w=5):

python scripts/run_models.py ganomaly --dataset linux_disk1 --window 5
# or all datasets
python scripts/run_models.py ganomaly --window 5

4️⃣ Threshold Computation
python scripts/compute_best_threshold.py --model ae
python scripts/compute_best_threshold.py --model ganomaly --window 5

5️⃣ Evaluation
# AE (Point-wise)
python scripts/run_evals.py ae

# GANomaly (Temporal)
python scripts/run_evals.py ganomaly --window 5


Evaluation outputs:

artifacts/reports/

🧪 Minimal Reproduction (Reviewer-Friendly)

To reproduce a core v1 result:

python -m src.data.preprocess
python -m src.models.train_ganomaly --dataset linux_disk1 --window 5
python -m src.models.eval_ganomaly --dataset linux_disk1 --window 5

🔌 System Integration (Optional / Non-Essential for Research)

The following components are not required to reproduce model performance
but demonstrate real-world deployment.

🔹 API Server
uvicorn src.api.main:app --reload --port 8000


Endpoint:

POST /score


Example payload:

{
  "asset_id": "iot_fridge",
  "timestamp": "2025-09-24T12:00:00Z",
  "features": { "fridge_temperature": 999 }
}

🔹 Frontend (Optional)
cd ui
npm install
npm run dev

🔹 Streaming Simulator
python -m src.stream.streamer \--input data/processed/iot_fridge_test.parquet \--rate 2 \--speed 1.0 \--endpoint http://127.0.0.1:8000/score


Alerts stored in:

artifacts/stream/stream.db

🔹 Blockchain Logging (Optional)

Start Ganache:

npx ganache


Create .env:

RPC_URL=http://127.0.0.1:8545
PRIVATE_KEY=<GANACHE_PRIVATE_KEY>
CHAIN_ID=1337
VITE_API_BASE=http://localhost:8000


Deploy contract:

python src/blockchain/deploy_contract.py


Verify alert:

python -m src.blockchain.verify_alert <tx_hash>

🧬 Adversarial Data Generation (Optional)
python scripts/run_all_cgan.py
python scripts/generate_all_cgan_samples.py
python scripts/eval_all_generated.py


Outputs:

artifacts/adversarial/


🏷️ Git Tags
v0-baseline
v1-temporal
v2-feature-aware
v3-final