# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DiffDock is a state-of-the-art molecular docking method using diffusion models. It predicts the 3D structure of protein-ligand complexes and supports both single complex predictions and batch processing. The repository includes both the original DiffDock model and the newer DiffDock-L version with improved performance.

## Development Environment

### Environment Setup
- Use conda environment from `environment.yml`: `conda env create --file environment.yml`
- Activate environment: `conda activate diffdock`
- Python 3.9.18 with specific PyTorch 1.13.1+cu117 and PyG versions
- Key dependencies: ESM-2, OpenFold, RDKit, e3nn, PyTorch Geometric

### Docker Development
- Build container: `docker build . -f Dockerfile -t diffdock`
- Use pre-built: `docker pull rbgcsail/diffdock`
- Run with GPU: `docker run -it --gpus all rbgcsail/diffdock`
- Activate environment in container: `micromamba activate diffdock`

## Core Commands

### Inference (Docking Prediction)
Single complex with files:
```bash
python -m inference --config default_inference_args.yaml --protein_path protein.pdb --ligand ligand.sdf --out_dir results
```

Batch processing with CSV:
```bash
python -m inference --config default_inference_args.yaml --protein_ligand_csv data/protein_ligand_example.csv --out_dir results/user_predictions_small
```

Using protein sequence (ESMFold):
```bash
python -m inference --config default_inference_args.yaml --protein_sequence "SEQUENCE" --ligand "SMILES" --out_dir results
```

### Training
```bash
python train.py [args]
```

### Evaluation
```bash
python -m evaluate --config default_inference_args.yaml --dataset pdbbind --data_dir data/PDBBind_processed/ --split test --esm_embeddings_path data/esm2_embeddings.pt
```

### Web UI
```bash
python app/main.py  # Available at http://localhost:7860
```

### ESM Embedding Preparation
1. Generate sequences: `python datasets/esm_embedding_preparation.py`
2. Extract embeddings using ESM repository
3. Convert to PyTorch: `python datasets/esm_embeddings_to_pt.py`

## Architecture

### Model Types
- **AAModel**: All-atom model using detailed representations (`models/aa_model.py`)
- **CGModel**: Coarse-grained model for efficiency (`models/cg_model.py`)
- **Confidence Model**: Separate confidence prediction model (`confidence/`)

### Key Components
- **Diffusion Process**: Manages the reverse diffusion sampling process (`utils/diffusion_utils.py`)
- **Sampling**: Handles position randomization and sampling (`utils/sampling.py`)
- **Geometry**: SO(3) and torus operations for rotations/torsions (`utils/so3.py`, `utils/torsion.py`)
- **Data Loading**: Complex molecular data processing (`datasets/`)
- **Inference Pipeline**: Main inference logic (`inference.py`)

### Data Processing
- Supports multiple input formats: PDB files, SDF, MOL2, SMILES strings
- ESM-2 language model embeddings for protein sequences
- Automatic protein folding with ESMFold when sequence provided
- Preprocessed datasets: PDBBind, BindingMOAD, DockGen, PoseBusters

### Configuration
- `default_inference_args.yaml`: Main inference parameters
- Model directories in `./workdir/v1.1/` for pre-trained weights
- Confidence model weights in confidence_model_dir

### Important Notes
- First run precomputes SO(2)/SO(3) lookup tables (takes a few minutes)
- GPU recommended for performance, CPU supported but slower
- Model outputs confidence scores: >0 (high), -1.5 to 0 (moderate), < -1.5 (low)
- Uses file descriptor limits (64000) for batch processing

### Testing and Validation
- Evaluate on time-split test sets for generalization
- Use `--save_visualisation` flag to save SDF files
- Support for RMSD calculations and symmetry corrections