<h1 align="center">Fiber Bundle Skeleton Computation</h1>

<p align="center">
  Computes the skeleton (backbone or centreline) of a white matter bundle from a .tck file with a DIPY-based tool and outputs it as a .tck.
</p>

<p align="center">
  <a href="https://github.com/brainlife/abcd-spec"><img src="https://img.shields.io/badge/ABCD_Spec-v1.1-green.svg" alt="ABCD Spec v1.1"></a>
  <a href="https://doi.org/10.25663/brainlife.app.915"><img src="https://img.shields.io/badge/Brainlife-brainlife.app.915-blue.svg" alt="Run on brainlife.io"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-see_LICENSE-lightgrey.svg" alt="License"></a>
</p>

## At a glance

<!-- brainlife-app:begin (generated from the App manifest; edit the manifest, not this block) -->

> **Computes the skeleton (backbone or centreline) of a white matter bundle from a .tck file with a DIPY-based tool and outputs it as a .tck.**

Runs bundle_skeleton.py: selects the core streamlines of the bundle, resamples them to N_points and averages them into one representative backbone, or optionally returns the streamline closest to it. Use it to get a single centreline for tract profiling or visualisation. Needs a single bundle and a reference T1w image in the same space.

| | |
|---|---|
| **Inputs** | `neuro/track/tck` · `neuro/anat/t1w` |
| **Outputs** | `neuro/track/tck` |
| **Container** | `gamorosino/tract_align:latest` |
| **Run it** | [doi.org/10.25663/brainlife.app.915](https://doi.org/10.25663/brainlife.app.915) |

#### Inputs

| Input | Datatype | Required | Description |
|---|---|---|---|
| `track` | `neuro/track/tck` | yes | Single white matter bundle (.tck). |
| `t1` | `neuro/anat/t1w` | yes | Reference T1-weighted image in the same space as the bundle. |

#### Outputs

| Output | Datatype | Description |
|---|---|---|
| `track` | `neuro/track/tck` | Bundle skeleton (or representative streamline) as track.tck. |

<details>
<summary><b>Parameters</b> (9)</summary>

| Parameter | Type | Default | Description |
|---|---|---|---|
| `perc` | number | `0.75` | Core streamline percentile used to select the bundle core (0 disables). |
| `length_thr` | number | `0.75` | Keep streamlines longer than length_thr times the maximum length. |
| `smooth_density` | boolean | `false` | Enable density smoothing in core selection. |
| `N_points` | number | `100` | Number of points used to resample each streamline. |
| `keep_endpoints` | boolean | `false` | Use endpoint_mode for endpoints and preserve them through smoothing. |
| `average_type` | enum | `mean` | Aggregation method for interior points (mean or median). Options: `mean`, `median`. |
| `endpoint_mode` | enum | `median` | Endpoint aggregation method when keep_endpoints is enabled. Options: `median`, `mean`, `median_project`. |
| `representative` | boolean | `false` | Output the streamline closest to the backbone instead of the backbone. |
| `spline_smooth` | boolean | `false` | Apply spline smoothing to the backbone. |

</details>

<!-- brainlife-app:end -->

## Running the App

### On brainlife.io

Open the App at [doi.org/10.25663/brainlife.app.915](https://doi.org/10.25663/brainlife.app.915), choose `neuro/track/tck`, `neuro/anat/t1w` from your project, set the parameters and press **Execute**. The results are saved to your project as `neuro/track/tck`.

### Locally

You need [Singularity/Apptainer](https://apptainer.org/) (the App runs in `gamorosino/tract_align:latest`), [`jq`](https://jqlang.github.io/jq/).

1. Clone this repository.
2. Create `config.json` with the paths to your inputs and the parameters:

   ```json
   {
       "tck": "/path/to/track/tck",
       "t1": "/path/to/t1/t1",
       "perc": 0.75,
       "length_thr": 0.75,
       "smooth_density": false,
       "N_points": 100,
       "keep_endpoints": false,
       "average_type": "mean",
       "endpoint_mode": "median",
       "representative": false,
       "spline_smooth": false
   }
   ```

3. Run `./main`.

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

## 1) Inputs

| config key |   Type | CLI mapping | Notes                          |
| ---------- | -----: | ----------- | ------------------------------ |
| `tck`      | string | positional  | Input tractogram (`track.tck`) |
| `t1`       | string | positional  | Reference anatomical image     |

---

## 2) Core parameters

| config key       |  Type | CLI flag           | Default  |
| ---------------- | ----: | ------------------ | -------- |
| `perc`           | float | `--perc`           | disabled |
| `smooth_density` |  bool | `--smooth-density` | true     |
| `length_thr`     | float | `--length-thr`     | 0.90     |
| `N_points`       |   int | `--n-points`       | 32       |

---

## 3) Backbone computation options

| config key       |   Type | CLI flag           | Default |
| ---------------- | -----: | ------------------ | ------- |
| `average_type`   | string | `--average-type`   | mean    |
| `endpoint_mode`  | string | `--endpoint-mode`  | median  |
| `keep_endpoints` |   bool | `--keep-endpoints` | false   |

Valid values:

* `average_type`: `mean`, `median`
* `endpoint_mode`: `mean`, `median`, `median_project`

---

## 4) Output mode options

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

## Example 1 — Default backbone

```json
{
  "tck": "/path/to/bundle.tck",
  "t1": "/path/to/t1.nii.gz"
}
```

## Example 2 — Core streamline selection

```json
{
  "tck": "/path/to/bundle.tck",
  "t1": "/path/to/t1.nii.gz",
  "perc": 0.75,
  "smooth_density": true,
  "length_thr": 0.90
}
```

## Example 3 — Representative streamline + smoothing

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

## Citing

If you use this App, please cite it, the software it builds on and brainlife.io:

- Fiber Bundle Skeleton Computation. brainlife.io App. [doi:10.25663/brainlife.app.915](https://doi.org/10.25663/brainlife.app.915)
- Amorosino G, Olivetti E, Jovicich J, Avesani P. How Does White Matter Registration Affect Tractography Alignment? IEEE ISBI (2023). [doi:10.1109/ISBI53787.2023.10230615](https://doi.org/10.1109/ISBI53787.2023.10230615)
- Garyfallidis E, et al. Dipy, a library for the analysis of diffusion MRI data. Front Neuroinform 8:8 (2014). [doi:10.3389/fninf.2014.00008](https://doi.org/10.3389/fninf.2014.00008)
- Hayashi S, Caron BA, Heinsfeld AS, et al. brainlife.io: a decentralized and open-source cloud platform to support neuroscience research. *Nature Methods* 21:809–813 (2024). [doi:10.1038/s41592-024-02237-2](https://doi.org/10.1038/s41592-024-02237-2)
- Avesani P, McPherson B, Hayashi S, et al. The open diffusion data derivatives, brain data upcycling via integrated publishing of derivatives and reproducible open cloud services. *Scientific Data* 6:69 (2019). [doi:10.1038/s41597-019-0073-y](https://doi.org/10.1038/s41597-019-0073-y)

## Authors

Gabriele Amorosino

## License

See [LICENSE](LICENSE).
