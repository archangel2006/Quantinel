# Quantinel — Fullstack System Design 


## 1. System Overview

Quantinel is a hybrid intrusion detection + attack characterization system with:

- Frontend Dashboard → visualization + dataset interaction
- Backend API (FastAPI) → orchestration layer
- ML Pipeline (Classical + Quantum) → inference engine
- Colab Notebooks → research + experimentation layer
- Explainability Engine (ThreatLens) → rule-based interpretation

---

## 🏗️ 2. High-Level Architecture Flow

```

CSV Upload (Frontend)
        ↓
POST /api/upload
        ↓
Dataset Validation + Storage
        ↓
POST /api/analyze/{dataset_id}
        ↓
Preprocessing Pipeline
        ↓
Classical ML Model (IDS)
        ↓
Prediction Table Generated
        ↓
IF Attack:
    ├── Classical Similarity Module
    ├── Quantum Similarity Module (Qiskit)
        ↓
Attack Family Mapping
        ↓
ThreatLens Explainability Engine
        ↓
Structured JSON Response
        ↓
Frontend Dashboard Rendering

```

---

📁 3. Project Folder Structure

```
quantinel/
│
├── backend/
│   ├── main.py                  # FastAPI entry
│   ├── routes/
│   │   ├── upload.py
│   │   ├── analyze.py
│   │   ├── results.py
│   │
│   ├── services/
│   │   ├── preprocessing.py
│   │   ├── classical_model.py
│   │   ├── quantum_model.py
│   │   ├── similarity.py
│   │   ├── threatlens.py
│   │
│   ├── models/
│   │   ├── rf_model.pkl
│   │   ├── xgboost.pkl
│   │
│   ├── utils/
│   │   ├── encoders.py
│   │   ├── scaler.py
│   │   ├── validators.py
│
├── frontend/
│   ├── src/
│   │   ├── pages/
│   │   │   ├── Dashboard.jsx
│   │   │   ├── UploadPage.jsx
│   │   │   ├── RecordDetails.jsx
│   │   │
│   │   ├── components/
│   │   │   ├── DataTable.jsx
│   │   │   ├── AttackGraph.jsx
│   │   │   ├── SeverityChart.jsx
│   │   │   ├── ThreatPanel.jsx
│   │
│   │   ├── api/
│   │   │   ├── client.js
│
├── notebooks/
│   ├── 01_data_exploration.ipynb
│   ├── 02_preprocessing_pipeline.ipynb
│   ├── 03_classical_models.ipynb
│   ├── 04_quantum_feature_maps.ipynb
│   ├── 05_qs_kernel_experiments.ipynb
│   ├── 06_similarity_analysis.ipynb
│   ├── 07_ablation_study.ipynb
│
├── datasets/
│   ├── raw/
│   ├── processed/
│
└── README.md

```
---

## 🌐 4. Backend API Design (IMPORTANT)

### 📤 Dataset Upload
```
POST /api/upload
```

**Input:** CSV file (network traffic dataset)
**Output:**
```
{
  "dataset_id": "abc123",
  "rows": 50000,
  "features": 41,
  "status": "uploaded"
}
```

### ⚙️ Run Full Analysis
```
POST /api/analyze/{dataset_id}
```

**Internal Flow:**
- preprocessing
- classical model inference
- quantum similarity (if attack)
- ThreatLens explanation

**Output:**
```
{
  "summary": {
    "normal": 47000,
    "attack": 3000
  }
}
```

### 📊 Get Full Results Table
```
GET /api/results/{dataset_id}
```
**Output (row-level enriched CSV):**
| src_ip | dst_ip | prediction | attack_family | classical_conf | quantum_similarity | severity |
| ------ | ------ | ---------- | ------------- | -------------- | ------------------ | -------- |

### 🔍 Get Single Record Insight
```
GET /api/record/{dataset_id}/{row_id}
```

**Output:**
```
{
  "prediction": "Attack",
  "family": "Probe",
  "quantum_similarity": 0.78,
  "classical_similarity": 0.62,
  "severity": "MEDIUM",
  "threat_indicators": [
    "Sequential connection attempts",
    "High destination host diversity"
  ],
  "rule_explanation": "Possible reconnaissance behavior detected"
}
```

---

### 🤖 5. ML + QML Pipeline Connection

**Classical Pipeline:** 
```
CSV → Encoding → Scaling → Random Forest / XGBoost → Prediction
```

**Output:**
- Normal / Attack
- Confidence score


**Quantum Pipeline (Triggered only if Attack):**

```
Feature Vector
     ↓
ZZFeatureMap
     ↓
Quantum State Encoding
     ↓
Fidelity Quantum Kernel
     ↓
QSVC / Similarity Search
     ↓
Attack Family Mapping
```

**Output:**
- Probe / DoS / R2L / U2R

---

### 📊 6. Frontend Dashboard Design

🧩 1. **Upload Page**
- CSV upload
- dataset status
- preprocessing progress

📊 2. **Global Analytics Dashboard**

- Pie chart → Normal vs Attack distribution
- Bar chart → Attack families
- Line chart → severity distribution over dataset
- Heatmap → suspicious IP clusters

📋 3. **Data Table View**

| Feature    | Value               |
| ---------- | ------------------- |
| Prediction | Attack / Normal     |
| Confidence | %                   |
| Family     | Probe / DoS         |
| Severity   | Low / Medium / High |


✔ Table is editable view only
✔ Sorted by severity or confidence

🔍 4. **Record Drilldown (IMPORTANT)**

Click any row → opens:

**Shows:**
- Full feature vector
- Classical prediction explanation
- Quantum similarity score
- ThreatLens rule triggers
- Attack family reasoning
- Mini graphs:
  - similarity comparison (classical vs quantum)
  - severity gauge
  - timeline (if batch)

---

### 📈 7. Dashboard Data Flow

```
Backend Results JSON
        ↓
Frontend API Client
        ↓
State Store (React / Context)
        ↓
Table Rendering
        ↓
Click Event
        ↓
GET /record/{id}
        ↓
Threat Panel + Graphs Rendered
```

---

### 📓 8. Colab / Notebook Pipeline (RESEARCH CORE)

**01_data_exploration.ipynb**
- dataset structure
- label distribution
- feature correlation
  
**02_preprocessing_pipeline.ipynb**
- encoding
- scaling
- feature selection

**03_classical_models.ipynb**
- RF / XGBoost / SVM
- baseline metrics

**04_quantum_feature_maps.ipynb**
- ZZFeatureMap experiments
- circuit visualization

**05_qs_kernel_experiments.ipynb**
- Fidelity kernel similarity
- QSVC training

**06_similarity_analysis.ipynb**
- classical vs quantum comparison
- distance metrics

**07_ablation_study.ipynb**
- remove quantum layer
- compare performance drop
- similarity quality metrics

---

### 🎯 9. Key Research Outputs

- Row-level enriched predictions
- Attack family mapping
- Similarity score comparison (classical vs quantum)
- Explainability rules
- Dataset-level analytics

---

### 💡10. Core Idea

Quantinel transforms raw network traffic into a structured intelligence system that not only detects attacks but explains their behavioral similarity to known threat families using classical and quantum feature spaces.
