# STGSCCA
STGSCCA: Spatiotemporal Graph Contrastive Learning with Dual-Graph Regularized Sparse Canonical Correlation Analysis
Official PyTorch implementation of the paper:

"Spatiotemporal Graph Contrastive Learning with Dual-Graph Regularized Sparse Canonical Correlation Analysis for Longitudinal Imaging-Multi-Omics Integration in Early Mild Cognitive Impairment"

1. Overview
STGSCCA is a two-stage computational framework designed for longitudinal neuroimaging-genetics integration:
1. Stage 1 — Front-End Spatiotemporal Graph Contrastive Learning (ST-GCL):

Constructs a brain Morphological Similarity Network (MSN) and employs a multi-layer Graph Convolutional Network (GCN) coupled with a Temporal Dynamics Self-Attention module. Under self-supervised InfoNCE contrastive learning and anatomical reconstruction constraints, it purifies noisy multi-timepoint longitudinal sMRI sequences (T1∼T4) into a unified subject-level spatiotemporal representation matrix.
2. Stage 2 — Back-End Dual-Graph Regularized Multi-Task SCCA (DG-MTSCCA):

Performs joint multi-task sparse canonical correlation analysis between the purified longitudinal neuroimaging representation and baseline multi-omics profiles (SNPs and DNA methylation). It incorporates L1-norm element sparsity, L2,1-norm group sparsity across tasks, and dual-graph Laplacian regularization to preserve endogenous linkage disequilibrium (LD) and epigenetic co-expression topologies.





2. Repository Structure
STGSCCA/
├── data/                       # Input directory containing synthetic demo CSV files
│   ├── MRI_T1.csv              # Baseline sMRI gray matter volumes (N x 90 ROIs)
│   ├── MRI_T2.csv              # 6-month follow-up sMRI volumes (N x 90 ROIs)
│   ├── MRI_T3.csv              # 12-month follow-up sMRI volumes (N x 90 ROIs)
│   ├── MRI_T4.csv              # 24-month follow-up sMRI volumes (N x 90 ROIs)
│   ├── SNP.csv                 # Baseline SNP genotype matrix (N x P1)
│   └── Meth.csv                # Baseline DNA methylation beta values (N x P2)
├── results/                    # Output directory (automatically created during execution)
├── config.py                   # Global configuration, paths, and model hyperparameters
├── dataset.py                  # Multi-modal subject alignment and PyG dataset wrapper
├── utils.py                    # Morphological similarity network (MSN) graph construction
├── model.py                    # Front-end ST-GCL architecture (GCN + Temporal Attention)
├── train.py                    # Self-supervised ST-GCL training & MRI fusion export
├── run_scca.py                 # Back-end DG-MTSCCA nested CV & biomarker ranking
├── requirements.txt            # Python package dependencies
└── README.md                   # Project documentation

3. Environment Setup
Clone this repository and install the required dependencies (Python 3.8+ and PyTorch 1.13+ recommended):
git clone https://github.com/Jindsmu/STGSCCA.git
cd STGSCCA
pip install -r requirements.txt
Core Dependencies:
• numpy>=1.21.0
• pandas>=1.3.0
• scikit-learn>=1.0.2
• matplotlib>=3.5.0
• torch>=1.13.0
• torch-geometric>=2.2.0

4. Data Availability & Format Specification
Data Availability Statement
The real clinical neuroimaging and multi-omics datasets analyzed in our study were obtained from the Alzheimer's Disease Neuroimaging Initiative (ADNI) database (https://adni.loni.usc.edu/). Due to ADNI data use agreements and patient privacy regulations, raw subject-level clinical records cannot be redistributed publicly. Qualified researchers can apply for access directly through the ADNI LONI portal.
Built-in Synthetic Demo Data
To ensure out-of-the-box reproducibility and allow users to verify the complete computational pipeline immediately, we provide a set of synthetic benchmark data inside the ./data/ folder (MRI_T1.csv to MRI_T4.csv, SNP.csv, and Meth.csv) with matching feature dimensions (90 cerebral ROIs and 200 omics loci per modality).
Using Custom Datasets
If you wish to run STGSCCA on your own cohort or preprocessed ADNI data, place your 6 CSV files in ./data/ adhering to the following format:
• First Row (Header): Feature/biomarker names (e.g., SubjectID, rPCu, lInfFro, ... or SubjectID, rs405509, ...).
• First Column (SubjectID): Unique subject identifier. dataset.py and run_scca.py automatically take the intersection of SubjectID across all 6 files and sort them in strict alignment.
• Feature Columns: Numerical values (90 columns for MRI_T1.csv–MRI_T4.csv matching ROI_COUNT = 90 in config.py).

5. Quick Start & Usage Pipeline
Step 1: Front-End Spatiotemporal Representation Learning (ST-GCL)
Run train.py to build the training-set Morphological Similarity Network (MSN), train the self-supervised spatiotemporal graph contrastive model across the 4 follow-up timepoints, and export the fused subject-level neuroimaging matrix:
python train.py
Stage 1 Outputs (saved to ./results/):
• fusion_model.pth: Trained ST-GCL network weights.
• edge_index.pt: Constructed ROI morphological similarity graph adjacency list.
• train_loss.png: Self-supervised training convergence curve.
• MRI_Fused.csv: Purified subject-level spatiotemporal neuroimaging matrix (Z∈RN×90) with original anatomical ROI headers preserved.

Step 2: Back-End Multi-Omics Association & Biomarker Prioritization (DG-MTSCCA)
Once MRI_Fused.csv has been generated, run run_scca.py to execute the nested 5-fold cross-validation and export the global canonical weights:
python run_scca.py
Stage 2 Outputs (saved to ./results/):
• Console Evaluation: Reports out-of-sample 5-fold cross-validated Test CCC (Mean±SD) for both SNP-MRI and DNAm-MRI association tasks.
• Results_Top_SNPs.csv: Ranked canonical weight magnitudes (∣u1∣) for all SNP loci.
• Results_Top_DNAm.csv: Ranked canonical weight magnitudes (∣u2∣) for all DNA methylation CpG sites.
• Results_Top_MRI.csv: Ranked joint canonical weights (∥vj∥2) across tasks for all 90 brain ROIs.
