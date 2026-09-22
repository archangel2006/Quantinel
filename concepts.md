# Quantinel: Comprehensive Quantum Computing & QML Concepts Guide
**Project:** Quantinel — A Hybrid Quantum-Classical Machine Learning Framework for Intrusion Detection and Similarity-Based Attack Characterization  
**Organization:** DRDO Project  
**Author / Presenter:** Vaibhavi Srivastava  
**Target Audience:** Presentation & Defense before Mentors, Evaluators, and Technical Committees

---

## Table of Contents
1. [Core Quantum Computing Fundamentals](#1-core-quantum-computing-fundamentals)
   - [Classical Bit vs. Qubit](#classical-bit-vs-qubit)
   - [Superposition & The Bloch Sphere](#superposition--the-bloch-sphere)
   - [Hilbert Space & Dimensional Scaling](#hilbert-space--dimensional-scaling)
   - [Quantum Gates: Single-Qubit & Multi-Qubit](#quantum-gates-single-qubit--multi-qubit)
   - [Quantum Entanglement & Non-Locality](#quantum-entanglement--non-locality)
2. [Quantum Machine Learning (QML) Architecture](#2-quantum-machine-learning-qml-architecture)
   - [The Data Embedding (Feature Map) Problem](#the-data-embedding-feature-map-problem)
   - [Types of Quantum Encodings](#types-of-quantum-encodings)
   - [Deep Dive: ZZFeatureMap Circuit & Mathematics](#deep-dive-zzfeaturemap-circuit--mathematics)
   - [Computational Hardness & Classical Intractability](#computational-hardness--classical-intractability)
3. [Quantum Kernel Methods & QSVC](#3-quantum-kernel-methods--qsvc)
   - [The Classical vs. Quantum Kernel Trick](#the-classical-vs-quantum-kernel-trick)
   - [Quantum State Fidelity as an Inner Product](#quantum-state-fidelity-as-an-inner-product)
   - [Compute-Uncompute Protocol](#compute-uncompute-protocol)
   - [Quantum Support Vector Classifier (QSVC) Dual Optimization](#quantum-support-vector-classifier-qsvc-dual-optimization)
4. [Quantinel Pipeline Engineering & Design Decisions](#4-quantinel-pipeline-engineering--design-decisions)
   - [Why a Two-Stage Hybrid Architecture?](#why-a-two-stage-hybrid-architecture)
   - [Why 8 Qubits? (NISQ Era Realities)](#why-8-qubits-nisq-era-realities)
   - [The 3-Tier Feature Selection Strategy](#the-3-tier-feature-selection-strategy)
   - [Dataset A (3 Classes) vs. Dataset B (4 Classes / U2R Imbalance)](#dataset-a-3-classes-vs-dataset-b-4-classes--u2r-imbalance)
   - [Closed-World vs. Open-World Evaluation Dynamics](#closed-world-vs-open-world-evaluation-dynamics)
   - [The 34.58% vs. 78.72% Accuracy Distinction](#the-3458-vs-7872-accuracy-distinction)
5. [Mentor Defense Q&A (Anticipated Tough Questions)](#5-mentor-defense-qa-anticipated-tough-questions)

---

# 1. Core Quantum Computing Fundamentals

### Classical Bit vs. Qubit
- **Classical Bit:** Can exist only in one of two discrete states: $0$ or $1$. Physical realization: low/high voltage in a semiconductor transistor.
- **Qubit (Quantum Bit):** The fundamental unit of quantum information. A two-level quantum mechanical system represented as a state vector $|\psi\rangle$ in a 2-dimensional complex Hilbert space $\mathbb{C}^2$.

### Superposition & The Bloch Sphere
A qubit can exist in a linear combination of its basis states $|0\rangle$ and $|1\rangle$:
$$|\psi\rangle = \alpha |0\rangle + \beta |1\rangle, \quad \alpha, \beta \in \mathbb{C}$$
where:
$$|0\rangle = \begin{bmatrix} 1 \\ 0 \end{bmatrix}, \quad |1\rangle = \begin{bmatrix} 0 \\ 1 \end{bmatrix}$$
- **Born's Rule:** Measurement forces the qubit to probabilistically collapse into $|0\rangle$ with probability $|\alpha|^2$ or into $|1\rangle$ with probability $|\beta|^2$.
- **Normalization Condition:** Total probability must equal 1:
  $$|\alpha|^2 + |\beta|^2 = 1$$
- **Bloch Sphere Geometric Representation:** Parameterizing $\alpha = \cos\left(\frac{\theta}{2}\right)$ and $\beta = e^{i\phi}\sin\left(\frac{\theta}{2}\right)$:
  $$|\psi\rangle = \cos\left(\frac{\theta}{2}\right)|0\rangle + e^{i\phi}\sin\left(\frac{\theta}{2}\right)|1\rangle$$
  where $\theta \in [0, \pi]$ (polar angle) and $\phi \in [0, 2\pi)$ (azimuthal phase angle). Any pure state is a point on the surface of a unit sphere in $\mathbb{R}^3$.

### Hilbert Space & Dimensional Scaling
- An $n$-qubit register does not live in $\mathbb{R}^n$; it lives in the **tensor product space**:
  $$\mathcal{H} = \mathbb{C}^2 \otimes \mathbb{C}^2 \otimes \cdots \otimes \mathbb{C}^2 = \left(\mathbb{C}^2\right)^{\otimes n} \cong \mathbb{C}^{2^n}$$
- **Exponential Dimensionality:**
  - For $n = 1$ qubit: $\dim = 2^1 = 2$
  - For $n = 4$ qubits: $\dim = 2^4 = 16$
  - For $n = 8$ qubits (**Quantinel's implementation**): $\dim = 2^8 = \mathbf{256}$
  - For $n = 50$ qubits: $\dim \approx 1.12 \times 10^{15}$ (beyond the RAM capacity of any supercomputer on Earth).
- **Key Takeaway for Mentor:** In Quantinel, 8 classical normalized features are mapped into a **256-dimensional complex projective Hilbert space**, where non-linear boundaries between attack families become hyperplanes.

### Quantum Gates: Single-Qubit & Multi-Qubit
Quantum gates are represented by **unitary operators** ($U^\dagger U = I$), meaning state transformations are reversible and preserve norm (probability conservation):
1. **Hadamard Gate ($H$):** Creates an equal superposition from basis states:
   $$H = \frac{1}{\sqrt{2}}\begin{bmatrix} 1 & 1 \\ 1 & -1 \end{bmatrix}, \quad H|0\rangle = \frac{|0\rangle + |1\rangle}{\sqrt{2}} = |+\rangle$$
2. **Pauli-Z Gate ($Z$):** Flips the phase of $|1\rangle$:
   $$Z = \begin{bmatrix} 1 & 0 \\ 0 & -1 \end{bmatrix}, \quad Z|0\rangle = |0\rangle, \quad Z|1\rangle = -|1\rangle$$
3. **Rotation Gate ($R_z(\lambda)$):** Rotates the state vector around the Z-axis of the Bloch sphere by angle $\lambda$:
   $$R_z(\lambda) = \exp\left(-i\frac{\lambda}{2}Z\right) = \begin{bmatrix} e^{-i\lambda/2} & 0 \\ 0 & e^{i\lambda/2} \end{bmatrix}$$
4. **Controlled-NOT (CNOT) Gate:** A 2-qubit entangling gate. Flips the target qubit if and only if the control qubit is $|1\rangle$:
   $$\text{CNOT} = \begin{bmatrix} 1 & 0 & 0 & 0 \\ 0 & 1 & 0 & 0 \\ 0 & 0 & 0 & 1 \\ 0 & 0 & 1 & 0 \end{bmatrix}$$
5. **$R_{zz}(\theta)$ Gate (Two-Qubit Phase Coupling):** Implements phase rotation conditioned on the parity of two qubits:
   $$R_{zz}(\theta) = \exp\left(-i \frac{\theta}{2} (Z \otimes Z)\right)$$
   Constructed physically in Qiskit using: $\text{CNOT}(q_i, q_j) \rightarrow R_z(\theta) \text{ on } q_j \rightarrow \text{CNOT}(q_i, q_j)$.

### Quantum Entanglement & Non-Locality
- **Definition:** A state of a composite quantum system $|\psi_{AB}\rangle$ is entangled if it **cannot** be factored into tensor products of its individual subsystem states:
  $$|\psi_{AB}\rangle \neq |\psi_A\rangle \otimes |\psi_B\rangle$$
- **Example (Bell State / EPR Pair):**
  $$|\Phi^+\rangle = \frac{|00\rangle + |11\rangle}{\sqrt{2}}$$
  Neither qubit has a definite state on its own. Measuring qubit A instantaneously determines the measurement outcome of qubit B, regardless of distance.
- **Role in Quantinel:** Entanglement is the mechanism that encodes **cross-feature non-linear correlations** (e.g., between `dst_host_diff_srv_rate` and `count`). Without entanglement, the quantum state is separable, and the kernel would simply reduce to independent single-feature products with no quantum representational advantage over classical additive kernels.

---

# 2. Quantum Machine Learning (QML) Architecture

### The Data Embedding (Feature Map) Problem
In classical ML, a feature vector $x \in \mathbb{R}^d$ consists of numbers. A quantum computer cannot directly process continuous numbers; it must encode them into quantum states:
$$\phi: \mathcal{X} \subset \mathbb{R}^d \longrightarrow \mathcal{H}, \quad x \mapsto |\Phi(x)\rangle$$
This is executed by applying a parameterized unitary circuit $U_\Phi(x)$ to the initial ground state $|0\rangle^{\otimes n}$:
$$|\Phi(x)\rangle = U_\Phi(x) |0\rangle^{\otimes n}$$

### Types of Quantum Encodings
1. **Basis Encoding:** Encodes binary numbers into computational basis states ($x = 5 = 101_2 \Rightarrow |101\rangle$). Inefficient for continuous network metrics.
2. **Amplitude Encoding:** Encodes normalized vector into state amplitudes ($x = [x_1, \dots, x_N]^T \Rightarrow \sum x_i |i\rangle$). Requires $O(\log N)$ qubits but requires complex $O(N)$ depth state-preparation circuits that suffer from high noise on NISQ devices.
3. **Angle Encoding:** Rotates single qubits around an axis ($x_i \mapsto R_y(x_i)|0\rangle$). Simple, but cannot capture cross-feature interactions without explicit entangling gates.
4. **Hamiltonian / Quantum Feature Maps (ZZFeatureMap):** Encodes both individual features and pairwise feature products into single-qubit and two-qubit rotation phases through an entangling unitary evolution. **This is what Quantinel uses.**

### Deep Dive: ZZFeatureMap Circuit & Mathematics
Quantinel utilizes Qiskit's `ZZFeatureMap` with $n=8$ features and depth $\text{reps}=1$:
$$U_{\Phi(x)} = \left[ \prod_{(i, j) \in E} U_{ZZ}(x_i, x_j) \prod_{i=1}^n U_Z(x_i) H^{\otimes n} \right]^d$$

#### Mathematical Phase Functions:
1. **First-Order (Single-Qubit) Rotation:**
   $$\phi_{\{i\}}(x) = 2 x_i$$
   Implements $R_z(2x_i)$ after a layer of Hadamard gates $H^{\otimes 8}$. This embeds the normalized classical value of feature $i$.
2. **Second-Order (Pairwise Entangling) Interaction:**
   $$\phi_{\{i, j\}}(x) = 2(\pi - x_i)(\pi - x_j)$$
   Applied through $R_{zz}(\phi_{\{i, j\}}(x))$ between connected qubits.
   - Notice the non-linear interaction term: if both $x_i$ and $x_j$ approach $\pi$, the phase shift is small; if both deviate, the cross-phase interaction is magnified.

#### Circuit Steps for Quantinel's 8-Qubit ZZFeatureMap:
1. Initialize register: $|0\rangle^{\otimes 8} = |00000000\rangle$.
2. Apply Hadamard layer: $H^{\otimes 8} \Rightarrow$ uniform superposition of all 256 basis states.
3. Apply single-qubit phase shifts: $\bigotimes_{i=1}^8 R_z(2x_i)$.
4. Apply entangling two-qubit interactions for all pairs $(i, j)$:
   - $\text{CNOT}(q_i, q_j)$
   - $R_z(2(\pi - x_i)(\pi - x_j))$ on target $q_j$
   - $\text{CNOT}(q_i, q_j)$
5. The resulting state $|\Phi(x)\rangle$ is an entangled 256-dimensional vector capturing both individual network flow properties and non-linear inter-feature dependencies.

### Computational Hardness & Classical Intractability
Why not just compute this non-linear interaction classically?
- Havlíček et al. (Nature 2019) and Bremner, Montanaro, & Shepherd proved that computing the transition probabilities / state overlaps of circuits of the $ZZ$ family (related to Instantaneous Quantum Polynomial-time / IQP circuits) is **$\#P$-hard** to approximate classically in the worst case under standard complexity conjectures.
- Therefore, evaluating this specific kernel on a quantum architecture allows exploration of a high-dimensional feature space that classical polynomial-time algorithms cannot generally compute directly.

---

# 3. Quantum Kernel Methods & QSVC

### The Classical vs. Quantum Kernel Trick
- **Classical Kernel Trick:** Maps vectors $x, x' \in \mathbb{R}^d$ into a high-dimensional Hilbert space $\mathcal{V}$ via $\phi(x)$, computing the inner product via a closed-form kernel function:
  $$K(x, x') = \langle \phi(x), \phi(x') \rangle_{\mathcal{V}}$$
  Example (RBF Kernel): $K(x, x') = \exp(-\gamma ||x - x'||^2)$.
- **Quantum Kernel Trick:** The mapping is directly performed by quantum state preparation $\phi(x) \mapsto |\Phi(x)\rangle$. The kernel evaluates the **quantum inner product (fidelity)**:
  $$K_Q(x, x') = |\langle \Phi(x) | \Phi(x') \rangle|^2$$

### Quantum State Fidelity as an Inner Product
- In quantum mechanics, the overlap $|\langle \Phi(x) | \Phi(x') \rangle|^2$ represents the **transition probability** (fidelity) between the two quantum states:
  - If $x = x'$: $|\Phi(x)\rangle = |\Phi(x')\rangle \implies K_Q(x, x') = 1$ (states are identical).
  - If $x$ and $x'$ produce orthogonal quantum states: $K_Q(x, x') = 0$ (maximum dissimilarity).
  - For arbitrary inputs: $0 \le K_Q(x, x') \le 1$.
- Thus, $K_Q$ forms a valid, positive semi-definite **Gram matrix** (satisfying Mercer's Theorem) that can be directly passed to a Support Vector Machine.

### Compute-Uncompute Protocol
How does a quantum computer physically measure $|\langle \Phi(x) | \Phi(x') \rangle|^2$?
1. Start with ground state $|0\rangle^{\otimes n}$.
2. Apply the feature map for vector $x$: $U_\Phi(x)|0\rangle = |\Phi(x)\rangle$.
3. Apply the **inverse (Hermitian conjugate)** feature map for vector $x'$: $U^\dagger_\Phi(x')$.
4. The composite quantum state before measurement is:
   $$|\psi_{\text{final}}\rangle = U^\dagger_\Phi(x') U_\Phi(x) |0\rangle^{\otimes n}$$
5. Measure all qubits in the computational basis ($Z$-basis).
6. The probability of observing the all-zero outcome $|00\dots0\rangle$ is:
   $$P(|00\dots0\rangle) = |\langle 0^{\otimes n} | U^\dagger_\Phi(x') U_\Phi(x) | 0^{\otimes n} \rangle|^2 = |\langle \Phi(x') | \Phi(x) \rangle|^2 = K_Q(x, x')$$
- In Quantinel, this is implemented cleanly via `ComputeUncompute(sampler=StatevectorSampler())` from `qiskit_machine_learning.state_fidelities`.

### Quantum Support Vector Classifier (QSVC) Dual Optimization
Once the $N \times N$ quantum kernel matrix $K_Q$ is computed, the training optimization is solved using the standard classical **SVM dual formulation**:
$$\max_{\alpha} \sum_{i=1}^N \alpha_i - \frac{1}{2} \sum_{i=1}^N \sum_{j=1}^N \alpha_i \alpha_j y_i y_j K_Q(x_i, x_j)$$
subject to:
$$0 \le \alpha_i \le C, \quad \sum_{i=1}^N \alpha_i y_i = 0$$
- **Decision Function for New Sample $x_{\text{test}}$:**
  $$f(x_{\text{test}}) = \text{sign}\left( \sum_{i \in \text{Support Vectors}} \alpha_i y_i K_Q(x_i, x_{\text{test}}) + b \right)$$
- **Hybrid Division of Labor:**
  - **Quantum's Job:** Compute non-linear kernel evaluations $K_Q(x_i, x_j)$ in $256$-dimensional Hilbert space.
  - **Classical's Job:** Solve convex Quadratic Programming (QP) to find dual support vector weights $\alpha_i$ and intercept $b$.

---

# 4. Quantinel Pipeline Engineering & Design Decisions

### Why a Two-Stage Hybrid Architecture?
A single model cannot solve modern network security:
1. **Scale Problem:** Network backbones process millions of packets per minute. Quantum simulators (and real QPUs) cannot evaluate 100,000 kernel entries in real-time.
2. **Zero-Day Characterization Problem:** Supervised multi-class classifiers fail when an unseen attack arrives because its specific class label does not exist in training.
3. **Quantinel's Solution:**
   - **Stage 1 (Classical Filtering):** High-throughput Random Forest / XGBoost model classifies traffic as **Normal vs. Malicious**. (Filters out 80%+ benign traffic).
   - **Stage 2 (Quantum Characterization):** Flagged malicious intrusions are forwarded to the QSVC kernel engine, which maps the intrusion to the **closest behavioral attack family** (DoS, Probe, R2L, U2R).

```
Network Traffic
      │
      ▼
┌────────────────────────────────────────┐
│ Stage 1: Classical IDS (Random Forest) │
└────────────────────────────────────────┘
      │
      ├── Benign ─────────► Allowed / Logged
      │
      └── Intrusion (Malicious)
            │
            ▼
┌────────────────────────────────────────┐
│ Feature Selection (Top 8 Informative)  │
└────────────────────────────────────────┘
            │
            ▼
┌────────────────────────────────────────┐
│ Stage 2: Quantum Characterization      │
│   (ZZFeatureMap + QSVC Kernel)         │
└────────────────────────────────────────┘
            │
            ▼
Attack Family Assigned: [DoS | Probe | R2L | U2R]
(Enables tailored defensive countermeasures even for Zero-Day attacks!)
```

### Why 8 Qubits? (NISQ Era Realities)
- In the Noisy Intermediate-Scale Quantum (NISQ) regime, qubit counts and gate depths are constrained by decoherence times and two-qubit gate fidelities.
- In software simulation, simulating $n$ qubits requires storing $2^n$ complex numbers ($2^n \times 16$ bytes).
- 8 qubits represents the sweet spot:
  - $2^8 = 256$-dimensional Hilbert space (sufficiently expressive to capture complex attack boundaries).
  - Fast, stable, reproducible statevector simulation without out-of-memory crashes during batch evaluation across 12,000+ test samples.

### The 3-Tier Feature Selection Strategy
Raw NSL-KDD contains 41 features. Passing 41 features into a quantum circuit would require 41 qubits and thousands of entangling gates. To distill this down to the **8 most discriminative features**, Quantinel built a systematic 3-tier pipeline:
1. **Tier 1: Variance Thresholding (`VarianceThreshold(0.01)`)**
   - Eliminates near-constant features (features where 99%+ of samples have the identical value).
2. **Tier 2: Collinearity Filtering ($|r| > 0.85$)**
   - Computes Pearson correlation matrix; removes redundant features that carry duplicate mutual information.
3. **Tier 3: Mutual Information (`mutual_info_classif`)**
   - Ranks remaining features by non-linear dependency with the attack family target.
   - **The Top 8 Selected Features:**
     1. `dst_host_diff_srv_rate` (Rate of connections to different services on destination host)
     2. `count` (Number of connections to same destination host in past 2 seconds)
     3. `diff_srv_rate` (% of connections to different services)
     4. `flag_S0` (SYN error flag — indicative of SYN flood DoS)
     5. `same_srv_rate` (% of connections to same service)
     6. `dst_host_same_src_port_rate` (% of connections from same source port)
     7. `srv_count` (Number of connections to same service in past 2 seconds)
     8. `dst_host_same_srv_rate` (% of connections to same service on destination host)

### Dataset A (3 Classes) vs. Dataset B (4 Classes / U2R Imbalance)
- **Dataset A:** Focuses on the 3 dominant attack families: **DoS, Probe, R2L** (120 samples per class = 360 balanced samples).
- **Dataset B:** Includes all 4 attack families: **DoS, Probe, R2L, and U2R** (52 samples per class = 208 balanced samples).
- **The U2R Constraint:** In the entire NSL-KDD training set of 125,973 records, only **52 samples** belong to User-to-Root (U2R)! To prevent the model from ignoring rare attacks due to severe class imbalance, Dataset B balances all classes at 52 samples.

### Closed-World vs. Open-World Evaluation Dynamics
- **Closed-World Evaluation (`4a`):**
  - Evaluates models on known attack types (split from the training distribution).
  - **Dataset A Accuracy: 89.81% (0.90 macro F1)**
  - **Dataset B Accuracy: 85.48%**
  - Proves that the 8-qubit ZZ-feature map and quantum kernel cleanly separate known attack families in Hilbert space.
- **Open-World Evaluation (`4b`):**
  - Evaluates models on the real test set containing **17 completely unseen, zero-day attack variants** (e.g., *apache2, mailbomb, snmpget, mscan, saint*).
  - Evaluated on 12,764 actual attack traffic samples.
  - **Dataset A Accuracy: 78.72%**
    - DoS: 94.37% Precision, 76.82% Recall ($F_1 = 0.8469$)
    - Probe: 72.03% Precision, 85.42% Recall ($F_1 = 0.7816$)
    - R2L: 58.90% Precision, 78.02% Recall ($F_1 = 0.6712$)
  - **Dataset B Accuracy: 77.19%**
    - Successfully characterizes zero-day U2R attacks with 28.99% recall despite having only 52 training samples!

### The 34.58% vs. 78.72% Accuracy Distinction
> [!IMPORTANT]
> **This is the single most critical slide/talking point in your presentation!**
- In `4b (complete test data).ipynb`, accuracy is reported as **34.58%**.
- In `4b.ipynb`, accuracy is reported as **78.72%**.
- **Why the difference?**
  - In `complete test data`, all 22,507 records of the test set were passed in, including **13,461 normal traffic samples** artificially labeled as `"unknown"`.
  - However, QSVC is an **attack characterizer**, trained exclusively on attack family labels (`dos`, `probe`, `r2l`, `u2r`). It does not possess an "unknown / benign" output neuron.
  - As a result, all normal samples were forced into attack classes, getting 0% precision and dragging accuracy down to 34.58%.
  - When evaluated on **actual intrusions** (the true operational scope of Stage 2), the QSVC correctly maps zero-day attacks to their true behavioral families with **78.72% accuracy**!
  - **Conclusion:** This proves that QSVC cannot replace the classical IDS; it must operate in tandem as Stage 2 of the Quantinel pipeline.

---

# 5. Mentor Defense Q&A (Anticipated Tough Questions)

### Q1: "Why use a Quantum Feature Map instead of a classical RBF or Polynomial kernel?"
**Answer:**  
"Classical RBF kernels compute similarity based solely on Euclidean distance in the input space: $K(x, x') = \exp(-\gamma ||x - x'||^2)$. However, network intrusion behavior is governed by non-linear correlations between distinct features (e.g., a high connection count combined with a specific TCP error flag). The `ZZFeatureMap` generates explicit two-qubit phase interactions $\phi_{\{i,j\}}(x) = 2(\pi - x_i)(\pi - x_j)$ via entangling CNOT gates. This maps data into a 256-dimensional Hilbert space where cross-feature entanglements capture higher-order relationships that are classically hard to compute (IQP circuit complexity)."

### Q2: "Why did Centroid-based Cosine Similarity fail (37.5% accuracy) in Notebook 03?"
**Answer:**  
"Centroid-based cosine similarity computes a single mean vector for each attack family and measures the angle against that mean. Network attack families are multimodal: for example, the DoS family contains both volumetric flood attacks (high packet counts) and stealthy protocol-abuse attacks (normal packet counts, anomalous flags). Averaging them into a single centroid washes out the distinctive signatures. Furthermore, cosine similarity measures only angular direction, ignoring feature magnitude."

### Q3: "Why did KNN perform best at K=1 in Notebook 03?"
**Answer:**  
"In our experiments, increasing $K$ caused accuracy to degrade because attack families in NSL-KDD suffer from severe class imbalance (DoS and Probe heavily dominate). When $K$ is increased (e.g., $K=5$ or $K=9$), the large majority classes overwhelm the localized neighborhoods of rare attacks like R2L and U2R. $K=1$ works best classically because an unseen attack behaves most similarly to its closest individual behavioral ancestor."

### Q4: "Why is ROC-AUC 0.96 for Random Forest in Notebook 02, but accuracy is only 77%?"
**Answer:**  
"ROC-AUC evaluates ranking capability across all possible decision thresholds. The score of 0.9624 demonstrates that the model successfully assigns higher anomaly probabilities to attack packets than to normal packets. However, the test dataset contains 17 zero-day attacks with subtle evasion characteristics. At the fixed default decision threshold of 0.5, these novel attacks fall just below the decision boundary, reducing recall and capping classification accuracy at 77%."

### Q5: "How did you solve the scalability problem of evaluating 12,000+ test samples in QSVC?"
**Answer:**  
"Evaluating a full kernel matrix of $12,764 \times 12,764$ would require over 162 million quantum circuit simulations, which causes out-of-memory exceptions. We implemented a batched evaluation pipeline (5,000 samples per batch) that computes rectangular test-to-train kernel sub-matrices against only the identified support vectors ($|\text{SV}| = 178$ for Dataset A). This reduced the required quantum circuit evaluations by several orders of magnitude, making large-scale open-world evaluation computationally feasible."

### Q6: "Can this model be deployed on real quantum hardware like IBM Quantum Eagle or Heron?"
**Answer:**  
"Yes. The circuit was designed specifically with NISQ hardware constraints in mind:
1. It uses only **8 qubits**, well within current hardware capacity (127+ qubits).
2. It uses `reps=1` with linear nearest-neighbor entanglement, keeping circuit depth shallow and minimizing CNOT count.
3. On real QPUs, error mitigation techniques like Zero-Noise Extrapolation (ZNE) and readout error mitigation would be applied to counter decoherence."
