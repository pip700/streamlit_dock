# 🧬 Vina-GPU Virtual Screening Pipeline

> Professional molecular docking Streamlit app for computational biologists.

---

# Table of Contents

- [Features](#features)
- [Quick Start](#quick-start)
  - [Docker Container (Recommended)](#docker-container-recommended)
- [Pipeline Steps](#pipeline-steps)
  - [1. Ligand Preparation](#1--ligand-preparation)
  - [2. Protein Preparation](#2--protein-preparation)
  - [3. Grid Generation](#3--grid-generation)
  - [4. Docking](#4--docking)
  - [5. Pose Viewer](#5--pose-viewer)
  - [6. Results](#6--results)
- [RMSD Calculation](#rmsd-calculation)
- [Interaction Analysis](#interaction-analysis)
- [Directory Structure](#directory-structure)
- [Dependencies](#dependencies)

---

# Features

- ✅ **6-Page Sidebar Navigation**
  - Ligand Prep
  - Protein Prep
  - Grid
  - Docking
  - Pose Viewer
  - Results

- ✅ **Auto-refresh**
  - Automatic page navigation after:
    - Ligand preparation
    - Protein preparation
    - Grid definition
    - Docking

- ✅ **Atom-wise RMSD**
  - MCS-based atom mapping (PyMOL-like)
  - Three fallback methods

- ✅ **3D Pose Viewer**
  - Interactive **py3Dmol** visualization
  - Reference ligand overlay

- ✅ **Interaction Analysis**
  - ProLIF-based residue-level interaction detection
  - Side-by-side comparison with reference ligand

- ✅ **Reference Ligand Redocking**
  - Included in docking step
  - RMSD computed automatically

- ✅ **Vina CPU/GPU**
  - Automatically detects available docking engine
  - Falls back to Vina CPU when GPU/OpenCL is unavailable

---

# Quick Start

## Docker Container (Recommended)

The easiest way to run the application is using the pre-built Docker image.

### Requirements

- Docker installed
- NVIDIA Container Toolkit (for GPU acceleration)
- NVIDIA GPU *(optional; CPU fallback is fully supported)*

Run:

```bash
docker run -d --rm \
  --gpus all \
  -p 8500:8501 \
  -v $(pwd):/workspace \
  ghcr.io/pip700/dock:v2
```

After the container starts, open:

```
http://localhost:8500
```

---

# Pipeline Steps

## 1. 🧪 Ligand Preparation (`1_ligand_prepared/`)

### Input

- SMILES
- CSV/TSV upload
- PubChem CID
- PubChem compound name

### Workflow

- Generate 3D conformers
- Convert to PDBQT using:
  - Meeko
  - Open Babel
  - RDKit

### Calculates

- Molecular Weight (MW)
- LogP
- HBD
- HBA
- TPSA

---

## 2. 🧬 Protein Preparation (`2_receptor_prepared/`)

### Input

- PDB ID
- Uploaded protein structure

### Workflow

- Interactive chain selection
- Ligand selection
- 3D preview
- Extract reference ligand for RMSD calculation
- Generate receptor PDBQT using:
  - prepare_receptor
  - Open Babel

---

## 3. 📐 Grid Generation

Three available modes:

1. Reference ligand
2. Manual coordinates
3. Blind docking (whole protein)

### Features

- Live 3D preview
- Grid box overlay
- Automatically navigates to Docking after grid definition

---

## 4. 🚀 Docking (`4_docking/`)

### Features

- Optional reference ligand docking
- Automatic RMSD comparison
- Supports:
  - Vina CPU
  - Vina-GPU (NVIDIA GPU + OpenCL)
- Automatically navigates to Pose Viewer after docking
- RMSD displayed immediately after docking

---

## 5. 🎯 Pose Viewer

### Interactive 3D Visualization

- Reference ligand (green)
- Docked pose (red)

### Interaction Analysis

Residue-level interaction detection including:

- Hydrophobic
- Hydrogen bonds
- π-Stacking
- Metal interactions
- Additional ProLIF-supported interactions

### Additional Analysis

- Side-by-side comparison with reference ligand
- Tanimoto similarity scoring

---

## 6. 📊 Results (`5_results/`)

Includes:

- Full ranked docking table
- ΔΔG values
- Drug-likeness properties
- Interaction fingerprint heatmap
- Tanimoto similarity table
- Summary plots

### Export

- CSV
- JSON

---

# RMSD Calculation

The `RMSDCalculator` performs RMSD calculation using three progressively applied methods.

---

## Method 1 — MCS-Based Atom Mapping *(Most Accurate / PyMOL-like)*

Workflow:

- Finds the Maximum Common Substructure (MCS)
- Maps atoms between crystal and docked ligand
- Aligns structures using RDKit `AlignMol` with atom mapping

Best suited for:

- Different protonation states
- Flexible ligands
- Different atom ordering

---

## Method 2 — RDKit Simple AlignMol

Workflow:

- Assumes identical atom ordering
- Performs direct alignment
- Computes RMSD quickly

Best suited for:

- Same molecule
- Same atom indexing

---

## Method 3 — Element-Matched Kabsch Alignment

Workflow:

- Groups atoms by element
- Greedy nearest-neighbor atom matching
- Performs Kabsch rotation
- Computes RMSD

Used as the final fallback method.

---

# Interaction Analysis

Interaction analysis is performed using **ProLIF v2.2.0**.

Detected interaction types include:

- Hydrophobic
- VdWContact
- HBDonor
- HBAcceptor
- ImplicitHBDonor
- ImplicitHBAcceptor
- PiStacking
- EdgeToFace
- FaceToFace
- Anionic
- Cationic
- CationPi
- PiCation
- MetalAcceptor
- MetalDonor

If ProLIF fails, the pipeline automatically falls back to:

- RDKit-based interaction detection
- Coordinate-based interaction detection


---

# Dependencies

| Package | Purpose |
|----------|----------|
| **streamlit** | Web UI |
| **rdkit** | Cheminformatics, 3D conformers, RMSD |
| **openbabel-wheel** | PDB/PDBQT conversion |
| **prolif** | Interaction analysis |
| **mdanalysis** | Structure parsing for ProLIF |
| **py3Dmol** | Interactive 3D visualization |
| **scipy** | Statistical analysis, Kabsch alignment |
| **scikit-learn** | PCA, clustering |
| **matplotlib / seaborn** | Plot generation |
| **pubchempy** | PubChem compound lookup |

---
