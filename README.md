# Pulmonary Fibrosis CT Analysis Pipeline

A Python pipeline that processes 3D chest CT scans, segments lung tissue, and measures 
lung volume — then checks those volumes against patients' real clinical lung-function 
scores (FVC).

Built on the [OSIC Pulmonary Fibrosis Progression](https://www.kaggle.com/competitions/osic-pulmonary-fibrosis-progression) 
dataset (DICOM chest CT + clinical metadata).

---

## What it does

1. **Load** — read each patient's DICOM volume into a 3D array and match it to the clinical metadata.
2. **Quality control** — check metadata for missing values and screen every scan for corrupted/NaN pixels before anything runs.
3. **Segment lungs** — isolate lung tissue from bone and soft tissue using Hounsfield Units:
   - Gaussian smoothing to reduce noise
   - Threshold to the lung HU range (≈ −950 to −300; air ≈ −1000, tissue/bone much higher)
   - Border clearing to drop the scanner table and outside air
   - Connected-component labeling + size filtering to keep the lungs
   - Hole-filling for a clean, solid mask
4. **Measure & validate** — lung volume = voxel count × real-world voxel size (from each scan's own metadata), in cm³. Volumes are then correlated against FVC (% predicted) to see how well they track real lung function.

> Runs on a subset of patients for speed, but scales to any number — it just loops over whatever's present.

---

## Tech stack

- **imageio** — DICOM loading
- **NumPy** — array operations
- **SciPy** (`ndimage`) — filtering, labeling, hole-filling
- **scikit-image** — border clearing
- **pandas** — metadata
- **matplotlib** — visualization

---

## Results

Computed volumes land in a plausible range, but the volume–FVC correlation is weak on 
this small sample — expected, since CT measures lung *size* while FVC measures 
*function*, and fibrosis decouples the two. A larger patient set would give a more 
reliable picture.
