# EvoStruct-Sub: Gram-Positive Protein Subcellular Localization

## What This Project Does

This project predicts the subcellular location of proteins in gram-positive bacteria using machine learning classification. It takes amino acid sequences as input and predicts which of four subcellular compartments (cytoplasm, membrane, periplasm, extracellular) each protein is likely to be found in. The project combines evolutionary information from PSI-BLAST searches with structural features computed by SPIDER 2.0 to create a rich feature representation of each protein. Random Forest achieves 99.23% classification accuracy, demonstrating that combining multiple complementary features and ensemble methods can solve this challenging biological prediction problem.

## Can I Run This?

**YES – Classification only** (recommended)
- No external tools required
- Pre-extracted features are included
- Pre-trained models included (load `.pkl` files directly)
- ~10 minutes to run all classifiers

**PARTIAL – Full pipeline** (advanced)
- Requires PSI-BLAST (NCBI BLAST+) and SPIDER 2.0
- Requires ~500MB for Swiss-Prot database
- Computationally intensive (several hours)
- Only needed to reproduce from raw sequences

## Quick Start – Classification Only

### Prerequisites
- Python 3.8 or higher
- pip package manager

### Installation
```bash
pip install -r requirements.txt
```

### Run Classification (in this order)
1. **notebooks/data_processing.ipynb** — Merges all feature files into a single training matrix
2. **notebooks/knn.ipynb** — K-Nearest Neighbors classifier (77.74% accuracy)
3. **notebooks/random_forest.ipynb** — Random Forest classifier (99.23% accuracy)
4. **notebooks/svm_model.ipynb** — Support Vector Machine classifier (79.08% accuracy)

Or load pre-trained models without retraining:
```python
import joblib
model = joblib.load('models/rf_final_model.pkl')  # or knn_final_model.pkl, svm_final_model.pkl
predictions = model.predict(X_test)
```

## Key Results

| Model | Accuracy |
|-------|----------|
| Random Forest | 99.23% |
| SVM | 79.08% |
| KNN | 77.74% |

## Project Structure

The project is organized into logical folders for easy navigation:

### `data/` — Raw Data
- **g_data.csv** — Raw protein sequences (523 gram-positive bacterial proteins, 4 location classes)
- **g_data.fasta** — FASTA-formatted sequences. Input for PSI-BLAST external tool.
- **n_data.csv** — Alternative dataset (if available)
- **n_data.fasta** — FASTA version of alternative dataset

### `features/` — Extracted Features (Pre-computed)
- **pssm_features.csv** — Position-Specific Scoring Matrix features from PSI-BLAST (200 features per protein)
- **composition_features.csv** — Amino acid composition features (4 features per protein)
- **auto_cov_features.csv** — Auto-covariance structural features from SPIDER 2.0 (200 features per protein)
- **one_lead_bigram_asa.csv** — One-lead bigram Accessible Surface Area features (1 feature per protein)
- **merged_df.csv** — **Final feature matrix** combining all extracted features. This is the input to all classifiers (405 features × 523 proteins)

### `notebooks/` — Jupyter Notebooks
- **data_processing.ipynb** — Merges all feature CSVs into `merged_df.csv`. Run first.
- **features_extraction.ipynb** — Extracts PSSM and SPIDER 2.0 features from raw sequences. Only run if re-extracting from scratch.
- **knn.ipynb** — K-Nearest Neighbors classifier (77.74% accuracy). Saves model to `models/knn_final_model.pkl`.
- **random_forest.ipynb** — Random Forest classifier (99.23% accuracy). **Best performing.** Saves model to `models/rf_final_model.pkl`.
- **svm_model.ipynb** — Support Vector Machine classifier (79.08% accuracy). Saves model to `models/svm_final_model.pkl`.

