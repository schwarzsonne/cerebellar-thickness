# cerebellar-thickness

Measuring cortical thickness in stained cerebellar sections, built as practice
toward a PhD application on mesoscale cerebellar morphology.

## What this does

Starting from a single Nissl-stained human cerebellum section (Allen Human
Brain Atlas, specimen 100149965, downloaded at a known 32 µm/pixel
calibration), this project:

1. Segments the cerebellar granular layer using multi-Otsu thresholding
2. Measures its thickness across ~11,000 points using a distance-transform
   and skeleton-based method
3. Validates the automatic measurements against 10 independent manual
   measurements made in Fiji (Pearson r = 0.92, mean absolute error ≈ 73 µm)
4. Tests how sensitive the result is to pipeline parameter choices
5. Simulates what the same measurement would look like at typical 7T MRI
   resolutions (0.2, 0.4, 1.0 mm), to quantify the gap between in-vivo
   imaging and histological ground truth

## Key result

| Resolution | Measured thickness |
|---|---|
| Histology (32 µm, true) | 334 µm |
| 0.2 mm (7T layer-imaging protocol) | 507 µm (+52%) |
| 0.4 mm (MP2RAGE) | 758 µm (+127%) |
| 1.0 mm (fMRI) | 2338 µm (+600%) |

Granular layer thickness is substantially overestimated at every MRI
resolution tested, due to partial-volume blurring of the cerebellum's
tightly folded cortex. This is the quantitative case for why in-vivo 7T
measurements of cerebellar morphology need a postmortem histological
ground truth — which is what this PhD project would provide.

See `notebooks/01_explore.ipynb` for the full analysis, and
`thickness_vs_resolution.png` / `resolution_degradation.png` for the key
figures.

## Data

- Source: [Allen Human Brain Atlas](https://atlas.brain-map.org/), Nissl-stained
  coronal section, specimen 100149965
- Calibration: downloaded at downsample factor 32 from native 1 µm/pixel
  scan resolution, giving an exact 32 µm/pixel scale
- `notebooks/data/` contains the downloaded section and the cropped patch
  used for analysis

## Method

- Segmentation: multi-Otsu thresholding (3 classes) isolates the
  granular layer as the darkest intensity class, cleaned with morphological
  opening/closing to remove small artifacts
- Thickness: Euclidean distance transform + skeletonization; local
  thickness at each skeleton point = 2 × distance to nearest edge
- Validation: 10 points spread across the folia (fold tips, straight
  sections, crevices), measured independently in Fiji and compared to the
  automatic result at the same locations
- Resolution simulation: Gaussian blur matched to target voxel size,
  followed by downsample/upsample, approximating MRI partial-volume
  averaging

## Limitations

- Single 2D section from a single donor; no 3D context or inter-subject
  variability
- Manual validation used only 10 points on one image; a larger, blinded
  validation set would give a more robust agreement estimate
- The resolution-degradation simulation approximates partial-volume
  blurring but does not model real MRI noise, motion, or acquisition
  physics
- No real postmortem MRI data was used — the resolution experiment is a
  simulation on histology, not a histology-to-MRI registration

## Setup

```bash
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
jupyter lab
```

Open `notebooks/01_explore.ipynb` and run all cells.

## Status

Built October 2026.