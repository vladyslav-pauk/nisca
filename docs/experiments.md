# Experiment Design

## Structure

Each task is defined in `experiments/{task}/`:

- `config/`: Configuration YAML or JSON
- `results/`: Evaluation outputs
- `checkpoints/`: Model weights
- `wandb/`: Logging data

## Tasks

| Task          | Datasets           | Models           |
|---------------|--------------------|------------------|
| Synthetic     | Simulated mixtures | NISCA, VASCA     |
| Hyperspectral | Urban, Cuprite     | NISCA, CNAE      |
| MRI           | DCE-MRI            | NISCA, VASCA     |
| Finance       | Yahoo Finance      | NISCA            |


### Experiments

Training and evaluation are orchestrated through a unified experiment framework.
Each experiment can be configured for a specific evaluation objective, such as:

[//]: # (Each experiment encodes a specific computational task, typically including:)

- Model selection through multiple independent runs with statistical performance aggregation
- Systematic hyperparameter tuning with grid or random search
- Controlled performance benchmarking and generalization analysis across different datasets
- Model selection and ablation studies across different network architectures, size, or training regimes
- Evaluating identifiability and latent factors recovery across different dataset sizes, or noise regimes

[//]: # (- Model training and validation )
[//]: # (- Hyperparameter sweep for model selection  )
[//]: # (- Comparative analysis of architecture or training regimes  )
[//]: # (- Ablation studies under controlled conditions)

[//]: # (This modular and configuration-driven design ensures reproducibility, scalability, and ease of integration into automated pipelines.)

[//]: # (Each experiment is configured via `yaml` located in `experiments/{experiment_name}/config/` directory.)

Experiments are defined through configuration `.yaml` files located in the `experiments/{experiment_name}/config/` directory.

Find more information on the configuration files [here](doc/configuration.md).

[//]: # (Each configuration specifies model architecture, optimizer settings, and dataset parameters, enabling reproducible and scalable benchmarking.)

[//]: # (paralelize)
[//]: # ( The main entry point is `src/scripts/run_sweep.py`, which orchestrates the training and evaluation of models based on the provided configurations.)
[//]: # (The framework supports multiple experiments, each with its own configuration.)
[//]: # (- model_config.json — specifies encoder/decoder architecture, latent space priors, optimization parameters)
[//]: # (- data_config.json — defines the dataset source &#40;synthetic, medical, satellite&#41;, preprocessing, batch size, etc.)
[//]: # (Example fields in model_config.json:)
[//]: # (The `experiments/` directory contains JSON configuration files for models and datasets:)
[//]: # (- `model.json`: model architecture, prior type, latent dimension, etc.)
[//]: # (- `data.json`: dataset path, loader parameters)
[//]: # (- `sweep.json`: hyperparameter search grid)
[//]: # (```json)
[//]: # ({)
[//]: # (  "project": "isnmm",)
[//]: # (  "experiment": "lmm",)
[//]: # (  "model": "vasca",)
[//]: # (  "run_id": "run_name")
[//]: # (})
[//]: # (```)

Below is a summary of the available experiments:

| **Task**      | **Datasets** | **Models** | **Config Path**                        |
|---------------| --- | --- |----------------------------------------|
| Synthetic     | Synthetic | nisca, vasca | experiments/_tmp/synthetic/config/     |
| Hyperspectral | Urban, Cuprite | cnae, nisca | experiments/hyperspectral/config/      |
| MRI           | DCE-MRI | cnae, nisca, vasca | experiments/mri/config/                |
| Portfolio     | Yahoo Finance | nisca | experiments/_tmp/fin_portfolio_return/ |

Each subfolder contains:

•	data.json or .yaml: data model and generation parameters

•	model.json: model architecture, optimizer, trainer

•	sweep/: optional grid/random sweep over params

**Models**

| **Model** | **Type** | **Key Features**                                                        |
| --- | --- |-------------------------------------------------------------------------|
| **NISCA** | VAE | Dirichlet prior, simplex decoder, PNL encoder                           |
| **CNAE** | AE | Nonlinear ICA extensions, deterministic, constrained optimization |
| **SNAE** | AE | Simplex decoder + CNN                                                   |
| **VASCA** | VAE | Logit transform + convex hull priors                                    |

Architectures defined in src/model/architectures/. Additional benchmarks in src/model/benchmarks/.


Implemented in src/modules/metric/:

•	latent_mse, latent_sam, subspace_distance, r_square

•	mixture_mse_db, spectral_angle, matrix_change, volume

•	observed_psnr, residual_nonlinearity

These are tracked and logged using the experiment-specific ModelMetrics classes.

Datasets handled via src/modules/data/:

| **Type** | **Supported Formats** |
| --- | --- |
| Synthetic | .npy / on-the-fly generation |
| Hyperspectral | .npy, .h5 |
| MRI | .nii, .nii.gz |
| Financial | CSV / NumPy |

Preprocessing: standardization, whitening, masking, spatial reshaping.

# Configuration Guide

[//]: # (Experiments are defined via JSON/YAML files under the `experiments/` directory.)

[//]: # ()
[//]: # (## Example Configuration &#40;YAML&#41;)

[//]: # ()
[//]: # (```yaml)

[//]: # (method: grid)

[//]: # (parameters:)

[//]: # (  model_name: [nisca])

[//]: # (  encoder.hidden_layers:)

[//]: # (    - h1: 128)

[//]: # (      h2: 64)

[//]: # (  decoder.hidden_layers:)

[//]: # (    - h1: 128)

[//]: # (  optimizer.lr.encoder: [0.001])

[//]: # (  optimizer.lr.decoder: [0.01])

[//]: # (  trainer.max_epochs: [2000])

[//]: # (  snr: [25])

[//]: # (```)

[//]: # ()
[//]: # (## Supported Keys)

[//]: # ()
[//]: # (- `model_name`: One of `nisca`, `vasca`, `cnae`, `snae`, `aevb`)

[//]: # (- `data_model`: Dataset identifier)

[//]: # (- `latent_dim`, `snr`, `early_stopping.min_delta`)

[//]: # ()


## `config`

Below is an example of experiment configuration for a single training run:
```yaml
method: grid                        # Sweep strategy
metric:                             # Objective to optimize
  goal: minimize
  name: validation_loss
parameters:
  experiment_name: hyperspectral
  data_model: PaviaU
  nonlinearity: [cnae]
  model_name: [nisca]
  trainer.max_epochs: [2000]
  early_stopping.min_delta: [1e-3]
  data_loader.batch_size: [100]
  dataset_size: [1000]
  observed_dim: [16]
  model.latent_dim: [null]
  snr: [25]
  model.sigma: [null]
  decoder.hidden_layers: 
    - h1: 128
  encoder.hidden_layers:
    - h1: 128
      h2: 64
      h3: 32
      h4: 16
  optimizer.lr.encoder: [0.001]
  optimizer.lr.decoder: [0.01]
  model.mc_samples: [1]
  torch_seed: [1]
  data_seed: [12]
```
To initiate a sweep, provide a list of values to scan, e.g. to compare NISCA and CNAE models, set `model_name: [nisca, cnae]`.