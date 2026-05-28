# mdmot_bench

This project targets **passive localization and multi-target tracking research under distributed dynamic observer conditions**. It provides a **dataset specification** and an **evaluation toolkit** implementation to support reproducible experimental workflows, aiming to benchmark data association, localization, and tracking algorithms in multi-view, multi-target, and dynamic platform scenarios.

The project consists of two main components:

- Passive observation datasets for 2 vs 3 and 3 vs 5 dynamic observers
- Multi-target tracking evaluation toolkit (customized adaptation based on TrackEval)

------

## Part I-A: 2 vs 3 Dataset

### 1. Dataset Overview

This dataset is constructed for passive observation scenarios with **2 dynamic observers tracking 3 targets**. It simulates or collects visual observation data under conditions of continuous observer motion and multiple simultaneous targets, suitable for research in:

- Passive localization (Angle-only Localization)
- Multi-target data association
- Multi-target tracking (MOT)
- Multi-platform collaborative perception and state estimation

The download link for the recorded ROS bags is: [link](https://pan.baidu.com/s/1dEMe61BcntzswjDKO6Wnkw?pwd=mxmq)

------

### 2. Scenario Configuration

- Number of observers: 2
- Number of targets: 3
- Observation method: Passive visual observation
- Observer state: Dynamic motion
- Target state: Dynamic motion

------

### 3. Data Content

Each data sequence typically contains the following information:

- Image data: Image sequences from two observers, supporting RGB format
- Target pose data: Position and attitude of target aircraft in the world coordinate system
- Observer pose data: Position and attitude of observer aircraft in the world coordinate system

------

### 4. ROS Topic Structure

- Image topics
  - `/obs_01/image_raw`
  - `/obs_02/image_raw`
- Observer pose topics
  - `/vrpn_client_node/obs_01/pose`
  - `/vrpn_client_node/obs_02/pose`
- Target pose topics
  - `/vrpn_client_node/target_01/pose`
  - `/vrpn_client_node/target_02/pose`
  - `/vrpn_client_node/target_03/pose`

------

### 5. Camera Parameters

Stored in `2_3-parameter.txt`

------

## Part II: 3D Tracking Metrics Evaluation Toolkit

### 2.1 Toolkit Overview

This project uses a modified version of **TrackEval** for unified evaluation of multi-target tracking results, with custom extensions for 3D point matching scenarios, suitable for passive localization and multi-observer collaborative perception tasks.

The evaluation framework supports multiple mainstream multi-target tracking metrics (including HOTA series metrics) and provides complete evaluation workflows, result statistics, and visualization functions. Through parameter configuration, it can simultaneously support both **3D point-level matching** and **2D bounding box matching** evaluation modes.

------

### 2.2 Data Format Specification

The evaluation toolkit uses an extended MOTChallenge-style text format, where each line represents an observation or tracking result of a target in a specific frame:

```
<frame_number>, <target_id>, -1, -1, -1, -1, 1, x, y, z
```

Here `<x>, <y>, <z>`represent the 3D position of the target in the world coordinate system or a unified reference coordinate system.

------

### 2.3 Dataset Directory Structure and Preparation

#### (1) Ground Truth Data

Create a new dataset folder under:

```
TrackEval/data/gt/mot_challenge/
```

Example:

```
TrackEval/data/gt/mot_challenge/mydata-train/
```

Each dataset folder contains multiple sequence subfolders, with the following structure:

```
my_sequence/
├── gt/
│   └── gt.txt
└── seqinfo.ini
```

- `gt.txt`: Ground truth trajectory data for the corresponding sequence
- `seqinfo.ini`: Sequence configuration information file

The `seqinfo.ini`file must contain at least the following fields:

- `name`: Dataset name
- `frameRate`: Sequence frame rate
- `seqLength`: Total number of frames in the sequence
- `imWidth`: Image width (placeholder if not used)

------

#### (2) Tracking Results Data (Trackers)

Create a folder with the same name as the ground truth dataset under:

```
TrackEval/data/trackers/mot_challenge/mydata-train/
```

Each subdirectory under this path corresponds to a tracking method to be evaluated. Example structure:

```
my_tracker/
└── data/
    └── your_dataset_name.txt
```

- `your_dataset_name.txt`: Tracking result file for this method on the specified dataset
- After evaluation, metric statistics and visualization charts will be automatically generated in the corresponding subfolder

------

#### (3) Sequence Mapping Files (SeqMaps)

Create sequence mapping files under:

```
TrackEval/data/gt/mot_challenge/seqmaps/
```

This file must list all sequence names included in the current evaluation according to the official TrackEval format.

------

### 2.4 Evaluation Execution

For 3D point matching scenarios, execute the following command:

```
cd TrackEval
python3 scripts/run_mot_challenge.py \
    --BENCHMARK your_dataset_name \
    --BOUNDINGBOX False
```

Here, `--BOUNDINGBOX False`indicates that 2D bounding box overlap is not used during evaluation; instead, matching is based on Euclidean distance in 3D space. For evaluating data from different methods only, you can use the following example:

```
cd TrackEval
python3 scripts/run_mot_challenge.py \
    --BENCHMARK air2air \
    --BOUNDINGBOX False \
    --SEQ_INFO 3d-target
```

------

### 2.5 Python / Conda Environment Dependencies

The evaluation toolkit depends on the following Python libraries:

```
pip install pycocotools
pip install scipy
pip install tabulate
```

It is recommended to use a Python 3 runtime environment.

------

## Part III: Association Matching Accuracy Evaluation

For easier data evaluation, relevant data from observers and targets have been organized by combining ROS bag topic subscription and 2D object trackers, and are stored in the following folder:

```
Processed_Data/2_3/
```

Among them, `target1.csv`, `target2.csv`, and `target3.csv`contain ground truth data for the corresponding targets:

```
<timestamp>,x,y,z
```

While `obs_01.csv`and `obs_02.csv`contain ground truth data for the corresponding observers:

```
<timestamp>, <aircraft_x>, <aircraft_y>, <aircraft_z>, <aircraft_qx>, <aircraft_qy>, <aircraft_qz>, <aircraft_qw>, <camera_x>, <camera_y>, <camera_z>, <camera_qx>, <camera_qy>, <camera_qz>, <camera_qw>, <target1_u>, <target1_v>, <target2_u>, <target2_v>, <target3_u>, <target3_v>
```

Association matching accuracy can be calculated by comparing the detection boxes and IDs after association matching with the actual detection positions and IDs.

## Citation
If you publish work based on, or using, this code, we would appreciate citations to the following:

    @artical{liao2025drones,
        author       ={Xin Liao and Bohui Fang and Weiyu Shao and Wenxing Fu and Tao Yang},
        journal      ={Drones}, 
        title        ={Multi-Object Tracking with Distributed Drones’ RGB Cameras Considering Object Localization Uncertainty}, 
        year         ={2025},
        volume       ={9},
        number       ={12},
        pages        ={867},
        }