### `models/` — Pre-trained Models (Binary .pkl Format)
- **knn_final_model.pkl** — Trained KNN model (77.74% accuracy). Load and use immediately.
- **rf_final_model.pkl** — Trained Random Forest model (99.23% accuracy). **Best performing model.**
- **svm_final_model.pkl** — Trained SVM model (79.08% accuracy). Load and use immediately.

### `scripts/` — Utility Scripts
- **feature_extraction.R** — R script for amino acid composition feature extraction (alternative implementation)

### Root Configuration Files
- **.gitignore** — Specifies files to ignore in version control
- **README.md** — This file
- **requirements.txt** — Python package dependencies

### Root Data Folders (Kept for Reference)
- **g_data/** — Individual FASTA files for each of the 523 proteins
- **PSSM_Features/** — PSSM matrix files from PSI-BLAST (external tool output)
- **output/** — Structural prediction files from SPIDER 2.0 (external tool output)
- **file_list** — List of protein sequence file names

## Full Pipeline (Advanced – Reproduce from Scratch)

Only follow these steps if you want to re-extract features from the raw FASTA sequences. This requires external tools and is computationally intensive.

### Prerequisites
- PSI-BLAST (NCBI BLAST+) installed and in your PATH
- Swiss-Prot database (~250 MB) downloaded
- SPIDER 2.0 installed and configured
- Python 3.8+

### Step-by-Step Reproduction

**Step 1: Generate FASTA**
- Run code to parse `g_data.csv` and generate sequences in FASTA format
- Output: `g_data.fasta`

**Step 2: Run PSI-BLAST**
```bash
psiblast -query g_data.fasta -db /path/to/swiss_prot -num_iterations 3 -out_ascii_pssm output_dir/
```
- Creates PSSM files for each sequence in `PSSM_Features/gdata/` directory

**Step 3: Run SPIDER 2.0**
```bash
spider2.pl g_data.fasta output_dir/
```
- Generates structural predictions (stored in `output/` directory)

**Step 4: Extract Features**
- Run **notebooks/features_extraction.ipynb**
- Outputs: `features/pssm_features.csv`, `features/composition_features.csv`, `features/auto_cov_features.csv`, `features/one_lead_bigram_asa.csv`

**Step 5: Merge Features**
- Run **notebooks/data_processing.ipynb**
- Output: `features/merged_df.csv`

**Step 6: Train Classifiers**
- Run **notebooks/knn.ipynb**, **notebooks/random_forest.ipynb**, **notebooks/svm_model.ipynb** in any order
- Outputs: `models/knn_final_model.pkl`, `models/rf_final_model.pkl`, `models/svm_final_model.pkl`

### ⚠️ Notes
- PSI-BLAST searches are computationally expensive (1-2 hours for full dataset)
- Swiss-Prot database must be properly formatted for BLAST
- SPIDER 2.0 requires Perl and specific dependencies
- Pre-extracted features are already included; only do this if you need to modify the pipeline

## Tools Used
- **Python 3.8+** — Primary programming language
- **scikit-learn** — Machine learning algorithms (KNN, Random Forest, SVM)
- **pandas** — Data manipulation and CSV handling
- **numpy** — Numerical computations
- **joblib** — Model serialization (save/load .pkl files)
- **Jupyter** — Interactive notebook environment
- **R** — Alternative feature extraction (optional)
- **PSI-BLAST (NCBI BLAST+)** — Evolutionary profile generation (external tool)
- **SPIDER 2.0** — Secondary structure and surface area prediction (external tool)

## Dataset

**Source:** Gram-positive bacterial protein sequences (included in repo as `g_data.csv`)

**Size:** 523 protein sequences

**Classes:** 4 subcellular locations
- Cytoplasm (Fold1)
- Membrane (Fold2)
- Periplasm (Fold3)
- Extracellular (Fold4)

**Features per protein:**
- 200 PSSM features (evolutionary)
- 4 composition features (amino acid percentages)
- 200 auto-covariance features (structural)
- 1 one-lead bigram ASA feature (surface area)
- **Total: 405 features per sequence**
