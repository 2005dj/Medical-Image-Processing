# Pulmonary Fibrosis CT Analysis Pipeline

A Python pipeline for processing 3D chest CT scans, segmenting lung tissue, and measuring lung volume — with a validation step that checks the computed volumes against patients' real clinical lung-function scores.

Built around the [OSIC Pulmonary Fibrosis Progression](https://www.kaggle.com/competitions/osic-pulmonary-fibrosis-progression) dataset (DICOM chest CT + clinical metadata).

---

## Overview

The pipeline takes raw DICOM CT volumes and runs them through four stages:

1. **Load & organize** — read each patient's DICOM volume and join it with the clinical metadata.
2. **Quality control** — verify metadata completeness and screen every scan for missing or corrupted pixel data before anything downstream runs.
3. **Lung segmentation** — isolate lung tissue from surrounding bone and soft tissue.
4. **Measure & validate** — compute each patient's lung volume in cm³ and correlate it against their clinical FVC (% predicted) to check the measurements against ground truth.

---

## Pipeline

### 1. Data loading
- DICOM volumes are read with `imageio.volread(..., 'DICOM')`, producing a 3D array `(slices, height, width)` per patient.
- Volumes are keyed by patient ID and matched to the clinical metadata CSV.
- A single scan is loaded and displayed first as a sanity check before looping over the full set.

> The notebook runs on a subset of patients for speed, but the pipeline is patient-count agnostic — it loops over whatever patients are present.

### 2. Quality control
Before analysis, each scan is checked for:
- **Missing metadata** — null checks across the clinical fields.
- **Pixel integrity** — every volume is scanned for `NaN` / corrupted pixels.

Only scans that pass both checks continue into the pipeline.

### 3. Lung segmentation
Lungs are separated from surrounding anatomy using their distinct density in **Hounsfield Units (HU)**:
- Gaussian smoothing to reduce noise.
- Intensity thresholding to the lung HU range (roughly −950 to −300 HU; air ≈ −1000, soft tissue/bone sit much higher).
- Border clearing (slice-by-slice) to remove structures touching the scan edge.
- Connected-component labeling + size filtering to keep the true lung regions and drop small artifacts.
- Hole-filling to produce a clean, solid lung mask.

### 4. Volume measurement & validation
- **Volume:** lung voxel count × real-world voxel size (slice thickness × pixel spacing, pulled from the DICOM metadata), reported in cm³. Because spacing comes from each scan's own metadata, volumes stay consistent across scans of different resolutions.
- **Validation:** computed lung volumes are correlated against each patient's **FVC (% predicted)** — a standard clinical measure of lung function — to assess how well the automated measurements reflect real patient data.

---

## Tech stack

- **Python**
- **imageio** — DICOM volume loading
- **NumPy** — array operations
- **SciPy** (`ndimage`) — filtering, labeling, hole-filling
- **scikit-image** — border clearing
- **pandas** — metadata handling
- **matplotlib / seaborn** — visualization

---

