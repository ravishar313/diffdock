# DiffDock Usage Guide

This guide provides instructions for running DiffDock molecular docking predictions using the configured micromamba environment.

## Quick Start

### Activate Environment

```bash
micromamba activate diffdock
```

### Verify GPU Support

```bash
python -c "import torch; print('CUDA available:', torch.cuda.is_available()); print('GPU count:', torch.cuda.device_count()); print('GPU name:', torch.cuda.get_device_name(0) if torch.cuda.is_available() else 'No GPU')"
```

Expected output:
```
CUDA available: True
GPU count: 1
GPU name: NVIDIA GeForce RTX 4070
```

## Running Predictions

### Single Complex Prediction (Recommended)

For fast testing with minimal computational resources:

```bash
micromamba run -n diffdock python -m inference \
  --config default_inference_args.yaml \
  --protein_path examples/1a46_protein_processed.pdb \
  --ligand examples/1a46_ligand.sdf \
  --out_dir results/example_prediction \
  --samples_per_complex 3 \
  --batch_size 1
```

### Standard Single Complex Prediction

```bash
micromamba run -n diffdock python -m inference \
  --config default_inference_args.yaml \
  --protein_path examples/1a46_protein_processed.pdb \
  --ligand examples/1a46_ligand.sdf \
  --out_dir results/example_prediction
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
  --protein_sequence "GIQSYCTPPYSVLQDPPQPVV" \
  --ligand "COc(cc1)ccc1C#N" \
  --out_dir results
```

## Results and Output

### Output Directory Structure

Results are saved in the specified output directory with the following structure:

```
results/example_prediction/
└── complex_0/
    ├── rank1.sdf                    # Top prediction (highest confidence)
    ├── rank1_confidence-2.49.sdf    # Prediction with confidence score
    ├── rank2_confidence-2.99.sdf
    ├── rank3_confidence-3.29.sdf
    ├── rank4_confidence-3.42.sdf
    ├── rank5_confidence-3.68.sdf
    ├── rank6_confidence-3.84.sdf
    ├── rank7_confidence-4.49.sdf
    ├── rank8_confidence-4.73.sdf
    ├── rank9_confidence-5.34.sdf
    └── rank10_confidence-5.50.sdf    # Lowest confidence prediction
```

### Understanding Output Files

- **rank1.sdf**: The top-ranked prediction (best pose)
- **rankX_confidence-Y.sdf**: Alternative predictions ranked by confidence score
- **Confidence Score Interpretation**:
  - > 0: High confidence
  - -1.5 to 0: Moderate confidence  
  - < -1.5: Low confidence

The example above shows:
- Top prediction confidence: -2.49 (moderate confidence)
- 10 total predictions generated
- All predictions saved in SDF format for visualization

### File Formats

- **SDF Files**: Can be opened with:
  - PyMOL (`load rank1.sdf`)
  - VMD
  - Chimera
  - RDKit in Python
  - Other molecular visualization tools

## Performance Notes

### GPU Usage
- DiffDock automatically uses available GPU when detected
- RTX 4070 provides good performance for typical docking tasks
- Processing time varies based on protein size and samples requested

### First Run
- First execution precomputes SO(2)/SO(3) lookup tables (1-2 minutes)
- Subsequent runs are faster as tables are cached
- ESM language model embeddings are generated for each unique protein

### Resource Optimization
- Use `--samples_per_complex 3` for quick testing
- Use `--batch_size 1` for minimal memory usage
- Increase samples (default: 10) for better results but longer runtime

## Input File Requirements

### Protein Files
- Format: `.pdb` files
- Can use processed or unprocessed protein structures
- ESMFold automatically folds protein sequences if provided

### Ligand Files
- Supported formats: `.sdf`, `.mol2`, or SMILES strings
- RDKit-compatible molecular structures
- Single or multiple molecules per file


## Example Test Command

The following command was successfully tested on the provided examples:

```bash
micromamba run -n diffdock python -m inference \
  --config default_inference_args.yaml \
  --protein_path examples/1a46_protein_processed.pdb \
  --ligand examples/1a46_ligand.sdf \
  --out_dir results/example_prediction \
  --samples_per_complex 3 \
  --batch_size 1
```

This generated 10 predictions in approximately 1 minute using GPU acceleration.

## Troubleshooting

### Common Warnings (Normal Operation)
- `RuntimeWarning: invalid value encountered in sqrt` - Can be ignored
- `BiopythonDeprecationWarning` - Does not affect functionality
- `FutureWarning` about `torch.load` - Safe to ignore for trusted models

### GPU Not Detected
- Verify CUDA installation with `nvcc --version`
- Check GPU drivers are up to date
- Ensure conda environment has proper CUDA support

### Memory Issues
- Reduce `--samples_per_complex`
- Use `--batch_size 1`
- Consider CPU-only mode if GPU memory insufficient