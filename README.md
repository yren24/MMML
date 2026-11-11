# MMML

**Multi-Mathematical-Modal Learning for Molecules and Materials**

## Overview

**MMML** is a learning framework that treats *mathematical diversity itself* as a source of multimodality. Instead of distinguishing modalities by data type or source, MMML derives multiple representations of the same scientific structure from distinct mathematical formulations, and pairs them with a diverse set of deep learning architectures.

The mathematical modalities are rooted in algebraic topology, combinatorial spectral theory, commutative algebra, differential geometry, and discrete curvature. Three-dimensional molecular and materials structures are first mapped to mathematical domains such as bipartite graphs, simplicial complexes (Vietoris-Rips and alpha complexes), and differentiable manifolds. Five complementary modality families are constructed from those domains, each paired with four neural architectures that span topological and conventional designs. A consensus prediction is formed by aggregating selected modality-architecture base models.

The manuscript validates MMML on four benchmarks: protein-ligand binding affinity, metal-organic framework gas uptake, mutation-induced protein solubility, and molecular toxicity. This release focuses on the MOF gas-uptake and LD50 toxicity applications, including precomputed features, final component settings, training scripts, and consensus utilities.

<p align="center">
  <img src="assets/mmml_workflow.png" alt="MMML workflow" width="78%">
</p>

The five modality families and the mathematical domains they are built on (panels B and C above):

| Code | Modality family | Mathematical domain |
| --- | --- | --- |
| `CA` | Commutative algebra (facet, F-vector) | bipartite graph |
| `PH` | Persistent homology | simplicial complex |
| `PL` | Persistent Laplacian | simplicial complex |
| `EIC` | Element interactive curvature | differentiable manifold |
| `FPRC` | Forman persistent Ricci curvature | simplicial complex |

The four neural architectures each modality is paired with (panel D above):

| Code | Architecture |
| --- | --- |
| `SNN` | Simplicial neural network |
| `CTNN` | Copresheaf transformer |
| `ANN` | Artificial neural network |
| `CNN` | Convolutional neural network |

## Repository Structure

```text
MMML/
|-- applications/
|   |-- ld50/              # LD50 toxicity component training scripts
|   `-- mof/               # MOF gas-uptake scripts and MOF CTNN model
|-- assets/                # Workflow figure used by the README
|-- configs/               # Application-level component and consensus settings
|-- data/                  # Feature manifest and expected downloaded feature layout
|-- docs/                  # Additional notes on data, features, architectures, and environment
|-- metadata/              # Machine-readable final model settings and reported metrics
|-- scripts/               # Training dispatcher, evaluation, consensus, and audit utilities
|-- src/mmml/              # Reusable architecture definitions
|-- environment.yml
|-- requirements.txt
`-- README.md
```

## Precomputed Feature Download

The released feature archives are hosted on Google Drive:

[MMML final feature archives](https://drive.google.com/drive/folders/1Y5hwU1WikPbT5pzhIL0ocmJcf9MWE1mv)

The folder is expected to contain:

```text
MITNN_MOF_final_features.zip
MITNN_LD50_final_features.zip
```

The archive file names keep the earlier `MITNN_` prefix.

A concise manifest of the released feature groups is provided in:

```text
data/feature_manifest.csv
```

The feature archives are intentionally not stored directly in the Git repository because they are large generated artifacts.

## Installation

Clone the repository:

```bash
git clone https://github.com/yren24/MMML.git
cd MMML
```

Create the recommended Conda environment:

```bash
conda env create -f environment.yml
conda activate mmml
```

Alternatively, install the Python requirements into an existing environment:

```bash
pip install -r requirements.txt
```

After installation, verify the environment with:

```bash
python scripts/check_environment.py
```

## Preparing the Data

Download the two feature archives from Google Drive and extract them from the repository root:

```bash
unzip MITNN_MOF_final_features.zip
unzip MITNN_LD50_final_features.zip
```

This should create the `data/mof/` and `data/ld50/` directories shown above. The application scripts use these compact modality family names directly: `PH`, `PL`, `CA`, `EIC`, and `FPRC`.

To verify a downloaded feature package before training:

```bash
python scripts/audit_feature_archives.py \
    --mof-zip MITNN_MOF_final_features.zip \
    --ld50-zip MITNN_LD50_final_features.zip
```

The audit checks that every label entry has the expected `.npy` feature file under the directory names used by the code.

## Training

Each component is defined by an application, a mathematical modality, and a neural architecture:

```text
application + modality + architecture
```

Examples:

```text
LD50 + EIC + CTNN
MOF O2 + CA + CNN
```

The helper dispatcher in `scripts/train.py` builds the appropriate application command and, by default, injects the final component settings recorded in `metadata/`:

```bash
python scripts/train.py \
    --application ld50 \
    --modality EIC \
    --architecture CTNN
```

MOF O2 uptake with CA-CNN:

```bash
python scripts/train.py \
    --application mof_o2 \
    --modality CA \
    --architecture CNN
```

Arguments not consumed by the dispatcher are forwarded to the application script, so component-specific settings can still be supplied:

```bash
python scripts/train.py \
    --application ld50 \
    --modality FPRC \
    --architecture SNN \
    --epochs 300 \
    --batch-size 256
```

The underlying application scripts can also be called directly from `applications/mof/` or `applications/ld50/`.

Use `--no-final-defaults` if you want the raw defaults of the application scripts instead of the final-component settings from `metadata/`.

> The earlier flag name `--invariant` is still accepted as a deprecated alias of `--modality`.

## Evaluation

Training scripts write component outputs under `results/` by default. A lightweight metric utility is provided for prediction CSV files:

```bash
python scripts/evaluate.py \
    --csv results/example_predictions.csv \
    --true-column y_true \
    --pred-column y_pred
```

## Final Consensus

The released workflow treats a consensus model as an average over selected modality-architecture component predictions. After training the selected components and saving their ensemble prediction arrays, prepare a manifest CSV:

```text
component,pred_path,true_path,names_path,weight
ann_homology,results/ld50/ann/.../test-ensemble-pred.npy,results/ld50/ann/.../test-true.npy,results/ld50/ann/.../test-names.npy,1
cnn_facet,results/ld50/cnn/.../test-ensemble-pred.npy,results/ld50/cnn/.../test-true.npy,results/ld50/cnn/.../test-names.npy,1
ctnn_lap,results/ld50/copresheaf/.../test-ensemble-pred.npy,results/ld50/copresheaf/.../test-true.npy,results/ld50/copresheaf/.../test-names.npy,1
snn_curvature,results/ld50/snn/.../test-ensemble-pred.npy,results/ld50/snn/.../test-ensemble-true.npy,results/ld50/snn/.../test-ensemble-names.npy,1
```

Then run:

```bash
python scripts/consensus.py \
    --manifest configs/ld50/consensus_manifest.csv \
    --output-dir results/ld50/consensus \
    --name ld50_final
```

The consensus script aligns samples by name, checks target consistency, averages component predictions, and writes both consensus predictions and metrics.

## Citation

If you use this repository, please cite the MMML manuscript:

```bibtex
@misc{ren2026mmml,
  title  = {Multi-Mathematical-Modal Learning for Molecules and Materials},
  author = {Ren, Yiming and Liu, Xiang and Vermaas, Josh V. and Hajij, Mustafa and Li{\`o}, Pietro and Wei, Guo-Wei},
  year   = {2026}
}
```
