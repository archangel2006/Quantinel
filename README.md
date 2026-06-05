# Quantinel

Hybrid Quantum-Classical Intrusion Detection and Attack Behavior Characterization System

---

## Overview

Quantinel is a cybersecurity analysis system that combines classical machine learning with quantum machine learning to detect and characterize network intrusions.

It performs:
- Intrusion detection using classical ML models
- Attack behavior characterization using quantum similarity methods
- Mapping of unknown attacks to known attack families
- Explainable threat reporting using rule-based logic

The goal is not just to classify traffic, but to understand *what type of attack behavior it resembles*.

---

## Features

### Classical Detection
- Random Forest / XGBoost / SVM models
- Binary classification (Normal vs Attack)
- Fast and scalable detection pipeline

### Quantum Characterization
- Qiskit-based quantum feature encoding
- Fidelity quantum kernel similarity
- QSVC-based classification in quantum feature space
- Attack family mapping for unseen or ambiguous attacks

### Similarity Analysis
- Classical similarity (Cosine / KNN)
- Quantum similarity (kernel-based embedding space comparison)
- Comparison between classical vs quantum behavior mapping

### Threat Explanation
- Rule-based ThreatLens engine
- Severity scoring (Low / Medium / High / Critical)
- Human-readable attack indicators

---

## System Flow

<img width="2829" height="3836" alt="mermaid-diagram" src="https://github.com/user-attachments/assets/0d75113b-d51f-4e04-913c-1c28b5a59ce7" />


---

## Attack Families

- DoS
- Probe
- R2L
- U2R
- Normal

---
## Tech Stack

| Category | Tools |
|----------|------|
| Quantum Computing | Qiskit, Qiskit Machine Learning |
| Quantum Methods | ZZFeatureMap, Fidelity Quantum Kernel, QSVC |
| Classical ML | Scikit-learn, Random Forest, XGBoost, SVM |
| Data Processing | Pandas, NumPy |
| Preprocessing | StandardScaler, OneHotEncoding, Feature Selection |

---

## Methodology

### 1. Classical Detection Layer
- Detects whether network traffic is normal or malicious
- Uses supervised ML models (Random Forest / XGBoost / SVM)

### 2. Similarity Analysis Layer
Once an attack is detected:
- Classical similarity (Cosine / KNN)
- Quantum similarity (kernel-based embedding space)

### 3. Quantum Characterization Layer
- Encodes features into quantum states
- Computes similarity in Hilbert space
- Maps attack to closest family

### 4. Threat Interpretation Layer
- Rule-based reasoning (ThreatLens)
- Generates severity and behavioral indicators

---

## Dataset

Primary:
- NSL-KDD

Future:
- CICIDS2017
- UNSW-NB15


## Dataset Feature Groups

- Basic connection features (duration, protocol_type, service)
- Traffic statistics (count, srv_count, serror_rate)
- Host-based behavior metrics (dst_host_count, srv_diff_host_rate)
- Content-based features (failed logins, root access indicators)
- Labels (attack type / normal)

---

## Output Example

```
Prediction: Attack
Family: Probe
Similarity: 0.78
Confidence: 62%
Severity: Medium

Indicators:
- Sequential connection attempts
- High destination host diversity
- Service scanning behavior detected

```

---

## Goal

To explore whether quantum feature embeddings can improve similarity-based attack characterization compared to classical similarity methods in intrusion detection systems.


## Research Question

Can quantum-kernel-based feature embeddings improve attack family characterization for unseen or ambiguous network intrusions compared to classical similarity methods?


## Baseline Comparison

- Classical similarity: Cosine similarity, KNN
- Classical ML kernels: RBF kernel (SVM)
- Quantum similarity: Fidelity quantum kernel (QSVC)


---

## Research Contribution

- Hybrid classical + quantum intrusion detection pipeline
- Quantum similarity-based attack characterization
- Open-set attack mapping using behavioral embeddings
- Comparison of classical vs quantum similarity spaces

