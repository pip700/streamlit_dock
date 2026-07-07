# 🧬 Vina-GPU Virtual Screening Pipeline

Professional molecular docking Streamlit app for computational biologists.

## Features

- **6-Page Sidebar Navigation**: Ligand Prep → Protein Prep → Grid → Docking → Pose Viewer → Results
- **Auto-refresh**: Automatic page navigation after key steps (ligand prep, protein prep, grid definition, docking)
- **Atom-wise RMSD**: MCS-based atom mapping (PyMOL-like) with 3 fallback methods
- **3D Pose Viewer**: Interactive py3Dmol visualization with reference overlay
- **Interaction Analysis**: ProLIF-based residue-level interaction detection + comparison
- **Reference Ligand Redocking**: Included in docking step, RMSD computed automatically
- **Vina CPU/GPU**: Auto-detects available engine; falls back to Vina CPU

## Quick Start

```bash
pip install -r requirements.txt
streamlit run app.py
```

## Pipeline Steps

### 1. 🧪 Ligand Prep (`1_ligand_prepared/`)
- Input: SMILES, CSV/TSV upload, or PubChem CID/name
- Generates 3D conformers → PDBQT via Meeko/obabel/RDKit
- Computes MW, LogP, HBD, HBA, TPSA

### 2. 🧬 Protein Prep (`2_receptor_prepared/`)
- Input: PDB ID or upload
- Interactive chain/ligand selection with 3D preview
- Extract reference ligand for RMSD calculation
- Generates receptor PDBQT via prepare_receptor/obabel

### 3. 📐 Grid Generation
- Three modes: From reference ligand, Manual, Blind (whole protein)
- Live 3D preview with grid box overlay
- Auto-navigates to Docking after grid definition

### 4. 🚀 Docking (`4_docking/`)
- Includes reference ligand option for RMSD comparison
- Vina CPU (default) or Vina-GPU (if NVIDIA GPU + OpenCL available)
- Auto-navigates to Pose Viewer after completion
- RMSD displayed immediately after docking

### 5. 🎯 Pose Viewer
- Interactive 3D viewer with reference (green) + docked pose (red)
- Residue-level interaction analysis (Hydrophobic, H-bond, π-stacking, etc.)
- Side-by-side comparison with reference ligand
- Tanimoto similarity scoring

### 6. 📊 Results (`5_results/`)
- Full results table with ranking, ddG, drug-likeness
- Interaction fingerprint heatmap
- Tanimoto similarity table
- CSV/JSON download + summary plots

## RMSD Calculation

The `RMSDCalculator` uses three methods tried in order:

1. **MCS-based atom mapping** (most accurate, PyMOL-like):
   - Finds Maximum Common Substructure between crystal and docked poses
   - Maps atoms via MCS match
   - Aligns using RDKit `AlignMol` with atom map
   - Best for: Different protonation states, flexible ligands

2. **RDKit simple AlignMol** (fast, same-molecule assumption):
   - Assumes atoms are in the same order
   - Quick alignment and RMSD

3. **Element-matched Kabsch** (fallback):
   - Groups atoms by element
   - Greedy nearest-neighbor matching
   - Kabsch rotation + RMSD

## Interaction Analysis

Uses **ProLIF v2.2.0** for residue-level interaction detection:

- Hydrophobic, VdWContact
- HBDonor, HBAcceptor, ImplicitHBDonor, ImplicitHBAcceptor
- PiStacking, EdgeToFace, FaceToFace
- Anionic, Cationic, CationPi, PiCation
- MetalAcceptor, MetalDonor

Falls back to RDKit-based and coordinate-based detection if ProLIF fails.

## Directory Structure

```
vs_workspace/
├── 1_ligand_prepared/    # Ligand PDBQT files
│   └── ligands/
├── 2_receptor_prepared/  # Receptor PDBQT + reference ligand
│   ├── receptor/
│   └── ref_E20_A_601.pdb
├── 4_docking/            # Docking outputs
│   ├── ligands/
│   ├── receptor/
│   ├── output/           # *_out.pdbqt files
│   └── results/
└── 5_results/            # Final reports + plots
```

## Dependencies

| Package | Purpose |
|---------|---------|
| streamlit | Web UI |
| rdkit | Cheminformatics, 3D conformers, RMSD |
| openbabel-wheel | PDB/PDBQT conversion |
| prolif | Interaction analysis |
| mdanalysis | Structure parsing for ProLIF |
| py3Dmol | Interactive 3D visualization |
| scipy | Statistical analysis, Kabsch alignment |
| scikit-learn | PCA, clustering |
| matplotlib/seaborn | Plots |
| pubchempy | PubChem lookup |

## Troubleshooting

- **"No docking engine found"**: Install AutoDock Vina: `conda install -c conda-forge autodock-vina`
- **"obabel not found"**: Install OpenBabel: `pip install openbabel-wheel`
- **3D viewer not loading**: Check browser compatibility; py3Dmol requires WebGL
- **Limited interactions detected**: ProLIF detects more interactions with explicit hydrogens; try protonating your PDB with `obabel input.pdb -opdb -p 7.4 -O output.pdb`
