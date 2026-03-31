# Bundle Skeleton Computation (app-bundle-skeleton)

`app-bundle-skeleton` computes a streamline bundle skeleton (also referred to as backbone or centreline) from a tractography bundle using `bundle_skeleton.py` (based on `tracklib.get_bundle_backbone`). 

Repository contents:

* `bundle_skeleton.py`: Python CLI that computes the bundle skeleton/backbone.
* `main`: Bash wrapper that reads `config.json` and executes the pipeline in a Brainlife-compatible way.
* `tracklib.py`: Implementation of backbone computation.

---

## Author

Gabriele Amorosino (g.amorosino@gmail.com)

## Citation

If you use this app in your research, please cite:

Amorosino, G., Olivetti, E., Jovicich, J., & Avesani, P. (2023, April).
How Does White Matter Registration Affect Tractography Alignment?
In 2023 IEEE 20th International Symposium on Biomedical Imaging (ISBI) (pp. 1–5). IEEE.
https://doi.org/10.1109/ISBI53787.2023.10230615

This app also relies on several functions from DIPY. Please also cite:

Garyfallidis, E., Brett, M., Amirbekian, B., Rokem, A., Van Der Walt, S., Descoteaux, M., et al. (2014).
DIPY, a library for the analysis of diffusion MRI data.
Frontiers in Neuroinformatics, 8, 8.

---

## Usage

### Running on Brainlife.io

Use the Brainlife UI as usual:

* Provide a `track/tck` input bundle
* Provide a `neuro/anat/t1w` reference image
* Configure parameters (optional)
* Run the app

### Running locally

#### Prerequisites

* Python 3.x
* Required Python libraries (`numpy`, `scipy`, `nibabel`, `dipy`)

#### Steps

```bash
git clone https://github.com/your-username/app-bundle-skeleton.git
cd app-bundle-skeleton
chmod +x main
./main
```

You can also specify a custom config:

```bash
CONFIG=/path/to/config.json ./main
```

---

## Execution model

The app follows the standard Brainlife execution pattern:

* Reads inputs and parameters from `config.json`
* Validates parameters
* Runs `bundle_skeleton.py`
* Writes outputs under `./track/`
* Optionally writes `product.json`

This follows the Brainlife app contract (config-driven execution and datatype-based outputs). 

---

## `config.json` keys and CLI mapping

All keys are in **snake_case** and mapped to CLI arguments.

### 1) Inputs

| config key |   Type | CLI mapping | Notes                          |
| ---------- | -----: | ----------- | ------------------------------ |
| `tck`      | string | positional  | Input tractogram (`track.tck`) |
| `t1`       | string | positional  | Reference anatomical image     |

---

### 2) Core parameters

| config key       |  Type | CLI flag           | Default  |
| ---------------- | ----: | ------------------ | -------- |
| `perc`           | float | `--perc`           | disabled |
| `smooth_density` |  bool | `--smooth-density` | true     |
| `length_thr`     | float | `--length-thr`     | 0.90     |
| `N_points`       |   int | `--n-points`       | 32       |

---

### 3) Backbone computation options

| config key       |   Type | CLI flag           | Default |
| ---------------- | -----: | ------------------ | ------- |
| `average_type`   | string | `--average-type`   | mean    |
| `endpoint_mode`  | string | `--endpoint-mode`  | median  |
| `keep_endpoints` |   bool | `--keep-endpoints` | false   |

Valid values:

* `average_type`: `mean`, `median`
* `endpoint_mode`: `mean`, `median`, `median_project`

---

### 4) Output mode options

| config key       |  Type | CLI flag           | Default  |
| ---------------- | ----: | ------------------ | -------- |
| `representative` |  bool | `--representative` | false    |
| `spline_smooth`  | float | `--spline-smooth`  | disabled |

---

## Important behaviour

* If `perc` is **omitted**, core streamline selection is disabled
  (equivalent to `perc = 0`) 
* Output always contains **one streamline**
* Output is written to:

  ```
  ./track/track.tck
  ```

---

## Algorithm overview

The computation consists of:

1. Load tractogram and reference image
2. Resample streamlines to `N_points`
3. Align streamline orientations
4. Filter short streamlines (`length_thr`)
5. Compute backbone (mean or median)
6. Optionally:

   * Select representative streamline
   * Apply spline smoothing
7. Save output tractogram

This pipeline is implemented in `get_bundle_backbone(...)`. 

---

## Example configs

### Example 1 — Default backbone

```json
{
  "tck": "/path/to/bundle.tck",
  "t1": "/path/to/t1.nii.gz"
}
```

### Example 2 — Core streamline selection

```json
{
  "tck": "/path/to/bundle.tck",
  "t1": "/path/to/t1.nii.gz",
  "perc": 0.75,
  "smooth_density": true,
  "length_thr": 0.90
}
```

### Example 3 — Representative streamline + smoothing

```json
{
  "tck": "/path/to/bundle.tck",
  "t1": "/path/to/t1.nii.gz",
  "representative": true,
  "spline_smooth": 5.0,
  "keep_endpoints": true,
  "endpoint_mode": "median_project"
}
```

---

## Outputs

Outputs are written under `./track/`:

* `track.tck`: bundle skeleton/backbone (single streamline)
* `product.json`: summary of parameters (optional)

---

## License

MIT License. See `LICENSE` file for details.
