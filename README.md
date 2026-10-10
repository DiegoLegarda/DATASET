# Multi-Robot Visual Tracking Dataset (Full HD 1080p)

This repository provides the canonical annotations and structured dataset for multi-robot visual tracking introduced in the research article:

> **"SoC-FPGA Acceleration of PSO-Based Multi-Robot Visual Tracking via a Channel-Packed Binarized HOG Descriptor"**

---

## 1. Overview

- **Capture Setup:** Fixed overhead camera with native Full HD ($1920 \times 1080$) resolution at 30~FPS.
- **Targets:** Three differential-drive mobile ground robots simultaneously navigating a shared indoor arena.
- **Ground Truth Generation:** Annotated manually for every frame using the MATLAB Image Labeler tool (bounding boxes enclosing the robot chassis and ArUco visual reference) rather than automated marker detection, preventing evaluation benchmarks from being confounded by detector dropouts or occlusions. The proposed tracker operates in an appearance-agnostic manner, processing only grayscale binarized HOG texture patterns without marker identification or decoding.

---

## 2. Benchmark Sequences

The dataset comprises three experimental sequences specifically designed to decouple abrupt kinematic dynamics from background gradient clutter:

| Directory | Canonical Name | Synchronized Frames | Image Package (.zip) | Description |
| :--- | :--- | :---: | :---: | :--- |
| `sequence_1_smooth/` | **Sequence 1 (Smooth Circular Motion)** | 100 (Frames 1--100) | `sequence_1_smooth_images.zip` (12.7 MB) | Continuous curving trajectories over a uniform white paper arena floor with minimal background clutter. |
| `sequence_2_raw_arena/` | **Sequence 2 (90-Degree Turn, Raw Arena)** | 170 (Frames 1--170) | `sequence_2_raw_arena_images.zip` (26.4 MB) | Abrupt 90-degree turn (centered at Frame 124) over an unconstrained arena floor featuring dark borders and rectilinear tape markings. |
| `sequence_2_bg_extension/` | **Sequence 2 (90-Degree Turn, Background Extension)** | 170 (Frames 1--170) | `sequence_2_bg_extension_images.zip` (54.8 MB) | Identical 90-degree turn motion trajectories, with arena markings extended/inpainted to isolate kinematics from clutter. |

> **Note on Image Extraction:** Extracting any sequence archive (e.g., `unzip sequence_1_smooth_images.zip -d sequence_1_smooth/`) automatically populates the `img1/` folder expected by `seqinfo.ini` (`img1/001.jpg` to `img1/NNN.jpg`), strictly containing only the valid ground-truth frames.

---

## 3. Annotation Formats

Each sequence directory contains a `groundtruth/` subfolder providing two complementary formats:

### 3.1. Per-Robot Text Format (`robot_1.txt`, `robot_2.txt`, `robot_3.txt`)
Space-delimited text files containing:
```text
<frame> <bb_left> <bb_top> <bb_width> <bb_height>
```
- `<frame>`: 1-indexed video frame number.
- `<bb_left>`: Horizontal pixel coordinate of the top-left bounding box corner ($X$).
- `<bb_top>`: Vertical pixel coordinate of the top-left bounding box corner ($Y$).
- `<bb_width>`: Bounding box width in pixels.
- `<bb_height>`: Bounding box height in pixels.

### 3.2. Unified MOTChallenge Format (`gt_mot.txt`)
Single comma-delimited file compatible with standard Multiple Object Tracking (MOT) benchmark toolkits (e.g., TrackEval, `py-motmetrics`):
```text
<frame>, <target_id>, <bb_left>, <bb_top>, <bb_width>, <bb_height>, <conf>, <class>, <visibility>
```
Example:
```text
1,1,1457,348,46,49,1,1,1.0
1,2,1340,959,60,49,1,1,1.0
1,3,516,913,72,63,1,1,1.0
```

### 3.3. Sequence Metadata (`seqinfo.ini`)
Standard INI configuration file detailing geometric and temporal properties (example from `sequence_1_smooth/`):
```ini
[Sequence]
name=sequence_1_smooth
title=Sequence 1 (Smooth Circular Motion)
imDir=img1
frameRate=30
seqLength=100
imWidth=1920
imHeight=1080
imExt=.jpg
benchmarkStart=0
benchmarkEnd=40
```

---

## 4. Python Usage Example

```python
import numpy as np

# Load individual robot ground truth (Sequence 1, Robot 1)
data_r1 = np.loadtxt("dataset/sequence_1_smooth/groundtruth/robot_1.txt")
print("Robot 1 - Total annotated frames:", len(data_r1))

# Load full sequence ground truth in MOTChallenge format
mot_data = np.loadtxt("dataset/sequence_1_smooth/groundtruth/gt_mot.txt", delimiter=",")
# mot_data[:, 0]   -> frame number
# mot_data[:, 1]   -> robot target ID (1, 2, or 3)
# mot_data[:, 2:6] -> bounding box coordinates [x, y, w, h]
```
