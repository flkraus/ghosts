# Radar Ghost Dataset
This is the accompanying github repository and landing page for the **Radar Ghost Dataset**.

- Paper: https://ieeexplore.ieee.org/document/9636338
- Dataset:
  - Version 1.0: [![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.6474851.svg)](https://doi.org/10.5281/zenodo.6474851) (deprecated for lidar usage)
  - Version 1.1: [![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.6676246.svg)](https://doi.org/10.5281/zenodo.6676246)

## Overview
The **Radar Ghost Dataset** -- a dataset with detailed manual annotations for different kinds of radar ghost detections. We hope that our dataset encourages more researchers to engage in the fields of radar based multi-path objects.

## Dataset

### Changelog

#### Version 1.0
- Upload initial dataset to zenodo

#### Version 1.1
- Upload new version of dataset to zenodo
- Fixed timesync issue between lidar and radar
- Added data in car coordinates for lidar. Added columns `x_cc`, `y_cc`, and `z_cc`.
- Removed cartesian coordinates for sensor coordinate system in lidar. Removed columns `x_sc`, `y_sc`, and `z_sc`. This will break existing code which accesses those fields. Use the car coordinate variants (`_cc`) instead. This removes the need to manually take the mounting position into account.
- Added spherical coordinates for sensor coordinate system in lidar. Added columns `r_sc`, `theta_sc` (elevation), and `phi_sc` (azimuth). Unifies things between lidar and radar. Car coordinates are always cartesian and sensor coordinates always spherical/polar.
- Removed lidar noise. Previously lidar data contained points marked as noise. Those are now removed.
- Added `uuid` column for lidar data.

### Downloads
- `original.zip` contains the 111 hand labeled sequences as described in the paper
- `virtual.zip` contains additional 460 sequences with multiple objects. Those were created by overlaying original sequences from the same scenario.

### Images
Check the `images.zip` for example images of each scenario.

### Mirrors
Check `mirrors.zip` for short descriptions of the reflective surfaces. Contains a `.json` file for each sequence. Additional info regarding the `mirrors` column in the radar data.

### H5 Files
The dataset is provided as `h5` files (after unpacking the zip files). Each h5 file contains an `radar` and `lidar` dataset entry (check code example below).

```python
import h5py
import numpy as np

path = 'radar_ghost_example.h5'
with h5py.File(path, 'r') as data:
    # use np.copy when using context managers (with statement), otherwise data will be empty.
    radar_data = np.copy(data['radar'])  # numpy struct array
    lidar_data = np.copy(data['lidar'])  # numpy struct array

# work with data ...
```

### Annotations/Labels
Check `label_convention.md` for more information.

### Coordinate Systems
We use two types of coordinate systems. Sensor coordinate systems for each sensor (two radars, one lidar) and a "car coordinate" system which is the same for all sensors ([see exception for v1.0 of dataset](#issue:-missing-car-coordinate-system-for-lidar-in-v1.0)).

Each positional entry has an indicator attached which coordinate system it belongs to. The car coordinate system is indicated with `_cc` and the sensor coordinate systems with `_sc`.

All coordinate systems use the standard convention for car coordinate systems: x-axis points to the front, y-axis points to the left, and the z-axis points up. A positive azimuth angle (phi) within the radar coordinate frames indicates measures left to the y-axis (same as yaw).

All entries are given in meters [m] or radians [rad] for angles.

#### Sensor Mounting Positions
The mounting positions of all sensors in the car coordinate system. Note that the two radar sensors are mounted at an angle (indicated by the yaw angle).
```python
left_radar_mounting_pos = {
    'x': 3.739,  # [m]
    'y': 0.658,  # [m]
    'z': 0.0305,  # [m]
    'yaw': 0.523599  # [rad]
}

right_radar_mounting_pos = {
    'x': 3.739,  # [m]
    'y': -0.658,  # [m]
    'z': 0.0305,  # [m]
    'yaw': -0.523599  # [rad]
}

lidar_mounting_pos = {
    'x': 3.739,  # [m]
    'y': -0.194,  # [m]
    'z': 0.2806,  # [m]
}
```


### Radar Data
Description of each entry for radar data in the h5 file. Note: the radar has no elevation information.

| Column               | Description |
| -------------------- | ----------- |
| frame                | Each frame contains measurement cycle of both radar sensors (ignoring rare frame drops) |
| frame_timestamp      | Timestamp for each frame, relative to start of recording. |
| timestamp            | Timestamp of the actual measurement, left and right sensor trigger with slight offset from each other. Preferably use "frame" or "frame_timestamp" instead of this. Only use this if you want to differentiate between the two sensors. |
| sensor               | Which sensor took the measurement ("left" or "right") |
| x_cc                 | x-coordinate [m] in car coordinate system |
| y_cc                 | y-coordinate [m] in car coordinate system |
| r_sc                 | range [m] (distance to sensor) in sensor coordinate system (cf. mounting positions of sensors) |
| phi_sc               | Azimuth angle [rad] in sensor coordinate system (cf. mounting positions of sensors) |
| vr_sc                | Doppler value or "radial velocity" [m/s] in sensor coordinates |
| amp                  | Amplitude of radar echo |
| uuid                 | Universal unique identifier, each radar detection is assigned an unique id. This helps track single detections over the whole dataset. Useful for debugging. |
| original_uuid        | Only present for virtual sequences (sequences created by overlaying multiple sequences). When creating virtual sequences each detection is assigned a new uuid. The original uuid is preserved in this field. Note that the actual coordinates will differ from the original detection, since random noise is applied. |
| label_id             | Integer encoding the label (e.g. real object, type 1 second bounce, ...). Check `label_convention.md` for more information. |
| instance_id          | Instance or cluster id. May stretch over multiple frames. A single object might have multiple instance_ids if it is not visible for a certain number of frames. |
| human_readable_label | the label_id field decoded to a human friendly format. |
| group                | Whether a group was labeled. Only present in one scenario where a group of pedestrians was labeled. |
| mirror               | Only valid for certain multi bounce reflection. Short description of the reflective surface is given. A slightly more verbose description can be found inside mirrors.zip |

### Lidar Data v1.1
Description of each entry for lidar data in the h5 file. This applys only to data from the dataset version 1.1.

| Column               | Description |
| -------------------- | ----------- |
| timestamp            | Timestamp, relative to start of recording. |
| x_cc                 | x-coordinate [m] in car coordinate system |
| y_cc                 | y-coordinate [m] in car coordinate system |
| z_cc                 | z-coordinate [m] in car coordinate system |
| r_sc                 | range [m] (distance to sensor) in sensor coordinate system (cf. mounting positions of sensors) |
| theta_sc             | Elevation/polar angle [rad] in sensor coordinate system (cf. mounting positions of sensors) |
| phi_sc               | Azimuth angle [rad] in sensor coordinate system (cf. mounting positions of sensors) |
| uuid                 | Universal unique identifier, each lidar points is assigned an unique id. Useful for debugging. |

### Lidar Data v1.0 (Deprecated!)
Description of each entry for lidar data in the h5 file. This applys only to data from the dataset version 1.0.

| Column               | Description |
| -------------------- | ----------- |
| timestamp            | Timestamp, starting at zero. Not properly synced to radar in v1.0! |
| x_sc                 | x-coordinate [m] in sensor coordinate system |
| y_sc                 | y-coordinate [m] in sensor coordinate system |
| z_sc                 | z-coordinate [m] in sensor coordinate system |

### Issues with v1.0 Lidar Data
Version 1.0 of the dataset has three issues with the lidar data. If you want to work with Lidar data it is advised to use data from a newer version (v1.1).

#### Issue: Sensor Noise
Data contains lidar points which are marked as noise. Fixed in v1.1.

#### Issue: Timestamps Sync Issue between Radar and Lidar in v1.0
The currently provided timestamps for the lidar do not necessarily sync correctly with the radar. Might be off by some tenths of a second. The current timestamps for lidar and radar timestamps start at 0. This is incorrect, since measurements of lidar and radar do not necessarily start at the same time. Fixed in v1.1.

#### Issue: Missing Car Coordinate System for Lidar in v1.0
The lidar data currently only contains "sensor coordinates" (`_sc`) and not "car coordinates" (`_cc`).
In v1.1 data is provided in car coordinates. Quickfix: account for the mounting position relative to the car coordinate system.

```python
lidar_mounting_pos = {
    'x': 3.739,  # [m]
    'y': -0.194,  # [m]
    'z': 0.2806,  # [m]
}

# sensor coordinates to car coordinates
lidar_data['x_cc'] = lidar_data['x_sc'] + lidar_mounting_pos['x']
lidar_data['y_cc'] = lidar_data['y_sc'] + lidar_mounting_pos['y']
lidar_data['z_cc'] = lidar_data['z_sc'] + lidar_mounting_pos['z']
```


### Folder Structure and File Name Conventions
The dataset is split into train/val/test. Those are the same splits as used in our paper. Feel free to split the dataset whichever way you like. The presented splits are a suggestion and for reproducibility. If you chose to split the dataset differently (e.g. cross-validation) make sure that train and test to not contain sequences from the same scenario. There are also some scenarios where only the car position differs slightly, those should also be in the same split to avoid overfitting on the test data. Check the provided images to identify those similar scenarios. Note that in our split selection val and train use sequences from the same scenarios.

After unzipping the data you are presented with the following file structure. The folder `original/` contains the original 111 sequences whereas the `virtual/` folder contains the overlaid sequences with multiple objects. Check the paper for more information.

```
|- original/
   |- train/
      |- scenario-xx_sequence-01_xxx_train.h5
      |- scenario-xx_sequence-02_xxx_train.h5
      |- ...
   |- val/
      |- scenario-xx_sequence-01_xxx_val.h5
      |- scenario-xx_sequence-02_xxx_val.h5
      |- ...
   |- test/
      |- scenario-xx_sequence-01_xxx_test.h5
      |- scenario-xx_sequence-02_xxx_test.h5
      |- ...

|- virtual/
   |- train/
      |- scenario-xx_sequences-x-x_xxx_train.h5
      |- scenario-xx_sequences-x-x_xxx_train.h5
      |- ...
   |- val/
      |- scenario-xx_sequences-x-x_xxx_train.h5
      |- scenario-xx_sequences-x-x_xxx_train.h5
      |- ...
   |- test/
      |- scenario-xx_sequences-x-x_xxx_train.h5
      |- scenario-xx_sequences-x-x_xxx_train.h5
      |- ...
```


For the sequences in the  `original/` folder the naming conventions are as follows.
```
scenario-xx_sequence-0x_class_split.h5
```
Scenarios range from 01 to 21 and sequences depending on the scenario from 01 to 08.
The `class` entry is either `ped` or `cycl` depending on the class of the main object in the sequence (c.f. paper for more information on the recording process). The `split` is one of `train`, `val`, or `test`.

The sequences in the `virtual/` folder have a slightly more involved naming scheme:

```
scenario-xx_sequences-x-...-x_start-frames-x-...-x_class-...-class_split.h5
```
Each virtual sequence is comprised of multiple real sequences of a single scenario. The used sequences are denoted after the `sequences-` part (between two and 5 sequences). When overlaying the sequences we do not start each sequence from zero. The used start frames for each sequence is indicated after the `start-frames-` part of the name. This also allows to overlay the same sequence but started at different times. After that the classes of the main object of each sequence is listed (ped or cycl). Finally the split is indicated.
