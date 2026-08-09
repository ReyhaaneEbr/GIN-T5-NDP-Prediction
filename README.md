# 🧠 NDP_Prediction
Integrating Protein Sequence Embeddings and PPI Networks for Neurodegenerative Disease Protein Discovery (GIN-T5)
---
##📌 Overview
GIN-T5 is a graph-based deep learning framework designed to identify Neurodegenerative Disease–associated Proteins (NDPs) by jointly integrating protein–protein interaction (PPI) network topology and protein sequence-derived embeddings.

Neurodegenerative diseases (NDs) are characterized by progressive neuronal dysfunction and abnormal protein aggregation. While experimental identification of disease-associated proteins is costly and time-consuming, existing computational approaches largely rely on handcrafted descriptors or PPI network topology alone. To address these limitations, GIN-T5 combines Graph Isomorphism Networks (GIN) to model PPI structure and ProtT5 protein language model embeddings to capture sequence-level biological information, effectively supporting both connected and isolated protein nodes.

The framework focuses on four major neurodegenerative diseases:

Alzheimer’s Disease (AD)

Parkinson’s Disease (PD)

Huntington’s Disease (HD)

Amyotrophic Lateral Sclerosis (ALS)

##🧪 Data Collection & Labeling
###🧬 Disease-Associated Proteins

Curated from UniProtKB for AD, PD, HD, and ALS

Positive set (S 
pos
​
 ): 3,081 confirmed NDPs

🧫 Housekeeping Proteins

Selected from stable housekeeping genes as negative controls

Negative set (S 
neg
​
 ): 1,738 non-NDPs

Labels: 1 → Disease-associated protein (NDP), 0 → Housekeeping protein (non-NDP)

### 🌐 PPI Network & Pharmacological Data

Connected Nodes (V 
connected
​
 ): 4,925 proteins from the Integrated Interactions Database (IID)

Total Nodes (V 
total
​
 ): 6,081 proteins (includes isolated nodes lacking PPI interactions)

Drug Datasets: RepoDB (drug–disease links) and BindingDB (IC 
50
​
  binding affinities)
---
## ▦ Feature Representation
###🧫 Phase I Representation Selection

Systematically benchmarked handcrafted sequence features (BLOSUM62, Dipeptide composition), network topological metrics (18 graph descriptors, Word2Vec, Node2Vec), and protein language models (ESM2, ProtBERT, ProtT5) using an MLP baseline.

Identified ProtT5 (1024-dimensional) as the optimal sequence representation.

###🧩 Graph Construction & Node Handling

Nodes: Represent proteins, all initialized with 1024-dimensional ProtT5 embeddings.

Edges: Represent PPI interaction edges from IID.

Node Handling: Connected proteins update features by aggregating neighborhood context during message passing; isolated proteins retain native ProtT5 sequence embeddings, preserving biological utility without network edges.
---
## 🏛️ Model Architecture
GIN-T5 evaluates four message-passing operators (GCN, GAT, GraphSAGE, GIN) and selects GIN due to its injective sum-aggregation scheme and internal MLP parameterization:

Input layer: 1024-dimensional ProtT5 sequence embeddings

Stage 1 GINConv: 1024→256 dimensions (internal 2-layer MLP, ReLU, Dropout 0.5)

Stage 2 GINConv: 256→64 dimensions (internal 2-layer MLP, ReLU, Dropout 0.5)

Output layer: Linear classifier mapping 64 latent features →2 target classes

Interpretability: Integrated GNNExplainer highlights disease-informative subgraphs and sequence attributes
---
##⚙️ Training Pipeline
🧬 Extract position-averaged 1024-d ProtT5 protein sequence embeddings (Rostlab/prot_t5_xl_half_uniref50)

🌐 Construct the IID-based PPI graph supporting connected (V 
connected
​
 ) and disconnected (V 
total
​
 ) nodes

📉 Perform Phase I feature representation benchmarking

📈 Train GIN-T5 model for 50 epochs using Adam optimizer (lr=0.001)

⚖️ Apply Weighted Cross-Entropy Loss (class weights: 2.0 negative, 1.0 positive) to handle class imbalance

🔍 Extract disease-informative interaction subgraphs via GNNExplainer

---

## 📊 Evaluation Metrics
Model performance is evaluated across 5-fold cross-validation using:

✅ Accuracy

✅ Precision

✅ Recall

✅ F1-score

✅ ROC-AUC

✅ AUPRC

---

##💊 Drug Repurposing Analysis
Using the predicted ND-associated proteins:

- Candidate proteins were mapped to known drugs in RepoDB and filtered by binding affinity (IC 
50
​
 <1μM) in BindingDB to identify prioritized candidate NDPs

- Candidate proteins with well-documented interactions are supported by DrugBank Supported Pairs (DSPs)
Other candidate proteins are reported only in BindingDB, representing novel and underexplored targets for therapeutic drug repurposing.
