# DiffDock Installation Guide

This document provides step-by-step instructions for setting up DiffDock with micromamba.

## Prerequisites

- Linux operating system
- micromamba installed (version 2.3.2+)
- CUDA-compatible GPU (optional but recommended for performance)
- Git

## Environment Setup

### 1. Clone the Repository

```bash
git clone https://github.com/gcorso/DiffDock.git
cd DiffDock
```

### 2. Verify micromamba Installation

```bash
which micromamba
micromamba --version
```

Expected output should show micromamba path and version (2.3.2+).

### 3. Create Conda Environment

Create the environment using the provided `environment_simple.yml` file:

```bash
micromamba env create --file environment_simple.yml --name diffdock
```

This will install:
- Python 3.9.18
- PyTorch 2.5.1 with CUDA 12.1 support (upgraded for better performance)
- PyTorch Geometric and related packages
- ESM-2 language model
- RDKit for chemical informatics
- Other required dependencies

**Note**: This environment includes PyTorch 2.5.1 which is compatible with CUDA 12.9 for optimal performance.

### 4. Activate the Environment

```bash
micromamba activate diffdock
```

### 5. Verify Installation

Check that key packages are installed:

```bash
python --version
pip list | grep -E "(torch|esm|e3nn|gradio|rdkit)"
```

Verify PyTorch CUDA support:

```bash
python -c "import torch; print('PyTorch version:', torch.__version__); print('CUDA available:', torch.cuda.is_available())"
```

Expected output:
```
PyTorch version: 2.5.1+cu121
CUDA available: True
```

## OpenFold Installation

With CUDA 12.9 toolkit installed, OpenFold can be installed using the following steps:

### 1. Install CUDA Toolkit (if not already installed)

```bash
# Add NVIDIA CUDA repository
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt update
sudo apt install -y cuda-toolkit-12-9
```

### 2. Install aria2 (required for downloading parameters)

```bash
sudo apt install -y aria2
```

### 3. Clone and Install OpenFold

```bash
# Clone OpenFold repository
git clone https://github.com/aqlaboratory/openfold.git
cd openfold

# Set environment variables
export CONDA_PREFIX=/home/ravi/micromamba/envs/diffdock
export LIBRARY_PATH=$CONDA_PREFIX/lib:$LIBRARY_PATH
export LD_LIBRARY_PATH=$CONDA_PREFIX/lib:$LD_LIBRARY_PATH

# Install third-party dependencies
micromamba run -n diffdock ./scripts/install_third_party_dependencies.sh

# Download AlphaFold parameters
mkdir -p ../openfold_resources
micromamba run -n diffdock ./scripts/download_alphafold_params.sh ../openfold_resources
```

### 4. Verify OpenFold Installation

```bash
export CONDA_PREFIX=/home/ravi/micromamba/envs/diffdock
export LIBRARY_PATH=$CONDA_PREFIX/lib:$LIBRARY_PATH
export LD_LIBRARY_PATH=$CONDA_PREFIX/lib:$LD_LIBRARY_PATH
micromamba run -n diffdock python -c "import openfold; print('OpenFold installed successfully!')"
```

**Note**: OpenFold provides enhanced protein structure prediction capabilities compared to ESMFold alone. The parameter download is ~5.2GB.

## Running DiffDock

### Single Complex Prediction

```bash
micromamba run -n diffdock python -m inference \
  --config default_inference_args.yaml \
  --protein_path protein.pdb \
  --ligand ligand.sdf \
  --out_dir results
```

### Batch Processing

```bash
micromamba run -n diffdock python -m inference \
  --config default_inference_args.yaml \
  --protein_ligand_csv data/protein_ligand_example.csv \
  --out_dir results/user_predictions_small
```

### Using Protein Sequence (ESMFold)

```bash
micromamba run -n diffdock python -m inference \
  --config default_inference_args.yaml \
  --protein_sequence "SEQUENCE" \
  --ligand "SMILES" \
  --out_dir results
```

### Web UI

```bash
micromamba run -n diffdock python app/main.py
```

Access at http://localhost:7860

## Environment Management

### Activate Environment

```bash
micromamba activate diffdock
```

### Run Single Command

```bash
micromamba run -n diffdock <command>
```

### List Environments

```bash
micromamba env list
```

### Remove Environment

```bash
micromamba env remove --name diffdock
```

## Troubleshooting

### Common Issues

1. **CUDA Toolkit Installation**:
   - Ensure CUDA toolkit version matches your driver (CUDA 12.9 recommended)
   - Set CUDA_HOME to `/usr/local/cuda-12.9`
   - Reboot system after CUDA installation if needed

2. **PyTorch CUDA Version Mismatch**:
   - The environment uses PyTorch 2.5.1+cu121 compatible with CUDA 12.9
   - If using different CUDA version, update PyTorch accordingly

3. **OpenFold Installation Issues**:
   - Ensure all environment variables are set correctly
   - Verify CUDA toolkit installation with `nvcc --version`
   - Check that aria2 is installed for parameter downloads

4. **CUDA Memory Issues**:
   - Reduce batch size in configuration
   - Use CPU-only mode if necessary

5. **Package Conflicts**:
   - Use the provided environment file
   - Avoid manual package installation

### Verification Commands

```bash
# Check PyTorch
python -c "import torch; print('PyTorch version:', torch.__version__); print('CUDA available:', torch.cuda.is_available())"

# Check key imports
python -c "import e3nn, rdkit, esm; print('All imports successful')"

# Check OpenFold (if installed)
export CONDA_PREFIX=/home/ravi/micromamba/envs/diffdock
export LIBRARY_PATH=$CONDA_PREFIX/lib:$LIBRARY_PATH
export LD_LIBRARY_PATH=$CONDA_PREFIX/lib:$LD_LIBRARY_PATH
python -c "import openfold; print('OpenFold installed successfully')"

# Check CUDA version
nvcc --version
```

## Notes

- First run may take a few minutes to precompute SO(2)/SO(3) lookup tables
- GPU is recommended for performance but CPU is supported
- Environment includes gradio for web interface
- Uses fair-esm for protein folding capabilities
- OpenFold provides enhanced protein structure prediction (optional installation)
- File descriptor limits may need adjustment for large batch processing (64000+)
- PyTorch 2.5.1 provides better performance and CUDA 12.9 compatibility
- OpenFold parameters download is ~5.2GB but required for full functionality