# The Radar Ghost Dataset
This is the landing page for the radar ghost dataset (work in progress).

Accompanying github repository for the **Radar Ghost Dataset**.

- Paper: https://ieeexplore.ieee.org/document/9636338
- Dataset: [![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.6474851.svg)](https://doi.org/10.5281/zenodo.6474851)


## Overview
The **Radar Ghost Dataset** -- a dataset with detailed manual annotations for different kinds of radar ghost detections. We hope that our dataset encourages more researchers to engage in the fields of radar based multi-path objects.

## Dataset

### Downloads
- `original.zip` contains the 111 hand labeled sequences as described in the paper
- `virtual.zip` contains additional 460 sequences whith multiple objects. Those were created by overlaying original sequences from the same scenario.

### H5 Files
The dataset is provided as `h5` files (after unpacking the zip files).

```python
import h5py

path = 'radar_ghost_example.h5'
with h5py.File(path, 'r') as data:
    radar_data = data['radar']  # numpy struct array
    lidar_data = data['lidar']  # numpy struct array

# work with data ...
```

### Radar Data
Description of data format - work in progress.

### Lidar Data
Description of data format - work in progress.

### Folder Structure and File Name Conventions
Description of folder structure and naming conventions - work in progress.

## Scenario Description
Image of each scenario - work in progress.
