# 🧠 NDP_Prediction

### Integrating Protein Sequence Embeddings and PPI Networks for Neurodegenerative Disease Protein Discovery (GIN-T5)

---

# 📌 Overview

**GIN-T5** is a graph-based deep learning framework designed to identify **Neurodegenerative Disease-associated Proteins (NDPs)** by jointly integrating:

- Protein–protein interaction (PPI) network topology
- Protein sequence-derived embeddings

Neurodegenerative diseases (NDs) are characterized by:

- Progressive neuronal dysfunction
- Abnormal protein aggregation

Experimental identification of disease-associated proteins is costly and time-consuming. Existing computational approaches often rely on:

- Handcrafted sequence descriptors
- PPI network topology alone

To address these limitations, **GIN-T5** combines:

- **Graph Isomorphism Networks (GIN)** to model PPI network structure
- **ProtT5 protein language model embeddings** to capture sequence-level biological information

This integrated framework supports prediction across both:

- Connected protein nodes within PPI networks
- Isolated protein nodes lacking interaction information

The framework focuses on four major neurodegenerative diseases:

- Alzheimer’s Disease (AD)
- Parkinson’s Disease (PD)
- Huntington’s Disease (HD)
- Amyotrophic Lateral Sclerosis (ALS)

---

# 🧪 Data Collection & Labeling

## 🧬 Disease-Associated Proteins

- Curated from **UniProtKB** for:
  - Alzheimer’s Disease (AD)
  - Parkinson’s Disease (PD)
  - Huntington’s Disease (HD)
  - Amyotrophic Lateral Sclerosis (ALS)

### Positive Set

- **3,081 confirmed NDPs**

## 🧫 Housekeeping Proteins

- Selected from stable housekeeping genes as negative controls

### Negative Set

- **1,738 non-NDPs**

### Labels

- `1` → Disease-associated protein (NDP)
- `0` → Housekeeping protein (non-NDP)

---

# 🌐 PPI Network & Pharmacological Data

## 🕸️ PPI Network Construction

- Protein–protein interaction data were obtained from the **Integrated Interactions Database (IID)**

Network statistics:

- Connected nodes (**V_connected**): **4,520 proteins**
  - Includes:
    - 3,020 NDP-associated proteins
    - 710 non-NDP proteins
    - 455 drug-target proteins
    - 335 additional interacting proteins

- Total nodes (**V_total**): **6,421 proteins**
  - Includes connected proteins and isolated proteins without documented PPI interactions

---

# ▦ Feature Representation

## 🧫 Phase I: Representation Benchmarking

Multiple feature representations were systematically evaluated using an MLP baseline:

### Sequence-Based Features

- BLOSUM62
- Dipeptide composition
- Protein language models:
  - ESM2
  - ProtBERT
  - ProtT5

### Network-Based Features

- 18 graph topology descriptors
- Word2Vec
- Node2Vec

### Selected Representation

- **ProtT5 embeddings**
- 1024-dimensional protein sequence representation

---

# 🧩 Graph Construction & Node Handling

## Graph Definition

- Nodes represent proteins
- Edges represent PPI interactions from IID
- Node features are initialized with 1024-dimensional ProtT5 embeddings

## Handling Connected and Isolated Proteins

- Connected proteins:

  - Update representations by aggregating neighborhood information during message passing

- Isolated proteins:

  - Retain native ProtT5 sequence embeddings
  - Preserve biological information without requiring network connections

---

# 🏛️ Model Architecture

**GIN-T5** evaluates multiple graph neural network operators:

- GCN
- GAT
- GraphSAGE
- GIN

The final model uses **Graph Isomorphism Network (GIN)** due to:

- Injective sum aggregation mechanism
- Internal MLP-based feature transformation

## Architecture

### Input Layer

- 1024-dimensional ProtT5 sequence embeddings

### Stage 1: GINConv

- 1024 → 256 dimensions
- Internal 2-layer MLP
- ReLU activation
- Dropout = 0.5

### Stage 2: GINConv

- 256 → 64 dimensions
- Internal 2-layer MLP
- ReLU activation
- Dropout = 0.5

### Output Layer

- Linear classifier
- 64 latent features → 2 target classes

---

# 🔍 Interpretability

GIN-T5 incorporates **GNNExplainer** to identify:

- Disease-informative subgraphs
- Important protein interactions
- Sequence-level biological attributes contributing to predictions

---

# ⚙️ Training Pipeline

The complete workflow includes:

- 🧬 Extracting position-averaged ProtT5 embeddings  
  (**Rostlab/prot_t5_xl_half_uniref50**)

- 🌐 Constructing IID-based PPI graphs

- 📉 Performing Phase I feature representation benchmarking

- 📈 Training GIN-T5:

  - Epochs: 50
  - Optimizer: Adam
  - Learning rate: 0.001

- ⚖️ Applying Weighted Cross-Entropy Loss:

  - Negative class weight: 2.0
  - Positive class weight: 1.0

- 🔍 Extracting disease-informative interaction subgraphs using GNNExplainer

---

# 📊 Evaluation Metrics

Model performance is evaluated using **5-fold cross-validation**:

- Accuracy
- Precision
- Recall
- F1-score
- ROC-AUC
- AUPRC

---

# 💊 Drug Repurposing Analysis

Using predicted ND-associated proteins:

- Candidate proteins were mapped to known drugs in **RepoDB**

- Drug candidates were filtered using **BindingDB** binding affinity:

  - IC₅₀ < 1 μM

- High-confidence therapeutic candidates were prioritized based on:

  - DrugBank Supported Pairs (DSPs)
  - Documented protein–drug interactions

- Additional candidates reported only in BindingDB represent:

  - Novel therapeutic targets
  - Underexplored opportunities for drug repurposing
